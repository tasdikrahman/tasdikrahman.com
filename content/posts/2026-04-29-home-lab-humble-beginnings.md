---
title: "Home Lab, Humble Beginnings"
date: 2026-04-29
draft: false
tags: ["homelab", "nixos", "self-hosting", "tailscale", "caddy"]
categories: ["infrastructure"]
description: "A walkthrough of my home server setup — hardware, NixOS, services like Actual Budget and Kanidm, and the trade-offs I made along the way."
---

## How It Started

A bit back, around 2020, I started hosting a small [Pi-hole](https://pi-hole.net) setup for DNS-based ad filtering. That was one of the setups which I had running for a bit longer before that, around 2017, when I had a DigitalOcean server where I would be running my own k8s cluster. I was using this k8s distribution called Typhoon, but it didn't really end up installing applications which I was using on a daily level.

A couple of my friends have their own home servers, and they were chatting about it for a bit. This was also an idea in the back of my head for some time, not to host something very serious, which would be mission critical for me to run, but in general just some small applications which I would try migrating off from which I use on a regular basis, and also something which I would want more control over.

## Finding the Right Box

And that's where it began for my home server setup, the beginnings, I must say, as it's just a single box. I started looking with different machines on eBay, Kleinanzeigen. After a few days of searching on and off, I did come across this one machine from Lenovo called The Think Center. It was selling for pretty cheap, so I ended up buying it from the last owner.

The model is a Lenovo ThinkCentre M75Q Generation 2 and comes with 8 GB of RAM and around 256 GB of space. It might seem less, but I just want to start with something rather than overthink too much about what configuration I would have wanted. Given that this ThinkCentre is configurable, I wasn't really concerned about extensibility.

From what I checked around, I can go maximum till 64 GB with 2 into 32 GB card slots. And for storage, I could have one on the primary slot and one attached to the secondary slot. From checking further, I could practically go all the way till 6 TB of NVMe storage on this machine. If required, I would always be able to attach an external drive through the USB. But again, something to figure out as I grow usage

The machine already came with a Wi-Fi dongle attached to it, which made my job easier, not having to wire it up on Ethernet on my home router, as the router sits in another room. I did look for repeaters, but it seemed a bit unnecessary at the current moment of setup until I really faced some issues with speed.

Here's what it looks like

<center><img src="/content/images/2026/04/homeserver-april-2026.jpeg"></center>

## Why NixOS

I have installed [NixOS](https://nixos.org) on my machine, and this is the first time I'm playing around with NixOS. To be honest, I really like the style of setup, with configuration being first-class and everything following from there.

This brings me to the point of me thinking of comparing it with other setups which I've tried for my home setup:
- Terraform
- running Ansible on a server
- other configuration engines like Chef


What really made me like the setup so far is Claude helping me out, first of all, in wrangling the Nix configuration, but also the readability of the syntax. The syntax is not something which I'm very familiar with, but Claude helped me wrangle on top of it to write and describe what I wanted. I hammered it further to to be modular and extensible in the way I wanted as of now what I also like about the setup is the builds for NixOS are just incremental, so I don't have to think about an LTS version, really like a Debian installation, and I can just keep upgrading. Since I didn't want to handle OS upgrades on a regular basis, where it would mean doing a fresh install.

But overall my pick for nixos was for setup as configuration being first class in the OS, and also incremental changes in general, plus me wanting to play around with nixos.

## Getting Connected

Now, connectivity to the home server was the next thing which I wanted to figure out. One thing I was sure of for now is that I didn't want to expose the home server to the internet to the internet. That opens a whole can of activities that you have to do to secure the server, and that was something which I didn't want to solve as of now. The first thing which I did after installing the bare minimum NixOS configuration was to set up [Tailscale](https://tailscale.com) on the machine and connect my personal laptop and the home server on the same network, getting ssh access solved.

The next immediate thing which I ended up doing was cloning the home server configuration repo of NixOS. The other thing which I did after that was just symlinking the directory to /etc/NixOS.

As of now, my flow is to make changes to git on my local machine; then that gets cloned over onto the home server. This is not automatic right now and then I have to run nixos rebuild switch. Probably something to check further on how to automate.

So that took care of connectivity and configuration in general on the home server, and the next bit was, of course, what was I going to set up on my machine?

## What I'm Running

I ended up setting a YNAB alternative called [Actual Budget](https://actualbudget.org). After that, for identity provider and handling user accounts and issuing OIDC tokens, I installed [Kanidm](https://kanidm.com) so that multiple members would have their own login ID inside my setup, which Actual Budget would be talking over to OIDC.

The reason why I ended up selecting my own oidc provider was because I just wanted to start using a self-hosted service for the oidc service on my home server and see how it works and just play around with it.

I ended up setting [Caddy](https://caddyserver.com) server for terminating HTTPS and reverse proxying to the containers which are running on the server. Another thing which I did was having Tail scale provide the TLS certificates for the hostname itself.

What I did next was to expose these services on my Tailscale server on different ports. I'm just using virtual hosts, and that is the configuration which I'm using as of now. The certificate being provided by Tailscale is just being used by the virtual host also at the same time.

So my usual flow now to use actual budget, for example on my phone, is to have my VPN configuration connect to my Tailscale network so that the Tailscale endpoint magic DNS is available. Then I go to the port where actual budget is bound, do a login.

That's a little bit about what kind of services I had set up and my reasoning behind choosing to set them up. Now I started the post by mentioning my setup for a pi hole, which I had done on a Raspberry Pi long back, which which was being used by all my devices which were connecting over Wi-Fi on my home router back then

## On DNS and Pi-hole

After setting up pihole, I added the home server private Tailscale endpoint to my Tailscale configuration's name server. I enabled the override DNS option so that all DNS requests on the network from each device would always use this name server for their DNS requests. Hence, Pi-hole coming into the picture and doing DNS-based ad blocking.

Since this setup would only query my Pi-hole setup when the device was connected to Tailscale, I could simply turn off my Tailscale VPN configuration on the device in case my Pi-hole was not working as expected or it was down for some reason. This would ensure my DNS would still work on my client device.

I recently stopped pihole and started using this service called [NextDNS](https://nextdns.io) instead and I'll share my reasoning behind it.

Which brings me to the point that I didn't want to run immediately something which I inherently rely on for the uptime quite a lot. If you notice, the other service called Actual Budget is just a bare minimum SQLite database which it uses, not a full-on setup with Redis or Postgres. I wanted to keep it very simple to begin with so that I am not spending a lot of time in the maintenance.

And something like DNS was something which I was thinking of offloading to a managed service provider. Given the cost for it was 19.9 euros for the whole year for any amount of devices, that seemed like a fair trade-off to make, again, I'm not arguing that you should not self-host by the way, pihole or ad guard and I might actually even reconsider this decision over time, but for now I just want to keep it very simple and at a bare minimum.

## What's Coming Next

Now, on what things I am thinking of hosting in the future. I was thinking of having a self-hosted version of Splitwise and a task manager running on the home server, potentially something which is written in go or Rust. The reason for that would be mostly just to be able to peek under the hood and be able to understand quickly. On why these two languages, Rust, I want to start learning/reading/writing and go I write for everyday things for myself and at work.

For future services, on my list to be added is paperless as for document management and potentially having an image server. For hosting my own photos, for that, I was thinking [Immich](https://immich.app). I was also mulling over the idea of running my own LLM on the server, which I can point to as a local model. For example, exploring the Gemma models from Google, but I have not yet had time to explore this more. Probably also something which I'll do after I increase the RAM, if I find it limiting when I try.

I'll share a post on how I set up actual budget with the configuration that it has, just for my own reference and also future work which I am putting on my home server.

## Backups

The number one reason I was not trying to put everything on to the server immediately was because I have not yet figured out backups on this machine. Not that any of the things which I've set up as of now are generating huge amounts of data or handling huge amounts of data, but this is a problem which will obviously come. Given I wanted to take an approach of a very laid-back setup where I would not have to deal with it day in and day out, this is something which I still have to figure out, and I think I'll most likely go with [restic](https://restic.net) for backups and parking it at either [Backblaze](https://www.backblaze.com) or a [Hetzner](https://www.hetzner.com)

I wanted to also share my nix OS configuration, which I have as of now and here's the link for it.

https://github.com/tasdikrahman/homeserver

## Ending notes

Finally have my home server setup, which I can play around with, and I can tell you that it does certainly give me joy when I'm working on it. I guess what I'll also figure out over time is how much complexity I want on my setup and what is the complexity which I want to deal with, which I feel is a semi-regular question you have to answer when self-hosting services, when would be something which you wanted to give off to a managed service provider, for example

But at the end, everything is a trade-off, and you, being the owner of your own home server, you should make that call for what you want to work with in general. I hope this post inspires somebody to take the leap on the personal home server space.




