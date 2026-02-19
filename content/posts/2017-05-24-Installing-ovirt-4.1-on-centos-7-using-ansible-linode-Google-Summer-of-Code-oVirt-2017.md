---
title: "Using Ansible Playbooks to Install oVirt 4.1 on centOS 7 (Linode)"
description: "Using Ansible Playbooks to Install oVirt 4.1 on centOS 7 (Linode)"
tags: [oss, gsoc, ansible]
comments: true
share: true
cover_image: '/content/images/2017/05/ansible-red-hat-blog-top.png'
---

In my [previous post](http://www.tasdikrahman.com/2017/05/21/Installing-ovirt-4.1-on-centos-7-digitalocean-Google-Summer-of-Code-oVirt-2017/), I played around how to install `ovirt-engine` on a remote VM.

Turns out we can automate the whole process using ansible playbooks!

<center><img src="/content/images/2017/05/060_net_00_featured.jpg"></center>

Secondly, I have thought of shifting from [digitalocean](https://digitalocean.com/) to [linode](https://linode.com). Why?

Well, first off. It's not that I don't like digitalocean. It's just that the prices for the VMS that I'am provisioning, are getting too high for me.

For a 4GB VM with 2 cores and 60GB of SSD to spare with. I am getting some 4TB of network I/O. 4TB for me is generous. All this adds up to a damage of $40/month or roughly speaking, $0.060/hour.

If you compare this to that of Linode's offerings. I am getting the same 4GB VM with 48GB SSD Storage. 2 CPU Cores and 3TB XFER. This costs $20/mo or (.03/hr)

That's like half of what I was paying to digitalocean for my servers! But the ease of use for digitalocean is pretty good. The only reason that I see if I want to use digitalocean for my VM's would be if I wanted low latency. As the closest datecenter to me for linode is in Singapore. While we digitalcoean has a datacenter around in bangalore.

Enough of me ranting around.

## Show me the Code

The playbook is already up there on [github](https://github.com/rhevm-qe-automation/ovirt-ansible/) which holds the ansible roles for quite a many things.

Having a quick look at them

```bash
$ ls -la roles
total 0
drwxr-xr-x  13 tasdik  tasdik  442 May 23 14:28 .
drwxr-xr-x  20 tasdik  tasdik  680 May 24 11:05 ..
-rw-r--r--   1 tasdik  tasdik    0 May 23 14:28 ansible.cfg
drwxr-xr-x   6 tasdik  tasdik  204 May 23 14:28 ovirt-collect-logs
drwxr-xr-x   6 tasdik  tasdik  204 May 23 14:28 ovirt-common
drwxr-xr-x   6 tasdik  tasdik  204 May 23 14:28 ovirt-engine-backup
drwxr-xr-x   7 tasdik  tasdik  238 May 23 14:28 ovirt-engine-cleanup
drwxr-xr-x   6 tasdik  tasdik  204 May 23 14:28 ovirt-engine-config
drwxr-xr-x   6 tasdik  tasdik  204 May 23 14:28 ovirt-engine-install-packages
drwxr-xr-x   7 tasdik  tasdik  238 May 23 14:28 ovirt-engine-remote-db
drwxr-xr-x   7 tasdik  tasdik  238 May 23 14:28 ovirt-engine-setup
drwxr-xr-x   7 tasdik  tasdik  238 May 23 14:28 ovirt-guest-agent
drwxr-xr-x   6 tasdik  tasdik  204 May 23 14:28 ovirt-iso-uploader-conf
```

I would be showing around the role `ovirt-engine-setup` in this post.

Provision your server and `ssh` into it.

I have created an entry on my `/etc/hosts` on my local dev box for the VM where I am going to install `ovirt-engine`.

```bash
$ cat /etc/hosts | grep gsoc
xxx.xxx.xx.xxx ovirtlinode.gsoc.org
```

`xx.xxx.xx.xxx` would be the public IP of your server.

You have to change the `hostname` of your server. Will explaing the why in a bit.

```bash
$ ssh root@ovirtlinode.gsoc.org
root@ovirtlinode.gsoc.org's password:
[root@ovirtlinode ~]# hostnamectl set-hostname ovirtlinode.gsoc.org
[root@ovirtlinode ~]# hostname
ovirtlinode.gsoc.org
[root@ovirtlinode ~]#
```

Now from your local dev box

```bash
$ git clone https://github.com/rhevm-qe-automation/ovirt-ansible/
$ cd ovirt-ansible/
$ touch site.yml inventory
```

The contents of the file above should be in the lines of

```bash
$ cat inventory
[all:vars]
ovirt_engine_type=ovirt-engine
ovirt_engine_version=4.1
# Make sure that link to release rpm is working!!!
ovirt_rpm_repo=http://resources.ovirt.org/pub/yum-repo/ovirt-release41.rpm
ovirt_engine_organization=ovirtlinode.gsoc.org
ovirt_engine_admin_password=secret

[engine]
ovirtlinode.gsoc.org ansible_ssh_user=root ansible_ssh_pass=secretpassword
$
$ cat site.yml
---
- hosts: engine
  roles:
    - role: ovirt-common
    - role: ovirt-engine-install-packages
    - role: ovirt-engine-setup
```

A quick note on the variable `ansible_ssh_pass=secretpassword`. This is not adviced! Don't do this as now obviously your ssh password is
stored in a text file which can be read by anyone. And god forbid if you add this file to version control accidentally and push it.

Rule of thumb is to always use ssh keys to run playbooks!

After this, we need to check the ansible fact `ansible_fqdn`. This one is important, as the we will be accessing the Admin panel of
ovirt-engine using the FQDN of this value. Otherwise our ansible playbook won't work.


```bash
$ ansible -m setup -i inventory engine -a 'filter=ansible_fqdn'
ovirtlinode.gsoc.org | SUCCESS => {
    "ansible_facts": {
        "ansible_fqdn": "ovirtlinode.gsoc.org"
    },
    "changed": false
}
```

Running the playbook, assuming you are on `ovirt-ansible/` dir

```bash
$ ansible-playbook site.yml -i inventory -vvv
```

This might take a few minutes to get done. Get a coffee or read some xkcd while it does its thing.

You can now access the admin panel through the url [https://ovirtlinode.gsoc.org/ovirt-engine/webadmin/](https://ovirtlinode.gsoc.org/ovirt-engine/webadmin/)

The url would obviously be different for you, citing the FQDN you decided to use during the installation process.

Cheerio!
