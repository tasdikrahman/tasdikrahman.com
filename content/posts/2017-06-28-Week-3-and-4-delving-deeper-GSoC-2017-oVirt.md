---
title: "Week 3 and 4, GSoC 2017 - dozens of cloud vm's, ansibling, finding bugs, testing"
description: "Week 3 and 4, GSoC 2017 - dozens of cloud vm's, ansibling, finding bugs, testing"
tags: [oss, python, ansible, gsoc]
comments: true
share: true
cover_image: '/content/images/2017/05/gsoc-cover.png'
---

At last I have got a hold of IRC's and I declare my love for [irssi](https://irssi.org/). The combination of [tmux](https://tmux.github.io/) and irssi is a boon for me. I tried different clients like [weechat](http://www.weechat.org/),[Epic](http://www.epicsol.org//) but I found irssi to be more appealing, quite frankly use any of these two if you are looking for something using which you can chat on irc's on the terminal. 

My current installation of irssi is there on my VPC hosted on linode. For logging my chat's and conversations, I have setup logrotate daemon to log the channels and private chats in their respective directories.

So what does it look like? Definitely a huge shift from the web chat interface and more customisable.

<center><img src="/content/images/2017/06/irssi_tmux.png"></center>

I love the current setup. Just one thing which I think is missing out from here is that I would like to get a notification when I get a message or someone mentions my name in a channel. Will have to look that up.

You can try [Limechat](limechat.net/mac/) if you are on a MAC.

Between, I will be there with the handle `tasdikrahman` hanging around the channels #ovirt on OFTC as well as on freenode, mostly #ansible and #gsoc.

Anyways, that's for my recent addition of irc love.

Talking about work, I have been working on my existing PR looking into the feedback which my mentor and other members of the infra team have given. 

And right now, I am working on adding playbooks for configuring a remote DWH on an ovirt-engine setup.

## Prelude

Reading up the docs, the DWH is a historic database that allows users to create reports over a static API using business intelligence suites that enable you to monitor the system. It contains the ETL (Extract Transform Load) process created using Talend Open Studio and DB scripts to create a working history DB. 

This history database(`ovirt_engine_history` to be precise) can be utilized by any application to extract a range of information at the data center, cluster, and host levels.

Simply put, it's a BI tool for your ovirt installation.

<center><img src="/content/images/2017/06/ovirt-arch.png"></center>

oVirt Engine uses PostgreSQL 8.4.x as the database platform to store information about the state of the virtualization environment, its configuration and performance. At install time, ovirt engine creates a PostgreSQL database called `engine`. You have the option to either install this on the same host or a different host(remote)

The `ovirt-engine-dwh` package creates a second database called `ovirt_engine_history`, which contains historical configuration information and statistical metrics collected every minute over time from the engine operational database. Tracking the changes to the database provides information on the objects in the database, enabling the user to analyze activity, enhance performance, and resolve difficulties.

You can track [configuration data](http://www.ovirt.org/documentation/data-warehouse/Tracking_configuration_history/), [statistical data](http://www.ovirt.org/documentation/data-warehouse/Recording_statistical_history/) into your `ovirt_engine_history` database based on your needs. 

## But why should someone run the DWH service on a different host?

Quite obviously running a whole lot of services on one host would be very memory heavy. And what happens when someone tries to squeeze out a lot of things from a single instance? 

There is a decrease in performance output due to fight amongst the processes for resources. So it's very logical to distribute your services to different hosts if possible.

<center><img src="/content/images/2017/06/ovirt-two-box-arch.jpg"></center>

The above architecture basically shows the simple setup of the ovirt-engine and dwh all being on the same host.

Now even when you are trying to seperate out the load by distributing the services running in a host, you further have a flexibily offered by oVirt here. 

Some being. 

1. `engine` db itself being on a differnt host rather than on the host where ovirt-engine is running.
2. DWH being installed on a different host than the one where ovirt-engine is installed.(feature being available only after oVirt Engine 3.5)
3. DWH being installed on a different host and the DWH db being on a 3rd host.

The first one is covered by the ansible role [ovirt-engine-remote-db](https://github.com/rhevm-qe-automation/ovirt-ansible/tree/master/roles/ovirt-engine-remote-db) which lets you do so

I tried tackling the 2nd one on the list in my 3rd week.

<center><img src="/content/images/2017/06/ovirt-one-box-arch.jpg"></center>

Had the initial discussions about [issue #9](https://github.com/rhevm-qe-automation/ovirt-ansible/issues/9) with Lukas and we narrowed down to the end goals and tasks for that particular issue on github.

The docs are pretty clear on the process itself and a bit of poring over them made the things which were to be done.

As I had mentioned in my [earlier post](https://medium.com/@tasdikrahman/week-1-and-2-gsoc-2017-travel-code-good-food-1f91d34344b9) about not automating something which you haven't achieved manually yet. 

That applied here too. 

For starters, I rebuilt 2x2gig Centos7 boxes on Linode which were hardened using my own custom ansible playbook called [ansible-bootstrap-server](https://github.com/tasdikrahman/ansible-bootstrap-server). I had a good lesson about security when [one of my servers got compromised](https://edgecoders.com/learnings-from-analyzing-my-compromised-server-linode-cd3be62dc286) so this one was a must. 

## Some assumptions for the two systems

<center><img src="/content/images/2017/06/vm-assumptions.jpg"></center>

Expanding on the first one, it should be able to do so by using the FQDN's of the respective VM's in question.

To add to the above, 

- Ports 80, 443, 5432(`postgres`) should be open on both these hosts. 

You can open the above ports using `firewalld` using (assuming you are `root`)

```bash
$ firewall-cmd --zone=public --add-port=80/tcp --permanent
$ # further adding any rules to appended to iptables
$ firewalld-cmd --reload  # for the rules to take effect
$ iptables -S  # to check whether the changes have taken effect
```

`iptables` would spit every rule defined out there for your system. That would get ugly real quick. A cleaner way would be to do a 

```bash
[root@dwhtest-3-engine ~]# iptables-save | grep 443
-A IN_public_allow -p tcp -m tcp --dport 443 -m conntrack --ctstate NEW -j ACCEPT
[root@dwhtest-3-engine ~]#
```

After these, the machine is just set for configuration.

## Manual installation

Install and setup ovirt-engine on machine A, ovirt-engine-dwh on machine B, see that dwhd on B collects data from the engine on A.

On A:

#### Installing DWH on a remote host from the one having ovirt-engine

```bash
[root@dwhtest-3-engine ~]# hostname
dwhtest-3-engine.ovirt.org
[root@dwhtest-3-engine ~]# yum install http://resources.ovirt.org/pub/yum-repo/ovirt-release41.rpm
[root@dwhtest-3-engine ~]# yum -y install ovirt-engine
[root@dwhtest-3-engine ~]# engine-setup
[ INFO  ] Stage: Initializing
[ INFO  ] Stage: Environment setup
          Configuration files: ['/etc/ovirt-engine-setup.conf.d/10-packaging-jboss.conf', '/etc/ovirt-engine-setup.conf.d/10-packaging.conf']
          Log file: /var/log/ovirt-engine/setup/ovirt-engine-setup-20170625105958-cyvseu.log
          Version: otopi-1.6.2 (otopi-1.6.2-1.el7.centos)
[ INFO  ] Stage: Environment packages setup
[ INFO  ] Stage: Programs detection
[ INFO  ] Stage: Environment setup
[ INFO  ] Stage: Environment customization

          --== PRODUCT OPTIONS ==--

          Configure Engine on this host (Yes, No) [Yes]:
          Configure Image I/O Proxy on this host? (Yes, No) [Yes]:
          Configure WebSocket Proxy on this host (Yes, No) [Yes]:
          Please note: Data Warehouse is required for the engine. If you choose to not configure it on this host, you have to configure it on a remote host, and then configure the engine on this host so that it can access the database of the remote Data Warehouse host.
          Configure Data Warehouse on this host (Yes, No) [Yes]: No
          Configure VM Console Proxy on this host (Yes, No) [Yes]:

          --== PACKAGES ==--

[ INFO  ] Checking for product updates...
[ INFO  ] No product updates found

          --== NETWORK CONFIGURATION ==--

          Host fully qualified DNS name of this server [dwhtest-3-engine.ovirt.org]:
[WARNING] Failed to resolve dwhtest-3-engine.ovirt.org using DNS, it can be resolved only locally
          Setup can automatically configure the firewall on this system.
          Note: automatic configuration of the firewall may overwrite current settings.
          Do you want Setup to configure the firewall? (Yes, No) [Yes]:
          The following firewall managers were detected on this system: firewalld
          Firewall manager to configure (firewalld): firewalld
[ INFO  ] firewalld will be configured as firewall manager.

          --== DATABASE CONFIGURATION ==--

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

          Organization name for certificate [ovirt.org]:

          --== APACHE CONFIGURATION ==--

          Setup can configure the default page of the web server to present the application home page. This may conflict with existing applications.
          Do you wish to set the application as the default page of the web server? (Yes, No) [Yes]:
          Setup can configure apache to use SSL using a certificate issued from the internal CA.
          Do you wish Setup to configure that, or prefer to perform that manually? (Automatic, Manual) [Automatic]:

          --== SYSTEM CONFIGURATION ==--

          Configure an NFS share on this server to be used as an ISO Domain? (Yes, No) [No]:

          --== MISC CONFIGURATION ==--


          --== END OF CONFIGURATION ==--

[ INFO  ] Stage: Setup validation
[WARNING] Cannot validate host name settings, reason: resolved host does not match any of the local addresses
[WARNING] Warning: Not enough memory is available on the host. Minimum requirement is 4096MB, and 16384MB is recommended.
          Do you want Setup to continue, with amount of memory less than recommended? (Yes, No) [No]: Yes

          --== CONFIGURATION PREVIEW ==--

          Application mode                        : both
          Default SAN wipe after delete           : False
          Firewall manager                        : firewalld
          Update Firewall                         : True
          Host FQDN                               : dwhtest-3-engine.ovirt.org
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
          PKI organization                        : ovirt.org
          DWH installation                        : False
          Configure local DWH database            : False
          Configure Image I/O Proxy               : True
          Configure VMConsole Proxy               : True
          Configure WebSocket Proxy               : True

          Please confirm installation settings (OK, Cancel) [OK]:
[ INFO  ] Stage: Transaction setup
[ INFO  ] Stopping engine service
[ INFO  ] Stopping ovirt-fence-kdump-listener service
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
[ INFO  ] Creating CA
[ INFO  ] Creating/refreshing Engine database schema
[ INFO  ] Configuring Image I/O Proxy
[ INFO  ] Setting up ovirt-vmconsole proxy helper PKI artifacts
[ INFO  ] Setting up ovirt-vmconsole SSH PKI artifacts
[ INFO  ] Configuring WebSocket Proxy
[ INFO  ] Creating/refreshing Engine 'internal' domain database schema
[ INFO  ] Generating post install configuration file '/etc/ovirt-engine-setup.conf.d/20-setup-ovirt-post.conf'
[ INFO  ] Stage: Transaction commit
[ INFO  ] Stage: Closing up
[ INFO  ] Starting engine service
[ INFO  ] Restarting ovirt-vmconsole proxy service

          --== SUMMARY ==--

[ INFO  ] Restarting httpd
          Please use the user 'admin@internal' and password specified in order to login
          The engine requires access to the Data Warehouse database.
          Data Warehouse was not set up. Please set it up on some other machine and configure access to it on the engine.
          Web access is enabled at:
              http://dwhtest-3-engine.ovirt.org:80/ovirt-engine
              https://dwhtest-3-engine.ovirt.org:443/ovirt-engine
          Internal CA 1F:E4:07:AF:E2:63:27:8F:4A:E2:A1:8D:F2:63:9B:BA:29:F7:3D:21
          SSH fingerprint: 38:ca:86:6d:82:50:ca:c7:9c:03:ed:bc:0b:3a:e5:33
[WARNING] Warning: Not enough memory is available on the host. Minimum requirement is 4096MB, and 16384MB is recommended.

          --== END OF SUMMARY ==--

[ INFO  ] Stage: Clean up
          Log file is located at /var/log/ovirt-engine/setup/ovirt-engine-setup-20170625105958-cyvseu.log
[ INFO  ] Generating answer file '/var/lib/ovirt-engine/setup/answers/20170625112418-setup.conf'
[ INFO  ] Stage: Pre-termination
[ INFO  ] Stage: Termination
[ INFO  ] Execution of setup completed successfully
[root@dwhtest-3-engine ~]#
```

So ovirt-engine is set on machine A.

The only difference here being when running `engine-setup` on the host, answering No to configuring Data Warehouse:

```bash
...
Configure Data Warehouse on this host (Yes, No) [Yes]: No
...
```

The other thing to note in the above installation is the `answerfile` which was generated out of the installation process.

## Answerfile?

Tools like `engine-setup`, `engine-cleanup` and `ovirt-engine-rename` which are based out on OTOPI(key value pair store of data and configuration), all generate something called as an answerfile upon completion of their (be it successful or erroneous). 

This file is placed under `/var/lib/ovirt-engine/setup/answers/` ending with a `*.conf`.

<center><img src="/content/images/2017/06/state0-to-state1.jpg"></center>

So have system A which is present on some state0, "S0" in this sense includes basically everything relevant - versions of relevant packages, enabled repos, history (such as previous runs of these tools, other data accumulated over time, etc), other manual configuration, etc.

The expected way to use these answer files is:

1. Have a system A in some state S0
2. Run one of the tools interactively, answer its questions as needed, let it create an answer file Ans1. Basically answering all the places where the program was waiting for `STDIN`.
3. System A is now in state S1.
4. Have some other system B in state S0, that you want to bring to state S1.
5. Run there the same tool with `–config-append=Ans1`

> When used this way, the tools should run unattended. If they still ask questions, it's generally considered a bug.

Which allows and makes way for automation.

Manually editing such an answer file is generally not supported/expected and should not be needed. You might do that to achieve special non-standard goals. If you do that, you should thoroughly verify that it works for you, and use in a controlled environment - same known initial state, same versions of relevant stuff, etc.

## Leveraging the things seen in this answerfile

We already have in place an ansible role for installing ovirt-engine along with DWH on the same host here named [ovirt-engine-setup](https://github.com/rhevm-qe-automation/ovirt-ansible/tree/master/roles/ovirt-engine-setup).

Having a look at an existing answerfile, take `ovirt-engine-setup/templates/answerfile_4.0_basic.txt.j2` for example.

As it's based out on OTOPI, and I had for reference the newly generated anwerfile from the succesfull install on machine A.

What I really wanted was a `diff` between these two answerfiles, one which was generated when DWH was installed along with the engine on the same host and the other being when the DWH would be configured on a remote host

```patch
46c46
< OVESETUP_DWH_CORE/enable=bool:True
---
> OVESETUP_DWH_CORE/enable=bool:False
49c49
< OVESETUP_DWH_DB/secured=bool:False
---
> OVESETUP_DWH_DB/secured=none:None
52,53c52,54
< OVESETUP_DWH_DB/host=str:localhost
< OVESETUP_DWH_DB/user=str:ovirt_engine_history
---
> OVESETUP_DWH_DB/host=none:None
> OVESETUP_DWH_DB/user=none:None
> OVESETUP_DWH_DB/password=none:None
55c56
< OVESETUP_DWH_DB/database=str:ovirt_engine_history
---
> OVESETUP_DWH_DB/database=none:None
57c58
< OVESETUP_DWH_DB/port=int:5432
---
> OVESETUP_DWH_DB/port=none:None
60,61c61,62
< OVESETUP_DWH_DB/securedHostValidation=bool:False
< OVESETUP_DWH_PROVISIONING/postgresProvisioningEnabled=bool:True
---
> OVESETUP_DWH_DB/securedHostValidation=none:None
> OVESETUP_DWH_PROVISIONING/postgresProvisioningEnabled=bool:False
```


This gave me a clear idea of what I needed to do with the existing answer file to make it suitable for configuring a remote DWH.

```patch
diff --git a/roles/ovirt-engine-setup/templates/answerfile_4.0_basic.txt.j2 b/roles/ovirt-engine-setup/templates/answerfile_4.0_basic.txt.j2
index 728a307..d60d03f 100644
--- a/roles/ovirt-engine-setup/templates/answerfile_4.0_basic.txt.j2
+++ b/roles/ovirt-engine-setup/templates/answerfile_4.0_basic.txt.j2
@@ -29,16 +29,29 @@ OVESETUP_DB/port=int:{ {ovirt_engine_db_port} }
 OVESETUP_DB/filter=none:None
 OVESETUP_DB/restoreJobs=int:2
 OVESETUP_DB/securedHostValidation=bool:False
+{% if ovirt_engine_dwh_db_configure %}
 OVESETUP_DWH_DB/secured=bool:False
 OVESETUP_DWH_DB/host=str:{ {ovirt_engine_dwh_db_host} }
 OVESETUP_DWH_DB/user=str:{ {ovirt_engine_dwh_db_user} }
 OVESETUP_DWH_DB/password=str:{ {ovirt_engine_dwh_db_password} }
-OVESETUP_DWH_DB/dumper=str:pg_custom
 OVESETUP_DWH_DB/database=str:{ {ovirt_engine_dwh_db_name} }
 OVESETUP_DWH_DB/port=int:{ {ovirt_engine_dwh_db_port} }
+{% else %}
+OVESETUP_DWH_DB/secured=none:None
+OVESETUP_DWH_DB/host=none:None
+OVESETUP_DWH_DB/user=none:None
+OVESETUP_DWH_DB/password=none:None
+OVESETUP_DWH_DB/database=none:None
+OVESETUP_DWH_DB/port=none:None
+{% endif %}
+OVESETUP_DWH_DB/dumper=str:pg_custom
 OVESETUP_DWH_DB/filter=none:None
 OVESETUP_DWH_DB/restoreJobs=int:2
+{% if ovirt_engine_dwh_db_configure %}
 OVESETUP_DWH_DB/securedHostValidation=bool:False
+{% else %}
+OVESETUP_DWH_DB/securedHostValidation=none:None
+{% endif %}
 OVESETUP_ENGINE_CORE/enable=bool:True
 OVESETUP_CORE/engineStop=none:None
 OVESETUP_SYSTEM/memCheckEnabled=bool:False
@@ -65,13 +78,21 @@ OVESETUP_CONFIG/imageioProxyConfig=bool:True
 OVESETUP_PROVISIONING/postgresProvisioningEnabled=bool:True
 OVESETUP_APACHE/configureRootRedirection=bool:True
 OVESETUP_APACHE/configureSsl=bool:True
+{% if ovirt_engine_dwh_db_configure %}
 OVESETUP_DWH_CORE/enable=bool:{ {ovirt_engine_dwh} }
+{% else %}
+OVESETUP_DWH_CORE/enable=bool:False
+{% endif %}
 OVESETUP_DWH_CONFIG/scale=str:1
 OVESETUP_DWH_CONFIG/dwhDbBackupDir=str:/var/lib/ovirt-engine-dwh/backups
 OVESETUP_DWH_DB/restoreBackupLate=bool:True
 OVESETUP_DWH_DB/disconnectExistingDwh=none:None
 OVESETUP_DWH_DB/performBackup=none:None
+{% if ovirt_engine_dwh_db_configure %}
 OVESETUP_DWH_PROVISIONING/postgresProvisioningEnabled=bool:True
+{% else %}
+OVESETUP_DWH_PROVISIONING/postgresProvisioningEnabled=bool:False
+{% endif %}
 OVESETUP_DB/password=str:{ {ovirt_engine_db_password} }
 OVESETUP_RHEVM_DIALOG/confirmUpgrade=bool:True
 OVESETUP_VMCONSOLE_PROXY_CONFIG/vmconsoleProxyConfig=bool:True
```

the variable `ovirt_engine_dwh_db_configure` would be a `bool` defined inside our play yaml file telling `ovirt-engine-setup` to not configure DWH on that host.

This modification sets us up for the first part.

## Configuring the remote DWH manually first

If you are trying to access the web adming on machine A, it would show you an error like this.

<center><img src="/content/images/2017/06/admin-panel-error-without-dwh.png"></center>

On machine B

```bash
[root@dwhtest-3-dwh ~]# yum -y update > /dev/null
[root@dwhtest-3-dwh ~]# yum -y install ovirt-engine-dwh-setup > /dev/null
[root@dwhtest-3-dwh ~]# engine-setup
[ INFO  ] Stage: Initializing
[ INFO  ] Stage: Environment setup
          Configuration files: ['/etc/ovirt-engine-setup.conf.d/10-packaging-jboss.conf']
          Log file: /var/log/ovirt-engine/setup/ovirt-engine-setup-20170625114213-l4cku9.log
          Version: otopi-1.6.2 (otopi-1.6.2-1.el7.centos)
[ INFO  ] Stage: Environment packages setup
[ INFO  ] Stage: Programs detection
[ INFO  ] Stage: Environment customization

          --== PRODUCT OPTIONS ==--

          Configure Data Warehouse on this host (Yes, No) [Yes]:

          --== PACKAGES ==--

[ INFO  ] Checking for product updates...
[ INFO  ] No product updates found

          --== NETWORK CONFIGURATION ==--

          Host fully qualified DNS name of this server [dwhtest-3-dwh.ovirt.org]:
[WARNING] Failed to resolve dwhtest-3-dwh.ovirt.org using DNS, it can be resolved only locally
          Setup can automatically configure the firewall on this system.
          Note: automatic configuration of the firewall may overwrite current settings.
          Do you want Setup to configure the firewall? (Yes, No) [Yes]:
          The following firewall managers were detected on this system: firewalld
          Firewall manager to configure (firewalld): firewalld
[ INFO  ] firewalld will be configured as firewall manager.
          Host fully qualified DNS name of the engine server []: dwhtest-3-engine.ovirt.org
          Setup will need to do some actions on the remote engine server. Either automatically, using ssh as root to access it, or you will be prompted to manually perform each such action.
          Please choose one of the following:
          1 - Access remote engine server using ssh as root
          2 - Perform each action manually, use files to copy content around
          (1, 2) [1]:
          ssh port on remote engine server [22]:
          root password on remote engine server dwhtest-3-engine.ovirt.org:

          --== DATABASE CONFIGURATION ==--

          Where is the DWH database located? (Local, Remote) [Local]:
          Setup can configure the local postgresql server automatically for the DWH to run. This may conflict with existing applications.
          Would you like Setup to automatically configure postgresql and create DWH database, or prefer to perform that manually? (Automatic, Manual) [Automatic]:

          Please provide the following credentials for the Engine database.
          They should be found on the Engine server in '/etc/ovirt-engine/engine.conf.d/10-setup-database.conf'.

          Engine database host []: dwhtest-3-engine.ovirt.org
          Engine database port [5432]:
          Engine database secured connection (Yes, No) [No]:
          Engine database name [engine]:
          Engine database user [engine]:
          Engine database password:

          --== OVIRT ENGINE CONFIGURATION ==--


          --== STORAGE CONFIGURATION ==--


          --== PKI CONFIGURATION ==--


          --== APACHE CONFIGURATION ==--


          --== SYSTEM CONFIGURATION ==--


          --== MISC CONFIGURATION ==--

          Please choose Data Warehouse sampling scale:
          (1) Basic
          (2) Full
          (1, 2)[2]:

          --== END OF CONFIGURATION ==--

[ INFO  ] Stage: Setup validation

          --== CONFIGURATION PREVIEW ==--

          Firewall manager                        : firewalld
          Update Firewall                         : True
          Host FQDN                               : dwhtest-3-dwh.ovirt.org
          Engine database secured connection      : False
          Engine database user name               : engine
          Engine database name                    : engine
          Engine database host                    : dwhtest-3-engine.ovirt.org
          Engine database port                    : 5432
          Engine database host name validation    : False
          DWH installation                        : True
          DWH database secured connection         : False
          DWH database host                       : localhost
          DWH database user name                  : ovirt_engine_history
          DWH database name                       : ovirt_engine_history
          DWH database port                       : 5432
          DWH database host name validation       : False
          Configure local DWH database            : True

          Please confirm installation settings (OK, Cancel) [OK]:
[ INFO  ] Stage: Transaction setup
[ INFO  ] Stopping dwh service
[ INFO  ] Stage: Misc configuration
[ INFO  ] Stage: Package installation
[ INFO  ] Stage: Misc configuration
[ INFO  ] Initializing PostgreSQL
[ INFO  ] Creating PostgreSQL 'ovirt_engine_history' database
[ INFO  ] Configuring PostgreSQL
[ INFO  ] Creating/refreshing DWH database schema
[ INFO  ] Generating post install configuration file '/etc/ovirt-engine-setup.conf.d/20-setup-ovirt-post.conf'
[ INFO  ] Stage: Transaction commit
[ INFO  ] Stage: Closing up

          --== SUMMARY ==--

[ INFO  ] Starting dwh service
          Please restart the engine by running the following on dwhtest-3-engine.ovirt.org :
          # service ovirt-engine restart
          This is required for the dashboard to work.

          --== END OF SUMMARY ==--

[ INFO  ] Stage: Clean up
          Log file is located at /var/log/ovirt-engine/setup/ovirt-engine-setup-20170625114213-l4cku9.log
[ INFO  ] Generating answer file '/var/lib/ovirt-engine/setup/answers/20170625115725-setup.conf'
[ INFO  ] Stage: Pre-termination
[ INFO  ] Stage: Termination
[ INFO  ] Execution of setup completed successfully
[root@dwhtest-3-dwh ~]#
```

Restarting the ovirt-engine service on machine A starts the whole thing.

### Debugging tips

If for any reason, the admin panel is still giving you an error. 

- Try checking the output of `iptables -S`, to see whether you are allowing incoming connections on port `5432` on both machines. 
- Are both hosts able to resolve to their public IP's using the provided FQDN
- Try entering those passwords slowly.
- If all fails, the logs of each `engine-setup` command would be there for your rescue.

## Automating it

<center><img src="/content/images/2017/06/ansible-playbook-flow.jpg"></center>

For testing my newly created `ovirt-engine-install-remote-dwh`, provisioned 2x2gig CentOS 7 vms on linode

```bash
tasrahma at TASRAHMA-M-C2MT in ~/development/gsoc/ovirt-ansible (remote-dwh-fresh-engine-install●●)
$ ssh root@dwhmanualenginetest.ovirt.org
root@dwhmanualenginetest.ovirt.org's password:
[root@dwhmanualenginetest ~]# hostname
dwhmanualenginetest.ovirt.org
[root@dwhmanualenginetest ~]# systemctl start firewalld
[root@dwhmanualenginetest ~]# firewall-cmd --zone=public --add-port=80/tcp --permanent
success
[root@dwhmanualenginetest ~]# firewall-cmd --zone=public --add-port=443/tcp --permanent
success
[root@dwhmanualenginetest ~]# firewall-cmd --zone=public --add-port=5432/tcp --permanent
success
[root@dwhmanualenginetest ~]# firewall-cmd --reload
success
[root@dwhmanualenginetest ~]# exit
logout
Connection to dwhmanualenginetest.ovirt.org closed.

tasrahma at TASRAHMA-M-C2MT in ~/development/gsoc/ovirt-ansible (remote-dwh-fresh-engine-install●●)
$ ssh root@dwhmanualdwhtest.ovirt.org
root@dwhmanualdwhtest.ovirt.org's password:
[root@li1462-178 ~]# hostname
dwhmanualdwhtest.ovirt.org
[root@li1462-178 ~]# vi /etc/hosts
[root@li1462-178 ~]# systemctl start firewalld
[root@li1462-178 ~]# firewall-cmd --zone=public --add-port=80/tcp --permanent
success
[root@li1462-178 ~]# firewall-cmd --zone=public --add-port=443/tcp --permanent
success
[root@li1462-178 ~]# firewall-cmd --zone=public --add-port=5432/tcp --permanent
success
[root@li1462-178 ~]# firewall-cmd --reload
success
[root@li1462-178 ~]# exit
logout
Connection to dwhmanualdwhtest.ovirt.org closed.
```

With these done, the servers are ready for my new ansible roles.

First would be the installation of the `ovirt-engine` host

```yaml
---
- hosts: engine
  vars:
    ovirt_engine_type: 'ovirt-engine'
    ovirt_engine_version: '4.1'
    ovirt_engine_organization: 'dwhmanualenginetest.ovirt.org'
    ovirt_engine_admin_password: 'secret'
    ovirt_rpm_repo: 'http://resources.ovirt.org/pub/yum-repo/ovirt-release41.rpm'
    ovirt_engine_organization: 'ovirt.org'
    ovirt_engine_dwh_db_host: 'remotedwh.ovirt.org'
    ovirt_engine_dwh_db_configure: false
  roles:
    - role: ovirt-common
    - role: ovirt-engine-install-packages
    - role: ovirt-engine-setup
```

Running this play file would be done like

```bash
$ ansible-playbook site.yml -i inventory --skip-tags skip_yum_install_ovirt_engine_dwh,skip_yum_install_ovirt_engine_dwh_setup
```

After which you have to configure your remote DWH installation to the previous host which has the ovirt-engine installation which again has two possibilities

On your machine B which would be `dwhmanualdwhtest.ovirt.org` in this case

```yaml
---
- hosts: dwh-remote
  vars:
    ovirt_engine_type: 'ovirt-engine'
    ovirt_engine_version: '4.1'
    ovirt_rpm_repo: 'http://resources.ovirt.org/pub/yum-repo/ovirt-release41.rpm'
    ovirt_engine_host_root_passwd: 'admin123' # the root password of the host where ovirt-engine is installed
    ovirt_engine_firewall_manager: 'firewalld'
    ovirt_engine_db_host: 'dwhmanualenginetest.ovirt.org' # FQDN of the ovirt-engine installation host, should be resolvable from the new DWH host
    ovirt_engine_host_fqdn: 'dwhmanualenginetest.ovirt.org'
    ovirt_engine_dwh_db_host: 'localhost'
    ovirt_dwh_on_dwh: True
  roles:
    - role: ovirt-common
    - role: ovirt-engine-install-packages
    - role: ovirt-engine-remote-dwh-setup
```

Running this play file would be done like

```bash
$ ansible-playbook site.yml -i inventory --skip-tags skip_yum_install_ovirt_engine,skip_yum_install_ovirt_engine_dwh
```

After this you have to restart your ovirt-engine on the host installed by doing a `service ovirt-engine restart`

And you have the whole setup now.

## Possibilty of the database of dwh on 3rd machine

I have not taken this case as of now in the new playbook which I wrote. I was able to figure out how to do it manually but the specific changes for getting this option to work using our roles is still pending.

## Next goals

- The very first thing to do now would be to modify the existing `ovirt-engine-setup-remote-dwh` that I wrote to include the case of putthing the DB of the DWH on a 3rd machine. 
- Also, Migration dwh from local machine of engine to remote machine in a new playbook which counts with already installed engine with local dwh and moved dwh to this remote server is also on the task list
- Writing tests for the new additions 

## Pitfalls

The most arcane error that I faced during this was when I was trying to automate the configuration of the remote DWH with the engine host.

<center><img src="/content/images/2017/06/arcane-otopi-var-missing-ansible-error.png"></center>

From a first glance at the error traceback, it was clear that there was some error reading the engine hostname from the OTOPI based answerfile that I had generated. 

I cross checked again and again from the answerfile which was generated from the manual installation of remote DWH as described in the process above, but to no avail.

Both of them looked just the same. I was not missing anything out from the original answerfile and they were basically the same thing minus the hostnames and passwords.

To be frank, I really didn't know what was missing out here.

On a closer look at the manual input being given while installation and the answerfile being generated from it, I noticed that the engine HOST FQDN that we provide is not being logged in the OTOPI answerfile.

Cross checking with the answerfile being generated on a new host, and sure enough. There was a prompt for the host engine FQDN which confirmed my hunch.

<center><img src="/content/images/2017/06/confirmed-hunch-prompt.png"></center>

This had to be the case! I mean there was no other explanation. I asked Lukas and he said this possibly might be a bug and suggested that I file a bug on bugzilla. Which I did here at [https://bugzilla.redhat.com/show_bug.cgi?id=1465859](https://bugzilla.redhat.com/show_bug.cgi?id=1465859) which lead me to my first bug report :)

Cross checking with the answerfile being generated on a new host, and sure enough. There was a prompt for the host engine FQDN which confirmed my hunch.

Anyway, now I just had to figure out a way of what was the OTOPI config being logged in the log files when I did enter this FQDN. Checking the logs for that particular engine-run, the logs showed me that the variable being logged.

It was `OVESETUP_ENGINE_CONFIG/fqdn`. I placed this on my template answerfile and added the remote engine host value to it. 

And sure enough, it ran successfully :)

Crazy day! But that feeling when you successfully debug something. It's always priceless no matter if it isn't the first time you are doing so.

Until next time!


## Links

- [Github branch for the work being done on issue #9 namely `remote-dwh-fresh-engine-install`](https://github.com/rhevm-qe-automation/ovirt-ansible/compare/master...tasdikrahman:remote-dwh-fresh-engine-install)
- [https://github.com/rhevm-qe-automation/ovirt-ansible/issues/9](https://github.com/rhevm-qe-automation/ovirt-ansible/issues/9)
- [https://bugzilla.redhat.com/show_bug.cgi?id=1465859](https://bugzilla.redhat.com/show_bug.cgi?id=1465859)
- [https://www.ovirt.org/documentation/architecture/architecture/](https://www.ovirt.org/documentation/architecture/architecture/)
- [http://www.ovirt.org/documentation/data-warehouse/Data_Warehouse_Guide/](http://www.ovirt.org/documentation/data-warehouse/Data_Warehouse_Guide/)
- [http://www.ovirt.org/documentation/data-warehouse/Migrating_Data_Warehouse_to_a_Separate_Machine/](http://www.ovirt.org/documentation/data-warehouse/Migrating_Data_Warehouse_to_a_Separate_Machine/)
- [http://www.ovirt.org/develop/release-management/features/engine/separate-dwh-host/](http://www.ovirt.org/develop/release-management/features/engine/separate-dwh-host/)
- [http://www.ovirt.org/develop/release-management/features/engine/migration-of-local-dwh-reports-to-remote/](http://www.ovirt.org/develop/release-management/features/engine/migration-of-local-dwh-reports-to-remote/)
- [https://www.ovirt.org/documentation/how-to/hosted-engine/](https://www.ovirt.org/documentation/how-to/hosted-engine/)
- [https://www.ovirt.org/develop/developer-guide/engine/engine-setup/](https://www.ovirt.org/develop/developer-guide/engine/engine-setup/)

