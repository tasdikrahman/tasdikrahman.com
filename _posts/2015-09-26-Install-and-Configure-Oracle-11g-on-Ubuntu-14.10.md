---
layout: post
title: "Install and Configure Oracle-11g on Ubuntu-14.10"
description: "Oracle 11g install"
tags: [sql, oracle11g]
comments: true
share: true
---

## Intro : 

Well as luck would have it, we were having our Database labs in `ORACLE 11g` as part of the coursework. And I must admit, I still have my love for `mysql`.

I installed it on my system(after a lot of trials!) this way.

>[Next part of this article](http://www.tasdikrahman.com/2015/09/27/My-Ramblings-about-Oracle-11g/)

## Pre-requisites:

JAVA should be installed and the environment variable for it should be set. Check it by doing 

{% highlight bash %}
tasdik@Acer:~$ java -version
java version "1.7.0_79"
OpenJDK Runtime Environment (IcedTea 2.5.5) (7u79-2.5.5-0ubuntu0.14.10.2)
OpenJDK 64-Bit Server VM (build 24.79-b02, mixed mode)
tasdik@Acer:~$ echo $JAVA_HOME
/usr/lib/jvm/java-7-openjdk-amd64
tasdik@Acer:~$ 
{% endhighlight %}

If not you can set it by placing the installed `jvm` to `/etc/environment`

Mine looks like this

{% highlight bash linenos %}
tasdik@Acer:~$ cat /etc/environment 
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games"
http_proxy="http://172.16.0.16:8080/"
https_proxy="https://172.16.0.16:8080/"
ftp_proxy="ftp://172.16.0.16:8080/"
socks_proxy="socks://172.16.0.16:8080/"
JAVA_HOME="/usr/lib/jvm/java-7-openjdk-amd64"
tasdik@Acer:~$ 
{% endhighlight %}


## Installing Oracle 11g R2 Express Edition

A couple of extra things are needed 


{% highlight bash %}
tasdik@Acer:~$ sudo apt-get install alien libaio1 unixodbc
{% endhighlight %}

Download Oracle 11g, you need to make an account for that(Yeah, I know!)

Link : [http://www.oracle.com/technetwork/database/database-technologies/express-edition/downloads/index.html](http://www.oracle.com/technetwork/database/database-technologies/express-edition/downloads/index.html)

Place annd Unzip the `zip ` file to the Directory of your choice. I did mine to 'Documents'

{% highlight bash %}
tasdik@Acer:~$ cd Documents/
tasdik@Acer:~/Documents$ unzip oracle-xe-11.2.0-1.0.x86_64.rpm.zip

## A new directory was added

tasdik@Acer:~$ cd Documents/Disk1/
{% endhighlight %}

Now we need to convert the rpm (A red hat package to .deb) package

{% highlight bash %}
tasdik@Acer:~/Documents/Disk1$ sudo alien --scripts -d oracle-xe-11.2.0-1.0.x86_64.rpm
{% endhighlight %}

This will take a while, so open another `terminal` window

{% highlight bash %}
tasdik@Acer:~/Documents/Disk1$ sudo nano /sbin/chkconfig
{% endhighlight %}

And then add the following to it

{% highlight bash linenos %}
#!/bin/bash
# Oracle 11gR2 XE installer chkconfig hack for Ubuntu
file=/etc/init.d/oracle-xe
if [[ ! `tail -n1 $file | grep INIT` ]]; then
echo >> $file
echo '### BEGIN INIT INFO' >> $file
echo '# Provides: OracleXE' >> $file
echo '# Required-Start: $remote_fs $syslog' >> $file
echo '# Required-Stop: $remote_fs $syslog' >> $file
echo '# Default-Start: 2 3 4 5' >> $file
echo '# Default-Stop: 0 1 6' >> $file
echo '# Short-Description: Oracle 11g Express Edition' >> $file
echo '### END INIT INFO' >> $file
fi
update-rc.d oracle-xe defaults 80 01
#EOF
{% endhighlight %}

Save it and give the required execution privileges 

{% highlight bash %}
tasdik@Acer:~$ sudo chmod 755 /sbin/chkconfig
{% endhighlight %}

After this, we have to create the file `/etc/sysctl.d/60-oracle.conf` to set the additional kernel parameters. Open the file by executing the following statement.

{% highlight bash %}
tasdik@Acer:~/Documents/Disk1$ sudo nano /etc/sysctl.d/60-oracle.conf
{% endhighlight %}

Copy and paste the following into the file. Kernel.shmmax is the maximum possible value of physical RAM in bytes. `536870912 / 1024 /1024 = 512 MB.`


{% highlight bash linenos %}
# Oracle 11g XE kernel parameters
fs.file-max=6815744
net.ipv4.ip_local_port_range=9000 65000
kernel.sem=250 32000 100 128
kernel.shmmax=536870912
{% endhighlight %}

Load the kernel parameters:

{% highlight bash %}
tasdik@Acer:~/Documents/Disk1$ sudo service procps start
{% endhighlight %}

The changes may be verified again by executing:

{% highlight bash %}
tasdik@Acer:~/Documents/Disk1$ sudo sysctl -q fs.file-max
fs.file-max = 6815744
tasdik@Acer:~/Documents/Disk1$ 
{% endhighlight %}

After this, execute the following statements to make some more required changes:

{% highlight bash %}
tasdik@Acer:~/Documents/Disk1$ sudo ln -s /usr/bin/awk /bin/awk
tasdik@Acer:~/Documents/Disk1$ mkdir /var/lock/subsys
tasdik@Acer:~/Documents/Disk1$ touch /var/lock/subsys/listener
{% endhighlight %}


## Install the .deb file


{% highlight bash %}
tasdik@Acer:~/Documents/Disk1$ sudo dpkg --install oracle-xe_11.2.0-2_amd64.deb
{% endhighlight %}

After that, 
Execute the following to avoid getting a ORA-00845: MEMORY_TARGET error. Note: replace “size=3804m” with the size of your (virtual) machine’s RAM in MBs.

>Note : Close Chrome before executing the following three lines as it crashes when doing so

To check your system RAM (allated), do a 

{% highlight bash %}
tasdik@Acer:~$ free -tom
             total       used       free     shared    buffers     cached
Mem:          3804       3465        338        910         81       1642
Swap:         3943         30       3913
Total:        7748       3496       4252
tasdik@Acer:~$ 
{% endhighlight %}


My RAM is `3804` here 
Then do the following

{% highlight bash %}
tasdik@Acer:~$ sudo rm -rf /dev/shm
tasdik@Acer:~$ sudo mkdir /dev/shm
tasdik@Acer:~$ sudo mount -t tmpfs shmfs -o size=3804m /dev/shm
{% endhighlight %}

Create the file /etc/rc2.d/S01shm_load.

{% highlight bash %}
tasdik@Acer:~$ sudo nano /etc/rc2.d/S01shm_load
{% endhighlight %}

And then add the following

{% highlight bash linenos %}
#!/bin/sh
case "$1" in
start) mkdir /var/lock/subsys 2>/dev/null
touch /var/lock/subsys/listener
rm /dev/shm 2>/dev/null
mkdir /dev/shm 2>/dev/null
mount -t tmpfs shmfs -o size=3804m /dev/shm ;;
*) echo error
exit 1 ;;
esac
{% endhighlight %}

Give permissions to it

{% highlight bash %}
tasdik@Acer:~$ sudo chmod 755 /etc/rc2.d/S01shm_load
{% endhighlight %}

## Configuring Oracle 11g R2 Express Edition

{% highlight bash %}
tasdik@Acer:~$ sudo /etc/init.d/oracle-xe configure
{% endhighlight %}

Go on choosing the defaults. I chose,  not to start `Oracle on startup`. So do accordingly.

Setting the environment vars


{% highlight bash %}
tasdik@Acer:~$ sudo gedit /etc/bash.bashrc
{% endhighlight %}	

And add the following 

{% highlight bash linenos %}
#### for oracle 11g
export ORACLE_HOME=/u01/app/oracle/product/11.2.0/xe
export ORACLE_SID=XE
export NLS_LANG=`$ORACLE_HOME/bin/nls_lang.sh`
export ORACLE_BASE=/u01/app/oracle
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH
export PATH=$ORACLE_HOME/bin:$PATH
{% endhighlight %}

Save it and `source` it

{% highlight bash %}
tasdik@Acer:~$ source /etc/bash.bashrc
## check the environment variables
tasdik@Acer:~$ echo $ORACLE_HOME
/u01/app/oracle/product/11.2.0/xe
tasdik@Acer:~$ 
{% endhighlight %}

I recommend rebooting the system at this point of time

## After a System Reboot

{% highlight bash %}
tasdik@Acer:~$ sudo service oracle-xe start
{% endhighlight %}

A file named oraclexe-gettingstarted.desktop is placed on your desktop. To make this file executable, navigate to you desktop.

{% highlight bash %}
tasdik@Acer:~$ cd ~/Desktop
tasdik@Acer:~/Desktop$ sudo chmod a+x oraclexe-gettingstarted.desktop
{% endhighlight %}

## Running `SQlplus`


You have to Unset `http_proxy` and `no_proxy`.

To do that

{% highlight bash %}
tasdik@Acer:~$ unset http_proxy
tasdik@Acer:~$ unset no_proxy
tasdik@Acer:~$ sqlplus

SQL*Plus: Release 11.2.0.2.0 Production on Sat Sep 26 11:34:06 2015

Copyright (c) 1982, 2011, Oracle.  All rights reserved.

Enter user-name: SYSTEM
Enter password: 

Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL> 

{% endhighlight %}

There you go!

## Ending notes:

Remember to do a

{% highlight bash %}
tasdik@Acer:~$ sudo service oracle-xe start
{% endhighlight %}

at startup, if you chose not to Start `Oracle 11 g` at system startup
