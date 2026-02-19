---
title: "Introduction to Configuration Management using Ansible"
description: "Introduction to Configuration Management using Ansible"
tags: [python, devops, ansible]
comments: true
share: true
cover_image: '/content/images/2017/02/ansible-logo600.png'
---

## Need for Configuration management

There are many devs/sysadmins out there who manage their servers by logging in through ssh. Making the changes and then logging out again. Sounds like you?

Well hey, you are not alone!

But do you feel that this can create snowflake servers?

Servers which are impossible to recreate because we missed out on some minute detail which the other dev had known.

> But Tasdik. This wouldn't happen if we have a very good documentation process giving a step by step guide on how to do so!

Good! I will say you guys have followed a very good engineering practice of documenting each and every other process that you do! But in most fast moving
environments. This is not the case!

## Enter Configuration management

It's good that we have a good range of config. management tools out there like [CFEngine](http://cfengine.com/), [Ansible](http://ansible.com/), [chef](http://getchef.com/chef) to name a few.

## Isn't this DevOps thing some buzzword out there?

Let me tell you a first hand experience of mine which really made me think about provisioning tools.

I am currently working on a project which deals with `OpenCV` 3 and some python dependencies thrown around. We have currently deployed the demo app on a humble 512MB RAM, 20 gig SSD on a DigitalOcean droplet which runs Xenial Xerius(64 bit).

The general process of getting the development env up and running for OpenCV involves
- Updating and upgrading the dist packages
- installing developer toolchains for compiling and installing the `C++` source code
- installing tons of libraries for image and video formats
- getting the actual source code of OpenCV and OpenCV contrib which we are gonna use
- Build a particular version of OpenCV required by us.
- Compiling it

And the list goes on!

## But you can still write a shell script for it. Right?

Yes! You absolutely can. No doubt.

I will go further one step and say that it DOES take time and resources to learn a config management tool and have some working knowlege to be productive with it.

But here are some points in favor for config mgmt. tools

- They can be re-used, distributed. Chef has cookbooks, juju has charms, Perl has CPAN, Java has Maven.
- The DSL's do take away some freedom, but they are much cleaner.
- **Idempotency**: Which means you can safely re-run it any number of times and each time it will go to the desired state, remain there or more closer to the desired state.
- Scalable!
- OS Agnostic.
- Version Controlling - In short, maintaining Infrastructure as Code.
- Easy to write

This is an example `ansible` role which updates the `apt-cache` for target server.

```
---
- name: Updating the apt-cache
  become: yes
  apt: update_cache=true
```

Looks good?


## Why Ansible?

I just wanted to get started with some or the other tool! That's it.

I am pretty sure that `chef`, `puppet`, `salt stack` and the like are equally good and will serve your use case. So do check them out too and have a feel around the different hammers around the market.

But one great thing about Ansible is that you don't need a PKI architecture or some special communication protocol for managing it's nodes. It(your manager) just uses plain on SSH for communicating with it's nodes. Although for older versions of ansible, it used to communicate using the paramiko SSH-2 python implementation

## A simple comparison

Nothing much, just a simple `apache2` virtual hosts setup for your VPS.

#### Manual install

```
$ sudo apt-get update && upgrade
$ sudo apt-get install apache2
$ cd /etc/apache2
/etc/apache2 $ sudo cp /files/awesome-app sites-available/awesome-app.conf
/etc/apache2 $ sudo chmod 640 sites-available/awesome-app.conf
/etc/apache2 $ sudo rm /etc/apache2/sites-enabled/default && /etc/apache2/sites-enabled/default-ssl
/etc/apache2 $ sudo ln -s /etc/apache2/sites-available/awesome-app /etc/apache2/sites-enabled/awesome-app
/etc/apache2 $ sudo service apache2 restart

```

#### Shell script

You can put all the necessary commands above and put it inside a `provision.sh` file and then call it using `$ sh provision.sh`

#### Ansible playbook

```
- hosts: web
  become: yes  # for escalated privileges
  tasks:
    - name: Installs apache web server
      apt: pkg=apache2 state=installed update_cache=true

    - name: Push default virtual host configuration
      copy: src=files/awesome-app dest=/etc/apache2/sites-available/awesome-app mode=0640

    - name: Disable the default virtualhost
      file: dest=/etc/apache2/sites-enabled/default state=absent
      notify:
        - restart apache

    - name: Disable the default ssl virtualhost
      file: dest=/etc/apache2/sites-enabled/default-ssl state=absent
      notify:
        - restart apache

    - name: Activates our virtualhost
      file: src=/etc/apache2/sites-available/awesome-app dest=/etc/apache2/sites-enabled/awesome-app state=link
      notify:
        - restart apache

  handlers:
    - name: restart apache
      service: name=apache2 state=restarted
```

The above configuration is just a simple POC showing the general relative ease of using tools like `Ansible`. You can do a lot more like provisioning full blown distributed servers from scratch, load balance them by putting a load balancer like HAProxy in front of it and what not!

So it's left on you to decide what suits you best.

Cheers!

On a side note, I automated installing and setting up the dev environment for OpenCV 3 for python3 using Ansible. The code as usual for every other project lies in [github](https://github.com/tasdikrahman/opencv3-ansible-vagrant-playbook)

