---
title: "Installing oVirt 4.1 on centOS 7 (DigitalOcean)"
description: "Installing oVirt 4.1 on centOS 7 (DigitalOcean)"
tags: [oss, gsoc, ansible]
comments: true
share: true
cover_image: '/content/images/2017/05/OVirt-logo-highres.png'
---

Was trying to install oVirt engine on a VM deployed on DigitalOcean. My learnings from it are documented here.

## Installing oVirt Engine

I would concentrate on the part of just installing oVirt-engine as I had a fair share of problems while doing so.

The VM I am installing it on is a 4GB centOS 7 box with 80GB of SSD to spare for. Also, make sure you read through the whole requirements [mentioned on the official docs](http://www.ovirt.org/documentation/quickstart/quickstart-guide/#ovirt-engine) while going forward with this.

A quick run through of what oVirt is. If you have used vSphere by VMWare, this product offered by Redhat is a competitor to it.

The oVirt platform consists of at least one oVirt Engine and one or more Nodes.

- oVirt Engine provides a graphical user interface to manage the physical and logical resources of the oVirt infrastructure.
- oVirt Engine runs virtual machines.

oVirt Engine is the control center of the oVirt environment. It allows you to define hosts, configure data centers, add storage, define networks, create virtual machines, manage user permissions and use templates from one central location.

## Installation

```bash
[root@centos-4gb-blr1-ovirt-engine ~]# yum update
[root@centos-4gb-blr1-ovirt-engine ~]# yum install http://resources.ovirt.org/pub/yum-repo/ovirt-release41.rpm
[root@centos-4gb-blr1-ovirt-engine ~]# yum -y install ovirt-engine
[root@centos-4gb-blr1-ovirt-engine ~]# engine-setup
[ INFO  ] Stage: Initializing
[ INFO  ] Stage: Environment setup
          Configuration files: ['/etc/ovirt-engine-setup.conf.d/10-packaging-jboss.conf', '/etc/ovirt-engine-setup.conf.d/10-packaging.conf']
          Log file: /var/log/ovirt-engine/setup/ovirt-engine-setup-20170520124451-wvuuny.log
          Version: otopi-1.6.1 (otopi-1.6.1-1.el7.centos)
[ INFO  ] Stage: Environment packages setup
[ INFO  ] Stage: Programs detection
[ INFO  ] Stage: Environment setup
[ INFO  ] Stage: Environment customization

          --== PRODUCT OPTIONS ==--

          Configure Engine on this host (Yes, No) [Yes]:
          Configure Image I/O Proxy on this host? (Yes, No) [Yes]:
          Configure WebSocket Proxy on this host (Yes, No) [Yes]:
          Please note: Data Warehouse is required for the engine. If you choose to not configure it on this host, you have to configure it on a remote host, and then configure the engine on this host so that it can access the database of the remote Data Warehouse host.
          Configure Data Warehouse on this host (Yes, No) [Yes]:
          Configure VM Console Proxy on this host (Yes, No) [Yes]:

          --== PACKAGES ==--

[ INFO  ] Checking for product updates...
[ INFO  ] No product updates found

          --== NETWORK CONFIGURATION ==--

          Host fully qualified DNS name of this server [centos-4gb-blr1-ovirt-engine]: ovirt.gsoc.org
[WARNING] Failed to resolve ovirt.gsoc.org using DNS, it can be resolved only locally
          Setup can automatically configure the firewall on this system.
          Note: automatic configuration of the firewall may overwrite current settings.
          Do you want Setup to configure the firewall? (Yes, No) [Yes]:
          The following firewall managers were detected on this system: firewalld
          Firewall manager to configure (firewalld):
[ ERROR ] Invalid value
          Firewall manager to configure (firewalld):
[ ERROR ] Invalid value
          Firewall manager to configure (firewalld): firewalld
[ INFO  ] firewalld will be configured as firewall manager.

          --== DATABASE CONFIGURATION ==--

          Where is the DWH database located? (Local, Remote) [Local]:
          Setup can configure the local postgresql server automatically for the DWH to run. This may conflict with existing applications.
          Would you like Setup to automatically configure postgresql and create DWH database, or prefer to perform that manually? (Automatic, Manual) [Automatic]:
          Where is the Engine database located? (Local, Remote) [Local]:
          Setup can configure the local postgresql server automatically for the engine to run. This may conflict with existing applications.
          Would you like Setup to automatically configure postgresql and create Engine database, or prefer to perform that manually? (Automatic, Manual) [Automatic]:

          --== OVIRT ENGINE CONFIGURATION ==--

          Engine admin password:
          Confirm engine admin password:
[WARNING] Password is weak: it is based on a dictionary word
          Use weak password? (Yes, No) [No]: Yes
          Application mode (Virt, Gluster, Both) [Both]:

          --== STORAGE CONFIGURATION ==--

          Default SAN wipe after delete (Yes, No) [No]:

          --== PKI CONFIGURATION ==--

          Organization name for certificate [gsoc.org]:

          --== APACHE CONFIGURATION ==--

          Setup can configure the default page of the web server to present the application home page. This may conflict with existing applications.
          Do you wish to set the application as the default page of the web server? (Yes, No) [Yes]:
          Setup can configure apache to use SSL using a certificate issued from the internal CA.
          Do you wish Setup to configure that, or prefer to perform that manually? (Automatic, Manual) [Automatic]:

          --== SYSTEM CONFIGURATION ==--

          Configure an NFS share on this server to be used as an ISO Domain? (Yes, No) [No]:

          --== MISC CONFIGURATION ==--

          Please choose Data Warehouse sampling scale:
          (1) Basic
          (2) Full
          (1, 2)[1]:

          --== END OF CONFIGURATION ==--

[ INFO  ] Stage: Setup validation
[WARNING] Cannot validate host name settings, reason: resolved host does not match any of the local addresses
[WARNING] Less than 16384MB of memory is available

          --== CONFIGURATION PREVIEW ==--

          Application mode                        : both
          Default SAN wipe after delete           : False
          Firewall manager                        : firewalld
          Update Firewall                         : True
          Host FQDN                               : ovirt.gsoc.org
          Configure local Engine database         : True
          Set application as default page         : True
          Configure Apache SSL                    : True
          Engine database secured connection      : False
          Engine database user name               : engine
          Engine database name                    : engine
          Engine database host                    : localhost
          Engine database port                    : 5432
          Engine database host name validation    : False
          Engine installation                     : True
          PKI organization                        : gsoc.org
          DWH installation                        : True
          DWH database secured connection         : False
          DWH database host                       : localhost
          DWH database user name                  : ovirt_engine_history
          DWH database name                       : ovirt_engine_history
          DWH database port                       : 5432
          DWH database host name validation       : False
          Configure local DWH database            : True
          Configure Image I/O Proxy               : True
          Configure VMConsole Proxy               : True
          Configure WebSocket Proxy               : True

          Please confirm installation settings (OK, Cancel) [OK]:
[ INFO  ] Stage: Transaction setup
[ INFO  ] Stopping engine service
[ INFO  ] Stopping ovirt-fence-kdump-listener service
[ INFO  ] Stopping dwh service
[ INFO  ] Stopping Image I/O Proxy service
[ INFO  ] Stopping vmconsole-proxy service
[ INFO  ] Stopping websocket-proxy service
[ INFO  ] Stage: Misc configuration
[ INFO  ] Stage: Package installation
[ INFO  ] Stage: Misc configuration
[ INFO  ] Upgrading CA
[ INFO  ] Initializing PostgreSQL
[ INFO  ] Creating PostgreSQL 'engine' database
[ INFO  ] Configuring PostgreSQL
[ INFO  ] Creating PostgreSQL 'ovirt_engine_history' database
[ INFO  ] Configuring PostgreSQL
[ INFO  ] Creating CA
[ INFO  ] Creating/refreshing Engine database schema
[ INFO  ] Creating/refreshing DWH database schema
[ INFO  ] Configuring Image I/O Proxy
[ INFO  ] Setting up ovirt-vmconsole proxy helper PKI artifacts
[ INFO  ] Setting up ovirt-vmconsole SSH PKI artifacts
[ INFO  ] Configuring WebSocket Proxy
[ INFO  ] Creating/refreshing Engine 'internal' domain database schema

[ INFO  ] Generating post install configuration file '/etc/ovirt-engine-setup.conf.d/20-setup-ovirt-post.conf'
[ INFO  ] Stage: Transaction commit
[ INFO  ] Stage: Closing up
[ INFO  ] Starting engine service
[ INFO  ] Starting dwh service
[ INFO  ] Restarting ovirt-vmconsole proxy service

          --== SUMMARY ==--

[ INFO  ] Restarting httpd
          Please use the user 'admin@internal' and password specified in order to login
          Web access is enabled at:
              http://ovirt.gsoc.org:80/ovirt-engine
              https://ovirt.gsoc.org:443/ovirt-engine
          Internal CA 80:BD:83:EA:FB:EB:F8:BD:F5:22:98:F2:90:57:03:92:62:B2:5A:62
          SSH fingerprint: ee:43:05:9e:95:cc:7e:bb:6a:6e:aa:28:0d:4d:c1:08
[WARNING] Less than 16384MB of memory is available

          --== END OF SUMMARY ==--

[ INFO  ] Stage: Clean up
          Log file is located at /var/log/ovirt-engine/setup/ovirt-engine-setup-20170520124451-wvuuny.log
[ INFO  ] Generating answer file '/var/lib/ovirt-engine/setup/answers/20170520131443-setup.conf'
[ INFO  ] Stage: Pre-termination
[ INFO  ] Stage: Termination
[ INFO  ] Execution of setup completed successfully
[root@centos-4gb-blr1-ovirt-engine ~]#
```

The most important thing to note above in the `engine-setup` command is the FQDN for the VM. You need this for accessing the Engine administration portal.

Take a look at this specific part

```bash
...
[ INFO  ] Restarting httpd
          Please use the user 'admin@internal' and password specified in order to login
          Web access is enabled at:
              http://ovirt.gsoc.org:80/ovirt-engine
              https://ovirt.gsoc.org:443/ovirt-engine
...
```

For resolving the admin portal, we need to add an entry inside `/etc/hosts` inside the VM first.

```bash
[root@centos-4gb-blr1-ovirt-engine ~]# cat /etc/hosts
# Your system has configured 'manage_etc_hosts' as True.
# As a result, if you wish for changes to this file to persist
# then you will need to either
# a.) make changes to the master file in /etc/cloud/templates/hosts.redhat.tmpl
# b.) change or remove the value of 'manage_etc_hosts' in
#     /etc/cloud/cloud.cfg or cloud-config from user-data
# The following lines are desirable for IPv4 capable hosts
127.0.0.1 centos-4gb-blr1-ovirt-engine centos-4gb-blr1-ovirt-engine
127.0.0.1 localhost.localdomain localhost
127.0.0.1 localhost4.localdomain4 localhost4

# The following lines are desirable for IPv6 capable hosts
::1 centos-4gb-blr1-ovirt-engine centos-4gb-blr1-ovirt-engine
::1 localhost.localdomain localhost
::1 localhost6.localdomain6 localhost6

xxx.xx.xx.xxx ovirt.gsoc.org
[root@centos-4gb-blr1-ovirt-engine ~]#
```

Where xxx.xx.xx.xxx would be the public IP of my VM.

So if you now do a `ping` to `ovirt.gsoc.org`, out local DNS would be successfully able to resolve the FQDN

```bash
[root@centos-4gb-blr1-ovirt-engine ~]# ping ovirt.gsoc.org
PING ovirt.gsoc.org (xxx.xx.xx.xxx) 56(84) bytes of data.
64 bytes from ovirt.gsoc.org (xxx.xx.xx.xxx): icmp_seq=1 ttl=64 time=0.034 ms
64 bytes from ovirt.gsoc.org (xxx.xx.xx.xxx): icmp_seq=2 ttl=64 time=0.041 ms
64 bytes from ovirt.gsoc.org (xxx.xx.xx.xxx): icmp_seq=3 ttl=64 time=0.033 ms
^C
--- ovirt.gsoc.org ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1999ms
rtt min/avg/max/mdev = 0.033/0.036/0.041/0.003 ms
[root@centos-4gb-blr1-ovirt-engine ~]#
```

Now on the local dev box, I did the same thing with my `/etc/hosts`

```bash
$ cat /etc/hosts
...
# DO server
xxx.xx.xx.xxx ovirt.gsoc.org
...
```

Now it's resolvable from my local dev box too

```bash
$ ping ovirt.gsoc.org
PING ovirt.gsoc.org (xxx.xx.xx.xxx): 56 data bytes
64 bytes from xxx.xx.xx.xxx: icmp_seq=0 ttl=59 time=5.322 ms
^C
--- ovirt.gsoc.org ping statistics ---
1 packets transmitted, 1 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 5.322/5.322/5.322/0.000 ms
```

## Admin portal

Now on your dev box, check the admin panel by going to the url https://ovirt.gsoc.org:443/ovirt-engine

<center><img src="/content/images/2017/05/ovirt-admin-landingpage_840x610.png"></center>

Let's take the admin panel for a spin

<center><img src="/content/images/2017/05/ovirt-admin-panel.png"></center>

Isn't she pretty?

Had been fighting with some dumb defaults for almost an hour. It's 5 am. in the morning. So I need to get some sleep. Stay tuned!

## Debugging tips

- make sure its (the remote host where you are installing `ovirt-engine`) resolvable from your machine.
- check if the engine is running (check `/var/log/engine.log` and `ovirt-engine` service)
- check firewall on the machine.
- try to connect to the web browser from the virtual machine to be sure its not network issue between you and VM `ping your.vm.com` should resolve that ip address

## Links

- [http://www.ovirt.org/documentation/quickstart/quickstart-guide/](http://www.ovirt.org/documentation/quickstart/quickstart-guide/)
