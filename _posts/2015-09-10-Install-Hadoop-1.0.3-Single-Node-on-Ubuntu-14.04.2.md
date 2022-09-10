---
layout: post
title: "Install Hadoop(Single node)"
description: "Install Hadoop Single node 1.0.3 on Ubuntu 14.04.2"
tags: [hadoop, bigdata, ubuntu]
comments: true
share: true
---
 

## On a starting note : 

  I am assuming that you have a fresh Ubuntu install on your system as this will cut down a lot of frustration trying to debug why Hadoop is not running. 

  I am using `nano` as the text editor for this article but you can use any other text-editor like `vi`, `sublime` or `atom` for doing the same

>Next article : [Install Hadoop Multi Node 1.0.3 on Ubuntu 14.04.2](http://www.tasdikrahman.com/2015/09/13/Setup-Hadoop-on-Ubuntu-Multi-Node-Cluster/)

## Installation

### Create a dedicated user 

Create a seperate user named `hadoop` for seperating out the configuration files and the installation files 

{% highlight bash %}
sys8@sys8:~$ sudo adduser hadoop
{% endhighlight %}


You would be then prompted to add the new UNIX password and details 

add this user to the sudoer's group

{% highlight bash %}
sys8@sys8:~$ sudo adduser hadoop sudo
{% endhighlight %}

Switch to the newly created user


{% highlight bash %}
sys8@sys8:~$ su - hadoop
{% endhighlight %}

Install JAVA which is in the default repos of Ubuntu

{% highlight bash %}
hadoop@sys8:~$ sudo apt-get install default-jdk
{% endhighlight %}

### Configure SSH:

Enabling ssh for localhost is the next step

Install `openssh-server`

{% highlight bash %}
hadoop@sys8:~$ sudo apt-get install openssh-server
{% endhighlight %}

Generate the keys
{% highlight bash %}
hadoop@sys8:~$ ssh-keygen -t rsa -P ""
Generating public/private rsa key pair.
Enter file in which to save the key (/home/hadoop/.ssh/id_rsa): 
Created directory '/home/hadoop/.ssh'.
Your identification has been saved in /home/hadoop/.ssh/id_rsa.
Your public key has been saved in /home/hadoop/.ssh/id_rsa.pub.
The key fingerprint is:
68:b0:84:c0:3f:16:41:38:d9:7e:d6:63:a3:a0:28:f5 hadoop@sys8
The key's randomart image is:
+--[ RSA 2048]----+
|o =o.            |
| * +             |
|  = + .          |
|  .B = *         |
|..o.* = S        |
|o.  Eo           |
|.                |
|                 |
|                 |
+-----------------+
hadoop@sys8:~$ cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys
{% endhighlight %}

Copy the keys over to enable passwordless ssh

{% highlight bash %}
hadoop@sys8:~$ ssh-copy-id -i $HOME/.ssh/id_rsa.pub hadoop@localhost
The authenticity of host 'localhost (127.0.0.1)' can't be established.
ECDSA key fingerprint is 17:f4:fc:aa:88:4d:51:b1:08:ae:df:75:2f:07:37:26.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed

/usr/bin/ssh-copy-id: WARNING: All keys were skipped because they already exist on the remote system.
{% endhighlight %}

Test the ssh connection to the localhost 

{% highlight bash %}
hadoop@sys8:~$ ssh hadoop@localhost
Welcome to Ubuntu 14.04.3 LTS (GNU/Linux 3.19.0-28-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

60 packages can be updated.
24 updates are security updates.

hadoop@sys8:~$ exit
logout
Connection to localhost closed.

hadoop@sys8:~$
{% endhighlight %}

### Apache Hadoop Installation:

Download hadoop from apache's site, 
 
But first change to hadoop's home directory first 

{% highlight bash %}
hadoop@sys8:~$ pwd
/home/hadoop
hadoop@sys8:~$ wget https://archive.apache.org/dist/hadoop/core/hadoop-1.2.1/hadoop-1.2.1.tar.gz
--2015-09-12 18:51:49--  https://archive.apache.org/dist/hadoop/core/hadoop-1.2.1/hadoop-1.2.1.tar.gz
Resolving archive.apache.org (archive.apache.org)... 140.211.11.131, 192.87.106.229, 2001:610:1:80bc:192:87:106:229
Connecting to archive.apache.org (archive.apache.org)|140.211.11.131|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 63851630 (61M) [application/x-gzip]
Saving to: ‘hadoop-1.2.1.tar.gz’

100%[======================================================================================================>] 6,38,51,630 1.82MB/s   in 42s    

2015-09-12 18:52:32 (1.45 MB/s) - ‘hadoop-1.2.1.tar.gz’ saved [63851630/63851630]
{% endhighlight %}

Extract and rename the file

{% highlight bash %}
hadoop@sys8:~$ sudo tar xzf hadoop-1.2.1.tar.gz
hadoop@sys8:~$ sudo mv hadoop-1.2.1 hadoop
{% endhighlight %}

Set the user permissions

{% highlight bash %}
hadoop@sys8:~$ sudo chown -R hadoop:hadoop hadoop
{% endhighlight %}


***

## Configuration

### Update `/etc/profile`

{% highlight bash %}
hadoop@sys8:~$ sudo nano /etc/profile
{% endhighlight %}

Add the following lines at the end

**NOTE** : 

This is assuming that you have installed the default-jdk from the repo's. Change it accordingly for any other java version

{% highlight bash linenos %}
export HADOOP_HOME="/home/hadoop/hadoop/"
export JAVA_HOME="/usr/lib/jvm/java-1.7.0-openjdk-amd64"

unalias fs &> /dev/null
alias fs="hadoop fs"
unalias hls &> /dev/null
alias hls="fs -ls"

export PATH="$PATH:$HADOOP_HOME/bin"
{% endhighlight %}

***

## Edit the configuration files for Hadoop

### Edit `hadoop-env.sh`

Change to the hadoop folder and add the JAVA home path

{% highlight bash %}
hadoop@sys8:~$ cd ~/hadoop
hadoop@sys8:~/hadoop$ ls
bin        CHANGES.txt  docs                     hadoop-core-1.2.1.jar         hadoop-test-1.2.1.jar   ivy.xml  LICENSE.txt  sbin   webapps
build.xml  conf         hadoop-ant-1.2.1.jar     hadoop-examples-1.2.1.jar     hadoop-tools-1.2.1.jar  lib      NOTICE.txt   share
c++        contrib      hadoop-client-1.2.1.jar  hadoop-minicluster-1.2.1.jar  ivy                     libexec  README.txt   src
hadoop@sys8:~/hadoop$ sudo nano conf/hadoop-env.sh
{% endhighlight %}

Add the following 

{% highlight bash %}
# The java implementation to use.  Required.
export JAVA_HOME="/usr/lib/jvm/java-1.7.0-openjdk-amd64"
export HADOOP_OPTS=-Djava.net.preferIPv4Stack=true
{% endhighlight %}


**Note** : 

Added the second line so as to disable ipv6 specifically for hadoop and not for the whole system

###Create the `tmp` folder for Hadoop

{% highlight bash %}
hadoop@sys8:~/hadoop$ sudo mkdir -p /app/hadoop/tmp
### add the permissions
hadoop@sys8:~/hadoop$ sudo chown hadoop:hadoop /app/hadoop/tmp
hadoop@sys8:~/hadoop$ sudo chmod 750 /app/hadoop/tmp
{% endhighlight %}


### Edit `conf/core-site.xml`

{% highlight bash %}
hadoop@sys8:~/hadoop$ sudo nano conf/core-site.xml
{% endhighlight %}


You would be having blank spaces in between the <configuration> tags

It should look something like this after editing

{% highlight xml linenos %}

<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<!-- Put site-specific property overrides in this file. -->

<configuration>
 <property>
  <name>hadoop.tmp.dir</name>
  <value>/app/hadoop/tmp</value>
  <description>A base for other temporary directories.</description>
 </property>
 <property>
  <name>fs.default.name</name>
  <value>hdfs://localhost:54310</value>
  <description>The name of the default file system.  A URI whose
  scheme and authority determine the FileSystem implementation.  The
  uri's scheme determines the config property (fs.SCHEME.impl) naming
  the FileSystem implementation class.  The uri's authority is used to
  determine the host, port, etc. for a filesystem.</description>
 </property>
</configuration>
{% endhighlight %}


### Edit `conf/mapred-site.xml`

{% highlight bash %}
hadoop@sys8:~/hadoop$ sudo nano conf/mapred-site.xml
{% endhighlight %}


It should look something like this after editing

{% highlight xml linenos %}
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<!-- Put site-specific property overrides in this file. -->

<configuration>
 <property>
  <name>mapred.job.tracker</name>
  <value>localhost:54311</value>
  <description>The host and port that the MapReduce job tracker runs
  at.  If "local", then jobs are run in-process as a single map
  and reduce task.
  </description>
</property>
</configuration>
{% endhighlight %}


### Edit `conf/hdfs-site.xml`

{% highlight bash %}
hadoop@sys8:~/hadoop$ sudo nano conf/hdfs-site.xml
{% endhighlight %}


It should look something like this after editing

{% highlight xml linenos %}
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<!-- Put site-specific property overrides in this file. -->

<configuration>
 <property>
  <name>dfs.replication</name>
  <value>1</value>
  <description>Default block replication.
  The actual number of replications can be specified when the file is created.
  The default is used if replication is not specified in create time.
  </description>
 </property>
</configuration>
{% endhighlight %}


### Format the HDFS file 

**Note** : Run this one command only once, i.e now

{% highlight bash %}
hadoop@sys8:~/hadoop$ bin/hadoop namenode -format
15/09/12 19:01:48 INFO namenode.NameNode: STARTUP_MSG: 
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = sys8/127.0.1.1
STARTUP_MSG:   args = [-format]
STARTUP_MSG:   version = 1.2.1
STARTUP_MSG:   build = https://svn.apache.org/repos/asf/hadoop/common/branches/branch-1.2 -r 1503152; compiled by 'mattf' on Mon Jul 22 15:23:09 PDT 2013
STARTUP_MSG:   java = 1.7.0_79
************************************************************/
15/09/12 19:01:49 INFO util.GSet: Computing capacity for map BlocksMap
15/09/12 19:01:49 INFO util.GSet: VM type       = 64-bit
15/09/12 19:01:49 INFO util.GSet: 2.0% max memory = 932184064
15/09/12 19:01:49 INFO util.GSet: capacity      = 2^21 = 2097152 entries
15/09/12 19:01:49 INFO util.GSet: recommended=2097152, actual=2097152
15/09/12 19:01:49 INFO namenode.FSNamesystem: fsOwner=hadoop
15/09/12 19:01:49 INFO namenode.FSNamesystem: supergroup=supergroup
15/09/12 19:01:49 INFO namenode.FSNamesystem: isPermissionEnabled=true
15/09/12 19:01:49 INFO namenode.FSNamesystem: dfs.block.invalidate.limit=100
15/09/12 19:01:49 INFO namenode.FSNamesystem: isAccessTokenEnabled=false accessKeyUpdateInterval=0 min(s), accessTokenLifetime=0 min(s)
15/09/12 19:01:49 INFO namenode.FSEditLog: dfs.namenode.edits.toleration.length = 0
15/09/12 19:01:49 INFO namenode.NameNode: Caching file names occuring more than 10 times 
15/09/12 19:01:49 INFO common.Storage: Image file /app/hadoop/tmp/dfs/name/current/fsimage of size 112 bytes saved in 0 seconds.
15/09/12 19:01:49 INFO namenode.FSEditLog: closing edit log: position=4, editlog=/app/hadoop/tmp/dfs/name/current/edits
15/09/12 19:01:49 INFO namenode.FSEditLog: close success: truncate to 4, editlog=/app/hadoop/tmp/dfs/name/current/edits
15/09/12 19:01:49 INFO common.Storage: Storage directory /app/hadoop/tmp/dfs/name has been successfully formatted.
15/09/12 19:01:49 INFO namenode.NameNode: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at sys8/127.0.1.1
************************************************************/
{% endhighlight %}


***

##Start the Single node

**NOTE** : 

`bin/start-all.sh` is deprecated so we will use `bin/start-dfs.sh` and then `bin/start-mapred.dfs` 

{% highlight bash %}
hadoop@sys8:~/hadoop$ bin/start-dfs.sh 
starting namenode, logging to /home/hadoop/hadoop/libexec/../logs/hadoop-hadoop-namenode-sys9.out
localhost: starting datanode, logging to /home/hadoop/hadoop/libexec/../logs/hadoop-hadoop-datanode-sys9.out
localhost: starting secondarynamenode, logging to /home/hadoop/hadoop/libexec/../logs/hadoop-hadoop-secondarynamenode-sys9.out
hadoop@sys8:~/hadoop$ bin/start-mapred.sh 
starting jobtracker, logging to /home/hadoop/hadoop/libexec/../logs/hadoop-hadoop-jobtracker-sys9.out
localhost: starting tasktracker, logging to /home/hadoop/hadoop/libexec/../logs/hadoop-hadoop-tasktracker-sys9.out
{% endhighlight %}


### Check if everything is running fine:

Run `jps` for that and the output should look something like this

{% highlight bash %}
hadoop@sys8:~/hadoop$ jps
11508 TaskTracker
10938 NameNode
11868 Jps
11250 SecondaryNameNode
11347 JobTracker
11085 DataNode
hadoop@sys8:~/hadoop$ 
{% endhighlight %}


***

## Stopping hadoop

{% highlight bash %}
hadoop@sys8:~/hadoop$ bin/stop-all.sh
{% endhighlight %}

That's all Folks!
