---
title: "Self-hosting Actual Budget on NixOS with Kanidm OIDC and Tailscale"
description: "How I set up Actual Budget on my NixOS homeserver, secured behind Tailscale, with Kanidm as the OIDC identity provider for multi-user access"
tags: [nixos, self-hosting, actual-budget, kanidm, tailscale, oidc]
comments: true
share: true
cover_image: ''
draft: true
showToc: true
TocOpen: true
---

This is a walkthrough of how I set up [Actual Budget](https://actualbudget.org/) on my NixOS
homeserver, secured behind Tailscale, with [Kanidm](https://kanidm.com/) as the OIDC identity
provider so multiple household members each have their own login.

## Architecture

```
Browser (Tailscale) → Caddy :5006 (TLS) → Actual Budget container :15006
                                              ↕ OIDC
                                         Kanidm :8443 (TLS)
```

- **Tailscale** provides the private network and issues TLS certificates for the hostname
- **Caddy** terminates HTTPS and reverse-proxies to the container
- **Actual Budget** runs as a Podman container
- **Kanidm** is the identity provider, handling user accounts and issuing OIDC tokens

All services are reachable only over Tailscale — nothing is exposed to the public internet.

## Prerequisites

- NixOS with Tailscale enabled (`services.tailscale.enable = true`)
- HTTPS enabled in the Tailscale admin console for your tailnet (required for cert issuance)
- Your Tailscale hostname, e.g. `nixos.example.ts.net`

## NixOS configuration

I split the configuration across three files imported by `configuration.nix`:

### containers.nix — enable Podman

```nix
{ config, pkgs, ... }:

{
  virtualisation.podman = {
    enable = true;
    dockerCompat = true;
  };

  virtualisation.oci-containers.backend = "podman";
}
```

### services.nix — Actual Budget, Caddy, Kanidm, TLS

```nix
{ config, pkgs, ... }:

{
  # Actual Budget container. Uses host networking so it can reach Kanidm on the
  # same machine — Podman's bridge network can't route to the Tailscale IP.
  # Listens on 15006 internally (ACTUAL_PORT) to avoid clashing with Caddy on 5006.
  virtualisation.oci-containers.containers.actual = {
    image = "actualbudget/actual-server:25.4.0";
    extraOptions = [ "--network=host" ];
    volumes = [ "/var/lib/actual:/data" ];
    autoStart = true;
    environment = {
      ACTUAL_PORT                   = "15006";
      ACTUAL_OPENID_DISCOVERY_URL   = "https://nixos.example.ts.net:8443/oauth2/openid/actual-budget";
      ACTUAL_OPENID_CLIENT_ID       = "actual-budget";
      ACTUAL_OPENID_SERVER_HOSTNAME = "https://nixos.example.ts.net:5006";
    };
    # Secret loaded from a file — never committed to git.
    # Create: sudo install -m 600 /dev/null /etc/actual/oidc-secret
    # Write:  ACTUAL_OPENID_CLIENT_SECRET=<value from `kanidm system oauth2 show-basic-secret actual-budget`>
    environmentFiles = [ "/etc/actual/oidc-secret" ];
  };

  systemd.tmpfiles.rules = [
    "d /var/lib/actual 0750 root root -"
    "d /var/lib/caddy/tls 0750 root caddy -"
  ];

  # Provisions a Tailscale-signed TLS cert before Caddy and Kanidm start.
  # Actual Budget requires HTTPS (SharedArrayBuffer won't work over plain HTTP).
  # Kanidm also requires TLS — it reads the same cert via the caddy group.
  systemd.services.tailscale-cert = {
    after = [ "tailscaled.service" "network-online.target" ];
    wants = [ "network-online.target" ];
    before = [ "caddy.service" "kanidm.service" ];
    wantedBy = [ "multi-user.target" ];
    serviceConfig = {
      Type = "oneshot";
      RemainAfterExit = true;
    };
    script = ''
      ${pkgs.tailscale}/bin/tailscale cert \
        --cert-file /var/lib/caddy/tls/cert.pem \
        --key-file /var/lib/caddy/tls/key.pem \
        nixos.example.ts.net
      chown root:caddy /var/lib/caddy/tls/cert.pem /var/lib/caddy/tls/key.pem
      chmod 640 /var/lib/caddy/tls/cert.pem /var/lib/caddy/tls/key.pem
    '';
  };

  # Tailscale certs expire after 90 days; renew weekly to stay well ahead.
  systemd.timers.tailscale-cert = {
    wantedBy = [ "timers.target" ];
    timerConfig = {
      OnCalendar = "weekly";
      Persistent = true;
    };
  };

  # Kanidm handles its own TLS on port 8443 — no Caddy proxy in front of it.
  # It reads the Tailscale cert via the caddy group.
  services.kanidm = {
    enableServer = true;
    package = pkgs.kanidm_1_9;
    serverSettings = {
      origin = "https://nixos.example.ts.net:8443";
      domain = "nixos.example.ts.net";
      bindaddress = "[::]:8443";
      tls_chain = "/var/lib/caddy/tls/cert.pem";
      tls_key = "/var/lib/caddy/tls/key.pem";
    };
  };

  users.users.kanidm.extraGroups = [ "caddy" ];

  systemd.services.kanidm.after = [ "tailscale-cert.service" ];
  systemd.services.kanidm.requires = [ "tailscale-cert.service" ];

  services.caddy = {
    enable = true;
    virtualHosts."nixos.example.ts.net:5006" = {
      extraConfig = ''
        tls /var/lib/caddy/tls/cert.pem /var/lib/caddy/tls/key.pem
        reverse_proxy localhost:15006
      '';
    };
  };

  networking.firewall.allowedTCPPorts = [ 5006 8443 ];
}
```

Also add `kanidm_1_9` to `environment.systemPackages` in `configuration.nix` — this gives you
the `kanidm` CLI for managing users and OAuth2 clients.

After rebuilding (`sudo nixos-rebuild switch`), continue with the manual steps below.

## Bootstrapping Kanidm

On first start, Kanidm generates an `idm_admin` password in the logs:

```bash
journalctl -u kanidm | grep "initial admin"
```

Log in with it:

```bash
kanidm login --name idm_admin
```

## Setting up the OAuth2 client in Kanidm

```bash
# Create the OAuth2 client
kanidm system oauth2 create actual-budget "Actual Budget" \
  https://nixos.example.ts.net:5006 --name idm_admin

# Add the OIDC redirect URL
kanidm system oauth2 add-redirect-url actual-budget \
  https://nixos.example.ts.net:5006/openid/callback --name idm_admin

# Actual Budget's openid-client library expects RS256; Kanidm defaults to ES256.
# Enable legacy RS256 signing for this client until Actual Budget adds ES256 support.
kanidm system oauth2 warning-enable-legacy-crypto actual-budget --name idm_admin

# Create a group for Actual Budget users
kanidm group create actual-budget-users --name idm_admin

# Map the required OIDC scopes to that group
kanidm system oauth2 update-scope-map actual-budget actual-budget-users \
  openid email profile --name idm_admin
```

## Creating users

```bash
kanidm person create tasdikrahman "Tasdik Rahman" --name idm_admin
kanidm person create partner "Partner Name" --name idm_admin

kanidm group add-members actual-budget-users tasdikrahman partner --name idm_admin
```

Set a password for each user via a credential reset token:

```bash
kanidm person credential create-reset-token tasdikrahman --name idm_admin
```

Visit the URL it outputs to set the password. Repeat for each user.

If Kanidm's account policy requires MFA and you want password-only login, lower the
credential type minimum:

```bash
kanidm group account-policy enable idm_all_persons --name idm_admin
kanidm group account-policy credential-type-minimum idm_all_persons any --name idm_admin
```

## Storing the client secret

Get the OAuth2 client secret Kanidm generated:

```bash
kanidm system oauth2 show-basic-secret actual-budget --name idm_admin
```

Write it to the secrets file on the server (never commit this to git):

```bash
sudo install -m 600 /dev/null /etc/actual/oidc-secret
echo "ACTUAL_OPENID_CLIENT_SECRET=<secret>" | sudo tee /etc/actual/oidc-secret
```

## Enabling OIDC in Actual Budget

Actual Budget requires a one-time script to enable OIDC in its database. First, complete
the initial bootstrap by visiting `https://nixos.example.ts.net:5006` and setting a server
password. Then run the enable script inside the container:

```bash
podman exec -it actual node /app/src/scripts/enable-openid.js
```

Restart the container:

```bash
sudo systemctl restart podman-actual
```

The login page should now show a "Login with OIDC" button alongside the password option.

## Gotchas

**`SharedArrayBuffer` error** — Actual Budget requires HTTPS. Plain HTTP won't work. The
Tailscale cert + Caddy setup above handles this.

**Podman bridge can't reach Kanidm** — When the container uses Podman's default bridge
network, it can't route to the Tailscale IP where Kanidm listens. The fix is
`--network=host`.

**Port conflict with `--network=host`** — Host networking means the container shares the
host's ports. Caddy already owns `5006`, so set `ACTUAL_PORT=15006` for the container and
proxy through Caddy.

**ES256 vs RS256** — Kanidm signs JWTs with ES256 (ECDSA). Actual Budget's OIDC client
currently expects RS256. The `warning-enable-legacy-crypto` command on the Kanidm side
is the workaround until [upstream support lands](https://github.com/actualbudget/actual/issues/5537).

**Kanidm package version** — On NixOS 25.11, use `pkgs.kanidm_1_9`. The plain `pkgs.kanidm`
and `pkgs.kanidm_1_7` packages are EOL.

**TLS cert timing** — Kanidm won't start without its TLS certs present. The `tailscale-cert`
systemd service runs first and the `requires` dependency on `kanidm.service` enforces the
ordering.
