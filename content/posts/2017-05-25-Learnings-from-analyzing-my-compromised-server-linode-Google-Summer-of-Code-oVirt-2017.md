---
title: "Learnings from analyzing my compromised server (Linode)"
description: "Learnings from analyzing my compromised server (Linode)"
tags: [infosec, linux]
comments: true
share: true
cover_image: '/content/images/2017/05/web-server-security.jpg'
---

> DISCLAIMER: All views presented are personal and not that of my employers or anyone else for that matter. In no occasion do I blame Linode for this security breach. It was because I did not follow the best practices which you will read and not repeat again.

Yesterday, I was having a great day!

I had the daily goal of walking 6000 steps done. Thanks to my swanky new Mi band 1 (bade goodbye to my Sony SWR10). Wrote about installing `ovirt-engine` using an ansible-playbook in a post [yesterday](http://www.tasdikrahman.com/2017/05/24/Installing-ovirt-4.1-on-centos-7-using-ansible-linode-Google-Summer-of-Code-oVirt-2017/). My mom wasn't telling me for a change to get a haircut and suggested I start packing some proper clothes (she means washed and ironed) for my upcoming [trip to Taiwan](https://tw.pycon.org/2017/en-us/events/talk/342865744498786414/) which would be my first international trip. 

You need some getting used to, to the event of waking up to your significant others video call early in the morning. And I tell you if you had some bad sleep like me yesterday. You would hardly be in the mood for it. Sorry sunshine.

Anyways. So about yesterday, I finally decided to shift from DigitalOcean to Linode. As evident from my tweet yesterday.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/tasdikrahman">@tasdikrahman</a> <a href="https://twitter.com/linode">@linode</a> Now, I&#39;m not trying to entice you further, but you could use the code &#39;linode10&#39; and get some credit to start with. It&#39;s nice to have.</p>&mdash; Feeling OK (@FeelingSohSoh) <a href="https://twitter.com/FeelingSohSoh/status/867382204430766080">May 24, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Thanks for the credits Linode. Appreciate it :)

Fast forward some hours. I have a 4GB centOS 7 box up and running on a Singapore datecenter. After some failed attempts, got my `ansible-playbook` to run on the remote machine which installed ovirt-engine on it.

## Happy period quickly turning to a bad one

Everything is fine and dandy and I am watching a basketball match with my friends after office hours. 4 minutes left to the final whistle, Cracking match on, everyone is playing like a pro. A very close call between the two teams, but one gets the better of the other one. 

Returning back home. I get this buzz on my phone. Turns out it's an email from Linode. Daym. I thought was I billed already?

<center><img src="/content/images/2017/05/outbound-network-linode-mail.png"></center>

Trust me on this, I was really not sure what to do of this for the first two minutes when I read the email.

I opened the Linode admin panel to check out what was my server up to. And the CPU graph had jumped off the hooks.

<center><img src="/content/images/2017/05/CPU-chart-linode-1.png"></center>

Same was the case with the network graph

<center><img src="/content/images/2017/05/network-chat-linode-1.png"></center>

Looking at the network log's suggested a high amount of outbound traffic coming from my server, further cementing the Linode support ticket that I got.

<center><img src="/content/images/2017/05/network-io-linode-1.png"></center>

I ssh'd inside my server to see what was going on. 

<center><img src="/content/images/2017/05/failed-ssh-tries.png"></center>

I will be damned. I don't remember sleep typing my password continuously for that long! 

Let me tell you, you don't do a `cat /var/log/secure` at this point as the file would just be spit continously at you with no end of stopping. 

Did `head` (even a `tail` can do) to it. Going through the start of the file, everything was fine until I started to see the extremely less epoch time between two failed attempts. This confirmed my hunch that some script kiddie was trying to brute force through the `root` user login. 

<center><img src="/content/images/2017/05/var-log-secure-failed-attempts.png"></center>

I know, I should have disabled root login at the start and used ssh-keys to access my server. But I just delayed it to be done the next day. My fault. 

> The logical thing now would be to start `iptables` (or) `ufw` and block outbound traffic as well as inbound except required stuff. Then take all the logs and look at them.

## yum install breaks

<center><img src="/content/images/2017/05/ufw-notfound.png"></center>

Now I am pretty sure most linux distrbutions, Ubuntu for example ships with `ufw` by default. An `SELinux` like `centOS` not shipping `ufw` cannot be remotely possible. My hunch was that the perpetrator must have removed it altogether. 

No problemo, I could do a `yum install ufw` right? 

<center><img src="/content/images/2017/05/yum-timeout.png"></center>

I try installing it and there are constant timeouts on the network calls being made by the server to the mirrors holding the package. Same is the case with other packages like `strace`, `tcpdump` et al.

<center><img src="/content/images/2017/05/yum-timeout-1.png"></center>

The connection is really sluggish even though I am on a network having very less latency. 

## when `ufw` fails go back to good old `iptables`

`ufw` stands for **uncomplicated firewall**. So that you don't have to directly deal with the low level intricacies

With `ufw`, for blocking all incoming traffic from a given IP address would have been as simple as doing a 

```bash
$ sudo ufw deny from <ip-address>
```

But since I don't have it, falling back to `iptables`. 

<center><img src="/content/images/2017/05/iptables-drop-ip.png"></center>

## Checking all outgoing requests/connections from the server

For that I did a `$ netstat -nputw`

lists all `UDP (u)`, `TCP (t)` and `RAW (w)` outgoing connections (not using l or a) in a numeric form (n, prevents possible long-running DNS queries) and includes the program (p) associated with that.

<center><img src="/content/images/2017/05/netstat.png"></center>

What has a `cd` command got to do with making network connections?

Did a `$ ps aux` to check all the system processes and the thing was standing out here too

<center><img src="/content/images/2017/05/ps-aux-cd.png"></center>

It was taking up 95% of the whole CPU! When was the last time you saw a `coreutil` doing that?

Digging down further, I wanted to see what were the files being opened by our, at this point I may say modified `cd` program

<center><img src="/content/images/2017/05/lsof-cd.png"></center>

The perpetrator had been running his malicious program under `/usr/bin` which obviously meant he did gain root access to my server. There was no other way I could think of, through which the perpetrator could have placed it under `/usr/bin` otherwise.

If you feel, someone else too is logged in, you can easily check that by doing a 

```bash
$ netstat -nalp | grep “:22″
```

To double check that he did get in, I ran a 

```bash
$ cat /var/log/secure* | grep ssh | grep Accept`
```

And not so surprisingly,

<center><img src="/content/images/2017/05/he-got-in.png"></center>

At this point I don't have anything like `strace` or `gdb` to it to see what this program is doing, as mentioned earlier the inability do install anything using `yum` through the network. 

Compiling from source?

`wget` was no where to be found!

I could only think of killing the process once and see what happens then. I do a `$ kill -9 3618` to kill the process and check `ps aux` again looking for the particular program only to find it back again as a new process

## Conjuring 3

<center><img src="/content/images/2017/05/cd-resurfaces.png"></center>

Again killing it and checking the processes. This time I don't see it. 

But I was wrong, looks like there is another process hogging CPU in a similar fashion like `cd` was doing.

<center><img src="/content/images/2017/05/resurface-as-ifconfig.png"></center>

Tbh, this was bat shoot crazy stuff for me. And I was feeling the adrenaline pump. Sleep was something I lost completely even though it was some 3a.m. in the morning. This crap was way too exciting to be left over to be done in the morning!

I check `ps aux` twice just to double check if there anything in the lines of what happened. And there is nothing this time to send a `SIG KILL` to.

## Relief?

I try installing packages through `yum` and it's the same old story of network-timeout.

You might also be wondering why I did not check `bash history`. Trust me, it was clean! I could only see the things which I had typed away. Whatever it was, it was smart. But I guess this is just the basic stuff of not leaving anything behind.

## nmap away all the things

Cause why not? 

Ran a quick scan on the server using 

```bash
$ sudo nmap ovirtlinode.gsoc.org

Starting Nmap 7.40 ( https://nmap.org ) at 2017-05-25 14:35 IST
Nmap scan report for ovirtlinode.gsoc.org
Host is up (0.035s latency).
Not shown: 978 filtered ports
PORT     STATE  SERVICE
22/tcp   open   ssh
25/tcp   closed smtp
80/tcp   open   http
113/tcp  closed ident
179/tcp  closed bgp
443/tcp  open   https
1723/tcp closed pptp
2000/tcp closed cisco-sccp
2222/tcp open   EtherNetIP-1
5432/tcp open   postgresql
6000/tcp closed X11
6001/tcp closed X11:1
6002/tcp closed X11:2
6003/tcp closed X11:3
6004/tcp closed X11:4
6005/tcp closed X11:5
6006/tcp closed X11:6
6007/tcp closed X11:7
6009/tcp closed X11:9
6025/tcp closed x11
6059/tcp closed X11:59
6100/tcp open   synchronet-db
```

Checking the UDP connections

```bash
$ sudo nmap -sS -sU -T4 -A -v ovirtlinode.gsoc.org
Starting Nmap 7.40 ( https://nmap.org ) at 2017-05-25 14:37 IST
NSE: Loaded 143 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 14:37
Completed NSE at 14:37, 0.00s elapsed
Initiating NSE at 14:37
Completed NSE at 14:37, 0.00s elapsed
Initiating Ping Scan at 14:37
Scanning ovirtlinode.gsoc.org (172.104.36.181) [4 ports]
Completed Ping Scan at 14:37, 0.01s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 14:37
Scanning ovirtlinode.gsoc.org (172.104.36.181) [1000 ports]
Discovered open port 80/tcp on 172.104.36.181
Discovered open port 443/tcp on 172.104.36.181
Discovered open port 22/tcp on 172.104.36.181
Discovered open port 6100/tcp on 172.104.36.181
Discovered open port 5432/tcp on 172.104.36.181
Discovered open port 2222/tcp on 172.104.36.181
Completed SYN Stealth Scan at 14:37, 4.22s elapsed (1000 total ports)
Initiating UDP Scan at 14:37
Scanning ovirtlinode.gsoc.org (172.104.36.181) [1000 ports]
Completed UDP Scan at 14:37, 4.19s elapsed (1000 total ports)
Initiating Service scan at 14:37
```

Did not get much out of it.

So this brings me to the moment where I see the attack vector which was placed by the perpetrator becomes dormant. I did not see any other activity which would wry my attention again. 

I let my server run for the night (stupid call but I was just plain old curious) to check in the morning what was the status.

Turned out there still had been a fair amount of outbound calls being made during that time. 

<center><img src="/content/images/2017/05/still-outboung-io.png"></center>

The CPU graph resonated the same

<center><img src="/content/images/2017/05/cpu-graph-morn.png"></center>

I turned off the server for a brief period of time after this. The CPU graphs can be seen below during that period.

<center><img src="/content/images/2017/05/final-cpu-graph.png"></center>

There were no outbound network calls after that.

<center><img src="/content/images/2017/05/final-network-io.png"></center>

## Aftermath

I turned off the server for good. There's no going back to it. I will be rebuilding the image to it.

It happened to have my public key on the server. This is not something one should immediately worry about. Going from public key to private key is exceptionally hard; that's how public key cryptography works. (By “exceptionally” I mean that it's designed to be resistant to well-funded government efforts; if it keeps the NSA from cracking you, it'll be sure good enough for stopping your average joe)

But just to be on the safer side, I regenerated my ssh keys, deleted the old ones at the places it was being used and updated it with the new ones.

Just to be extra sure of everything, I checked the activity logs of the services which did use the older ssh keys. No unusual activities.

<center><img src="/content/images/2017/05/github-logs.png"></center>


## Learnings

- disable root password login

The very first thing that I should have done after provisioning the server would have been to do this. This would infact have stopped the perpetrator from logging in as root and cause any havoc

- obfuscate the port `sshd` uses

change it to something not common as the default `22` would be known by you as well as the perpetrator. ~As someone rightfully said, security lies in obscurity.~

But thinking that security through obscurity is makes you fail safe. Think again. 

Security by obscurity is a beginner fail! one thing is to keep secrets and make it harder to exploit, the other is to rely on secrets for security. The financials were doing it for long until they learned it does not work.

It’s better to use known good practices, protocols and techniques than to come with your own, reinventing the wheel and trying to keep it secret. Reverse engineering has been done for everything, from space rockets to smart toasters. It takes usually 30min to fingerprint an OS version aling with libraries no matter how you protect it. Just looking at a ping RTT can identify the OS, for example.

Finally, nobody ever got blamed for using best practices. Yet, if you try to outsmart the system it’ll all be your fault.

- Use your public key to ssh into the machine instead of password login as suggested. 

- Take regular backups/snapshot’s of your server. That way if something funny does happen. You can always restore it to a previous state.

- I kept a really easy to guess, dictionary based password. I have to admit it, this is simply the stupidest thing one can do. And yes, I did it. My only reasoning for doing that was that this was a throwaway server, but that makes up for no excuse for not following security practices.

KEEP A STRONG PASSWORD! Even though there is a brute force attack on your server, it is relatively very hard to crack the password if you keep a strong one.

Bruce [has a nice essay](http://www.schneier.com/essay-148.html) about the subject here. Won’t repeat what he has said so take a look at it.

Research suggests that adding password complexity requirements like upper case/numbers/symbols cause users to make easy to predict changes and cause them to create simpler passwords overall due to being harder to remember.

Also read [OWASP’s blog](https://www.owasp.org/index.php/Authentication_Cheat_Sheet#Implement_Proper_Password_Strength_Controls) here about what they have to say about passwords 

There was a discussion on [security exchange](https://security.stackexchange.com/questions/29836/what-are-good-requirements-for-a-password) too over this 

- protect ssh with fail2ban

 Fail2ban can mitigate this problem by creating rules that automatically alter your iptables firewall configuration based on a predefined number of unsuccessful login attempts. This will allow your server to respond to illegitimate access attempts without intervention from you. 

- Some Files which are in the Common Attack Points:

```bash 
$ ls /tmp -la

$ ls /var/tmp -la

$ ls /dev/shm -la
```
Check these to have a look for something which should not be there. 

Unluckily, I had already rebooted my server at that point which caused me to lose this info.

- Keep a pristine copy of critical system files (such as ls, ps, netstat, md5sum) somewhere, with an md5sum of them, and compare them to the live versions regularly. Rootkits will invariably modify these files. Use these copies if you suspect the originals have been compromised.

- `aide` or `tripwire` will tell you of any files that have been modified - assuming their databases have not been tampered with.
Configure syslog to send your logfiles to a remote log server where they can't be tampered with by an intruder. Watch these remote logfiles for suspicious activity

- read your logs regularly - use `logwatch` or `logcheck` to synthesize the critical information.
- Know your servers. Know what kinds of activities and logs are normal.
- using tools like [chkrootkit](http://www.chkrootkit.org/download/) to check for rootkits regularly

## Closing notes

> How do you know if your Linux server has been hacked?

You don’t!

I know, I know — but it’s the paranoid, sad truth, really ;) There are plenty of hints of course, but if the system was targeted specifically — it might be impossible to tell. It’s good to understand that nothing is ever completely secure.

If your system was compromised, meaning once someone has root access on your host, you cannot trust anything you see because above and beyond the more obvious methods like modifying ps, ls, etc, one can simply attack kernel level system calls to subjugate IO itself. None of your system tools can be trusted to reveal the truth.

Simply put, you cant trust anything your terminal tells you, period.

**I would like to thank everyone at Linode. Their effort and dedication to maintaining our servers and their security, while ensuring an affordable price, enables us to succeed. Thanks guys**

**If I have made any mistakes or you think I should have done something else from what I have tried or not described something as it should be. Please feel free to point it out. I am just starting out on the infosec scene :)**

Cheers and stay secured!

And I just have to post this image.

<center><img src="/content/images/2017/05/today-was-a-good-day-meme.jpg"></center>

> Catch the discussion over [HN](https://news.ycombinator.com/item?id=14416982)


## Further read

- [https://www.webair.com/community/can-check-server-hacked/](https://www.webair.com/community/can-check-server-hacked/)
- [https://serverfault.com/questions/218005/how-do-i-deal-with-a-compromised-server](https://serverfault.com/questions/218005/how-do-i-deal-with-a-compromised-server)
- [https://security.stackexchange.com/questions/7443/how-do-you-know-your-server-has-been-compromised](https://security.stackexchange.com/questions/7443/how-do-you-know-your-server-has-been-compromised)
- [https://wiki.centos.org/HowTos/Network/IPTables](https://wiki.centos.org/HowTos/Network/IPTables)
- [https://stackoverflow.com/questions/884525/how-to-investigate-what-a-process-is-doing](https://stackoverflow.com/questions/884525/how-to-investigate-what-a-process-is-doing)
- [https://jorge.fbarr.net/2014/01/19/introduction-to-strace/](https://jorge.fbarr.net/2014/01/19/introduction-to-strace/)
- [https://stackoverflow.com/questions/3972534/dsa-what-can-a-hacker-do-with-just-a-public-key](https://stackoverflow.com/questions/3972534/dsa-what-can-a-hacker-do-with-just-a-public-key)
- [https://security.stackexchange.com/questions/44094/isnt-all-security-through-obscurity](https://security.stackexchange.com/questions/44094/isnt-all-security-through-obscurity)
- [https://stackoverflow.com/questions/533965/why-is-security-through-obscurity-a-bad-idea](https://stackoverflow.com/questions/533965/why-is-security-through-obscurity-a-bad-idea)