---
title: "Week 1 and 2, GSoC 2017 - Travel, Code, Good food"
description: "Week 1 and 2, GSoC 2017 - Travel, Code, Good food"
tags: [oss, python, ansible, gsoc]
comments: true
share: true
cover_image: '/content/images/2017/05/gsoc-cover.png'
---

So it's quite some time since I wrote a thing or two about the things which I have doing over the last 2 weeks. 

This month has been a roller coaster ride if you ask me. Many reasons to it. 

Some being that I travelled to [PyCon Taiwan](http://www.tasdikrahman.com/2017/06/12/PyCon-Taiwan-2017-Taipei/) which marked my first international trip and also my first PyCon talk. Got my uni results. Fingers crossed but heck. I scored a perfect 10 in the last sem! The final version of [trumporate's](https://github.com/tasdikrahman/trumporate/) UI is almost done and me and Rituraj have to just put some final touches to (blogpost for the whole development process is pending. You can find the first [one here](http://www.tasdikrahman.com/2017/05/06/Making-of-trumporate-using-markovipy-generating-sentences-using-markov-chains-part-1/))

As for the GSoC work. I have been working closely with Lukas on the `engine-rename` role which has been assigned as one of the first tasks for the first review.

## Approach

> Don't automate something which you haven't achieved manually yet!

I read this somewhere and the author was one of the employees of [chef software](https://www.chef.io/chef/) at one of the chef conferences. Now don't ask me what was I doing watching chef videos. I really don't remember what led me to that. Thanks to youtube's reco engines!

But never the less, I think the quote is quite true even in the literal sense.

I mean, if you are trying to repeat a task which consists of several small tasks with some automation tool. You ought to have gone through the manual process of doing that manually once. Especially when you are new to either the tool/process that you are trying to automate or new to the automation tool itself or both.

The reasoning behind this is very simple. As you are bound to get hiccups when trying to do the above, you will get frustrated trying to debug the errors that you are getting while automating the process. 

I mean what would you be pinpointing your error to? The process which you are following? Is that wrong? Or the syntax or way-of-doing-things which your automation tool follows?

Hard to say if you ask me.

So with this in mind I first tried renaming an ovirt-engine installation task manually. 

Nothing fancy, you find the exact steps over here from one of the ovirt [documentation pages](https://www.ovirt.org/documentation/how-to/networking/changing-engine-hostname/)

The command is very simple (assuming you are inside your ovirt-engine installation)

```bash
/usr/share/ovirt-engine/setup/bin/ovirt-engine-rename \
     --newname=ovirte1n.home.local \
     --otopi-environment="OSETUP_RENAME/forceIgnoreAIAInCA=bool:'True' \
     OVESETUP_CORE/engineStop=bool:'True' \
     OSETUP_RENAME/confirmForceOverwrite=bool:'False'"
```

The above tries, to the rename the engine-setup with the new name with minimal(read no) external interference.

More like `expect` 

so the value to the optional argument `--newname` would be the new engine-name that you wanna replace the older one with. This was given back at the time of the engine-installation but now you want it renamed.

When the `engine-setup` command is run in a clean environment, the command generates a number of certificates and keys that use the fully qualified domain name of the Manager supplied during the setup process. If the fully qualified domain name of the Manager must be changed later on (for example, due to migration of the machine hosting the Manager to a different domain), the records of the fully qualified domain name must be updated to reflect the new name. The `ovirt-engine-rename` command automates this task.


The `ovirt-engine-rename` command updates records of the fully qualified domain name of the Manager in the following locations:

```bash
/etc/ovirt-engine/engine.conf.d/10-setup-protocols.conf
/etc/ovirt-engine/imageuploader.conf.d/10-engine-setup.conf
/etc/ovirt-engine/isouploader.conf.d/10-engine-setup.conf
/etc/ovirt-engine/logcollector.conf.d/10-engine-setup.conf
/etc/pki/ovirt-engine/cert.conf
/etc/pki/ovirt-engine/cert.template
/etc/pki/ovirt-engine/certs/apache.cer
/etc/pki/ovirt-engine/keys/apache.key.nopass
/etc/pki/ovirt-engine/keys/apache.p12
```

Once the command is executed (assuming you got no error's), and with no surprises. The engine has been renamed. But there are some things yet to be done.

The next thing is to change the hostname. This is usually done by editing `/etc/hostname` and rebooting.

You can go with the `hostnamectl` command if you are on a RHEL based system. Even `hostname` command works if you. If you are using DNS to pull out the hostname, then prepare relevant DNS and/or `/etc/hosts` records for the new name.

Assuming you were accessing the web-portal before doing your engine-rename part. You will notice that you are note able to access the web interface

I was testing all this out on a 2gig Linode centos7 box (Love these guys). Right now it's all manual provisioning. I have to take a look at [terraform](https://www.terraform.io/) sometime later. 

The current workflow for me would be to create a new VM and then provision it using the [`ovirt-engine-setup`](https://github.com/rhevm-qe-automation/ovirt-ansible/tree/master/roles/ovirt-engine-setup) role which set's up the ovirt-engine for you. I have written a [blog post](http://www.tasdikrahman.com/2017/05/24/Installing-ovirt-4.1-on-centos-7-using-ansible-linode-Google-Summer-of-Code-oVirt-2017/) here if you are curious about how to do so using an ansible role.

Once that is done, your path for the role to rename the engine is quite clear.

For my personal setup, I had to now change the entry in my local dev box `/etc/hosts` to the new hostname which I gave for the ovirt-engine.

Now open the new DNS mapping on your browser. And you should be able to see the WEB UI up.

## Automating it

Now as we have the roadmap to what needs to be done, I needed to write an ansible-role for it.
 
A very simple to guess edge case here is that the new engine name is same as the one already present. How do I check that?

Checking logs would be the answer for that. I needed to check under `/etc/ovirt-engine` for that. Namely the files below

```bash
/etc/ovirt-engine/engine.conf.d/10-setup-protocols.conf
/etc/ovirt-engine/ovirt-vmconsole-proxy-helper.conf.d/10-setup.conf
/etc/ovirt-engine/isouploader.conf.d/10-engine-setup.conf
/etc/ovirt-engine/logcollector.conf.d/10-engine-setup.conf
```

The engine-name would be logged under these files. 

What I wanted to have in place was check whether the new name is not the same as the existing name out there.

So if I did a `grep` recursively on the directory of `/etc/ovirt-engine` for the with the `new-name`, it would either return something (this case means the engine-name is already at the required state) or nothing at all(this case being when you the new-name is a new one and not the one already present)

Registering this grep value to a file on `/tmp/engine-rename-logs-current.grep` and then comparing it with a `template` file using `diff` which stores the expected grep result does the job.

If the diff with this expected file returns nothing, we are already at our required state.

If not the `engine-rename` command is run.

After which again a recursive grep is done again on `/etc/ovirt-engine` with the search parameter as the new engine name.

This result is registered and compared to the expected grep result. If this passes, we now have our ovirt-engine at the desired state. i.e the name of the engine was renamed successfully.

After which we are checking the engine health at the end of the role.

The handlers which were notified when creating the temporary files are executed by the playbook at this point which delete them from the file system.

## Testing all the above

<center><img src="/content/images/2017/06/brace-yourselves-testing-is-coming.jpg"></center>

Now testing these changes out is the real thing.

We are using `docker` containers to test out the roles. I would say this is approach is quite fast and gives us the `works-everywhere` advantage. 

We need two containers, one for the `engine` and one for the `remote-db` which we are using [Chris Meyer's](http://chrismeyersfsu.me/), [provision_docker](https://github.com/chrismeyersfsu/provision_docker/) tooling.

The first role in any test would be to provision the docker containers and make sure that they are reachable.

`ovirt-ansible/tests/containers-deploy.yml` is the first role which is included as the first role to be executed in all of the `test-$.yml` files.

The above role inside, calls the role named `provision_docker` twice for provisioning the docker inventory groups specified as key:value pair to the role. Which would be nothing but the `engine` and the `remote-db` containers specified inside the `tests/inventory` file.

The other roles follow. 

The role `ovirt-engine-rename` would be coming just before we would be running the role of `ovirt-engine-cleanup`. 

Adding mine just above that will do the trick. 

> The particular PR which covers the above [https://github.com/rhevm-qe-automation/ovirt-ansible/pull/132/files](https://github.com/rhevm-qe-automation/ovirt-ansible/pull/132/files)

## Few gotcha's

The command `$ hostnamectl set-hostname test.ovirt.org` which would be used to to change the hostname was giving weird errors inside the CI and the error traceback didn't give much out apparently.

<center><img src="/content/images/2017/06/dbus-error.png"></center>

I mean what does `Success` mean here? Quite the irony if you ask me. Hah

Searching around, didn't result to much. Meanwhile Lukas thought there was some issue with the DBUS which was maybe due to some permissions change just before the particular task.

Lukas suggested that these tests were not specific to `ovirt` and that I try running the tests  on a local setup. Had another 2gig linode server for testing but all the same. The tests failed with the same error again. Which suggested that this must be an issue with the command itself being run inside the docker container.

After some fooling around this issue, I remembered that even the `hostname` command can be used to change the name of the dev box temporarily. 

Tried that and it worked!

<center><img src="/content/images/2017/06/travis-pass-log.png"></center>

Another thing which was making the tests fail were the `ansible-lints`.

I had to replace `shell` module usage with `command` module in most of the places. Turns out that both these modules are just the same.

In the most use cases both modules lead to the same goal. Here are the main differences between these modules.

- With the Command module the command will be executed without being proceeded through a shell. As a consequence some variables like $HOME are not available. And also stream operations like  `<`, `>`, `|` and `&` will not work.
- The Shell module runs a command through a shell, by default `/bin/sh`. This can be changed with the option executable. Piping and redirection are here therefor available.
- The command module is more secure, because it will not be affected by the user’s environment.

So by default, the `command` module is all that one requires to achieve most of things.

Also, there were some places in my PR where I had no choice but to use the `shell` module. For those, I had to pass the `skip_ansible_tag` for `ansible-lint` to skip those tasks.

And it’s always good to see your CI server mailing you things like this one.

<center><img src="/content/images/2017/06/travis-pass.png"></center>

So far it's been a great time for me and I have learned tons. Thanks for reading till this point. 

Cheerio!

**EDIT:**

[Yaniv Kaul](https://medium.com/@ykaul/why-not-look-at-http-docs-ansible-com-ansible-hostname-module-html-to-change-the-host-name-b3c68f1481ec) suggested me that I try renaming the container using the http://docs.ansible.com/ansible/hostname_module.html module to rename the engine, but the module failed with the same DBUS error as the previous one.

Not a big change per se for testing it out.

```patch
diff --git a/roles/ovirt-engine-rename/tasks/main.yml b/roles/ovirt-engine-rename/tasks/main.yml
index 8c96082..ed28298 100644
--- a/roles/ovirt-engine-rename/tasks/main.yml
+++ b/roles/ovirt-engine-rename/tasks/main.yml
@@ -49,7 +49,9 @@
 # to get the hostname, if not let it remain
 - name: Changing the system host name to reflect the new engine-name
   # command: hostnamectl set-hostname {{ ovirt_engine_rename_new_fqdn }}
-  command: hostname { { ovirt_engine_rename_new_fqdn } }
+  # command: hostname { { ovirt_engine_rename_new_fqdn } }
+  hostname:
+    name: '{ { ovirt_engine_rename_new_fqdn } }'
   tags:
     - skip_ansible_lint
```


Had chat with Lukas about it and he suggested that most of the systems use the DNS to pull out the hostname. So that task in the playbook ovirt-engine-rename would not be necessary and commented out on most cases.

Will be having a look at [Lago](https://lago.readthedocs.io/en/latest/) and ovirt-system-test as you suggested as Yaniv Kaul suggested for end to end testing.

## Older Blog post

- [https://medium.com/@tasdikrahman/using-ansible-playbooks-to-install-ovirt-4-1-on-centos-7-linode-9dadd90143af](https://medium.com/@tasdikrahman/using-ansible-playbooks-to-install-ovirt-4-1-on-centos-7-linode-9dadd90143af)
- [https://medium.com/@tasdikrahman/installing-ovirt-4-1-on-centos-7-digitalocean-fbb4fa037c11](https://medium.com/@tasdikrahman/installing-ovirt-4-1-on-centos-7-digitalocean-fbb4fa037c11)
- [https://medium.com/@tasdikrahman/community-bonding-period-gsoc-2017-with-ovirt-org-5d0faac95257](https://medium.com/@tasdikrahman/community-bonding-period-gsoc-2017-with-ovirt-org-5d0faac95257)
- [https://medium.com/@tasdikrahman/hello-ovirt-gsoc-2017-6c18100f21dd](https://medium.com/@tasdikrahman/hello-ovirt-gsoc-2017-6c18100f21dd)
- [https://medium.com/@tasdikrahman/testing-your-ansible-roles-using-travis-ci-d186749a8edf](https://medium.com/@tasdikrahman/testing-your-ansible-roles-using-travis-ci-d186749a8edf)