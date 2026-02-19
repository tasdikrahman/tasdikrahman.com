---
title: "Introducing Kingsly — The Cert Manager"
description: "How an internal hackathon project led to the creation of GOJEK’s open-source certificate management tool"
tags: [security, x509, devops, ruby, rubyonrails, letsencrypt, oss, ssl]
comments: true
share: true
cover_image: '/content/images/2020/04/kingsly_header.jpeg'
---

> This was originally published under [Gojek's engineering blog](https://blog.gojekengineering.com/introducing-kingsly-the-cert-manager-ced40746aa65) by me, this post is a repost.

There’s one thing all devices connected to the Internet have in common — they rely on protocols called SSL/TLS to protect information in transit.

> SSL/TLS are cryptographic protocols designed to provide secure communication over insecure infrastructure.

Any communication over the public internet should be encrypted, for which we need SSL certificates. There are many cases for public communication in GOJEK as well. Some of them are listed below:

- Client VPN connection
- Inbound connections from mobile frontend to backend
- Portals exposed over the public Internet.
- Service to service communication over the public Internet

While the industry is moving towards a [zero trust network](https://cloud.google.com/beyondcorp/), certificate management has been a big pain for us.

This post details how we built Kingsly, GOJEK’s open source certificate management tool.

### Managing SSL/TLS certs until yesterday

> A certificate is a digital document that contains a public key, some information about the entity associated with it, and a digital signature from the certificate issuer.

In other words, it’s a shell that allows us to exchange, store and use public keys. With that, certificates become the basic building block of the Public Key Infrastructure

We use letsencrypt heavily at GOJEK, to generate SSL/TLS certificates for numerous use cases, which can range from putting them behind our HAproxy Loadbalancers, envoy proxies, IPSec VPN’s and a lot of other places.

As of the end of 2018, the whole setup of renewing the certificates (it’s basically regenerating the certificate in the case of letsencrypt) installed at different places, was a manual process.

All letsencrypt SSL certificates, including renewals, are valid for no more than 90 days from their issue date. Thirty days before the certificate expires you will begin receiving renewal notices.

<center><img src="/content/images/2020/04/kingsly_1.png"></center>

Seeing the above, someone from our team would renew these certificates manually and do the needful.

Although, this served our purpose for the time being, it was becoming difficult and time-consuming for one person to manage this piece of infrastructure with the increasing number of IPSec VPN’s.

Managing your own PKI infrastructure is a hard problem in itself and leaving it to manual processes always leaves room for human error.

The Kernel team, which I am part of, focuses on solving infrastructure problems, improving systems resiliency and developer productivity.

> Hence the motto: Productivity through Automation

### Problems with our current approach

We have faced a lot of issues around cert management:

- Using a wildcard certificate. If such certificate gets compromised, then all the subdomains are affected.
- Renewal of expired certificates.
- Certificate inventory. Keeping a list of hundreds of subdomains and maintaining expiry is an operational nightmare. Installation of letsencrypt locally on all the VMs makes it difficult to keep inventory at one central place.
- Manual generation/non-standardized way of generating and renewal of SSL Certs leaves room of human error
- Certs shared over email/slack.
- No audit trail.
- 100+ Certs scattered across org with little or no visibility of when is it expiring until one gets an email.


We need a solution to this problem — something very basic without a high learning curve.

### The features we wanted in our tool

- Certificates stored in a central manner
- APIs exposed to create/renew the certificate
- Automatic renewal a certain period before a cert expires
- Centralised tracking and notification
- Common API for internal users

### Enter Kingsly

It all started last December, when our team was contemplating what to ship for our internal company hackathon.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">It&#39;s going to be a looooong night here. 20+ teams taking part in our internal hackathon. <a href="https://twitter.com/gojektech?ref_src=twsrc%5Etfw">@gojektech</a> <br><br>Let the games, sorry, hacks begin 😉 <a href="https://t.co/6yRjfo4GX6">pic.twitter.com/6yRjfo4GX6</a></p>&mdash; Gojek Tech (@gojektech) <a href="https://twitter.com/gojektech/status/1070276401885052928?ref_src=twsrc%5Etfw">December 5, 2018</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Our second internal hackathon in Bangalore starts today! 😎 <br><br>We&#39;re pulling an all nighter starting at 4pm today to 12pm tomorrow noon. 😈<br><br>01101000 01100001 01100011 01101011 <a href="https://t.co/BBN3Filu5D">pic.twitter.com/BBN3Filu5D</a></p>&mdash; Gojek Tech (@gojektech) <a href="https://twitter.com/gojektech/status/1070205884947750912?ref_src=twsrc%5Etfw">December 5, 2018</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

This tool was a perfect candidate for the night as it was both:

- a pain point
- something which we really wanted to automate

By the end of the night, we had it in a working condition and generating SSL certificates was as simple as doing a

```
$ curl -X POST http://kingsly.host/v1/cert_bundles \
  -u admin:password \
  -H 'Content-Type: application/json' \
  -d '{
        "top_level_domain":"your-domain.com",
        "sub_domain": "your-sub-domain"
    }'
```

and the response from the Kingsly server would be:

```
'{
  "private_key":"-----BEGIN RSA PRIVATE KEY-----\nFOO...\n-----END RSA PRIVATE KEY-----\n",
  "full_chain":"-----BEGIN CERTIFICATE-----\nBAR...\n-----END CERTIFICATE-----\n"
}'
```

A simple JSON response which can be digested by a client the way it wants.

### We had a framework for the product. Now what?

We didn’t want this to end up as a hackathon project that remains a desolate creature in one’s source code repository. So we decided to extend Kingsly and make it into something which fit the initial set of requirements we had in mind for it.

### Moar features!!

After the cert generation part, the next feature which we wanted was automatic renewals of those certs.

The client (the IPSec client which we built) would keep polling the Kingsly server, get the certificates, and see if the current cert inside the IPSec VPN was different from the one which the server returned. If not, the client would replace the certs in the IPSec VPN box with the ones it received from the Kingsly server. It would then continue with the flow of tasks needed for the IPSec VPN to pick up the new certs, something which used to be done manually by a human.

<center><img src="/content/images/2020/04/kingsly_2.png"></center>

### Who gets to request generation of certs?

This was still an unsolved problem in the whole equation. We would need to devise a way in which we would be able to deny requests from clients.

One thing which we were sure about was: authentication and authorization should not be handled by the application.

A very simple solution was to put the Kingsly web server behind an HAproxy. Here, we would have a frontend rule pointing to a backend having an ACL with a list of IPs which would be allowed through.

This checked the initial problem of only allowing the clients which were whitelisted.

<script src="https://gist.github.com/tasdikrahman/e65c36f8156eeef855c5fc24196655bb.js"></script>

But the other problem would be maintaining the updated list of all the clients being allowed. On top of that, we were trying to automate a process. Hence, settling for something which eventually needed manual intervention didn’t quite fit our vision. So, we started exploring other possibilities and stumbled upon IAP, which fit the bill for our use case.

> Identity Aware Proxy (IAP) is the GCP provided solution for user as well as service authentication.

IAP is nothing but Oauth 2.0 implementation over a proxy.

For authentication with IAP, some required headers need to be added to every request which goes out of the client box. We created a proxy service, which adds the authentication details to the request. For service-to-service authentication, it uses OAuth 2.0 using creds of service accounts.

<center><img src="/content/images/2020/04/kingsly_3.png"></center>

### Why use IAP for auth?

- Central authorization layer
- Application level access control
- Allows individual and group-based access policies
- Enforces HTTPs
- Mandatorily redirects HTTP requests to HTTPS
- Attaches client identity to request headers for further processing by downstream applications/proxy


### There’s more to come

- Build support for HAproxy, envoy proxy and the other places where we require x.509 certificates which will be interacting with Kingsly, in the kingsly-certbot client.
- Have better monitoring to ensure whether certificates installed correctly.
- Allow authorization on top of Kingsly, to open it up to developers to enable them to request SSL certificates and usher the road towards SSL/TLS enabled applications.
- CRD to generate certs for applications inside our k8s clusters. We will be evaluating the use cases similar to how we did for Kingsly before going ahead with it.

As the saying goes, there’s no silver bullet in software and Kingsly checks off the list of requirements we had from a tool.

> Kingsly is open source! Check the links below 🖖

### Links

- [https://github.com/gojekfarm/kingsly](https://github.com/gojekfarm/kingsly) — the API server which handles cert generation and renewals.
- [https://github.com/gojekfarm/kingsly/tree/master/docs/deploy/k8s](https://github.com/gojekfarm/kingsly/tree/master/docs/deploy/k8s) — kingsly-server and worker deployment docs
- [https://github.com/gojekfarm/kingsly-certbot](https://github.com/gojekfarm/kingsly-certbot) — the IPSec VPN client for kingsly
- [https://github.com/gojekfarm/kingsly-certbot-cookbook](https://github.com/gojekfarm/kingsly-certbot-cookbook) — chef cookbook to setup certbot while provisioning an IPSec VPN.
- [https://github.com/gojekfarm/iap_auth](https://github.com/gojekfarm/iap_auth) — Proxy for talking to an IAP enabled service.
- [https://github.com/gojekfarm/iap_authenticator](https://github.com/gojekfarm/iap_authenticator) — Ruby Gem to create authentication token for services running behind IAP
- [https://github.com/gojekfarm/iap-auth-cookbook](https://github.com/gojekfarm/iap-auth-cookbook) — chef cookbook to setup up the iap-auth proxy binary on an IPSec VPN box.

Kingsly was also presented by me over at DevConf, India

<script async class="speakerdeck-embed" data-id="3e0b36f62907438094dde34369646e50" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

