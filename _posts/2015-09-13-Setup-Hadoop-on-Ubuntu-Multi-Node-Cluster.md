---
layout: post
title: "Install Hadoop(Multi Node)"
description: "Install Hadoop Multi Node 1.0.3 on Ubuntu 14.04.2"
tags: [hadoop, bigdata, ubuntu]
comments: true
share: true
---


## Intro: 

  In this article,  I will describe the required steps for setting up a distributed, multi-node  Apache Hadoop cluster backed by the Hadoop Distributed File System (HDFS), running on Ubuntu Linux.

  I am using `nano` as the text editor for this article but you can use any other text-editor like `vi`, `sublime` or `atom` for doing the same

  >I suggest you set up hadoop on single node before directly going for multi node as it will be easier to debug any errors. I have explained about [how to setup single node hadoop on ubuntu](http://www.tasdikrahman.com/2015/09/10/Install-Hadoop-1.0.3-Single-Node-on-Ubuntu-14.04.2/)
 
  **Note** : 

  I will be demonstrating for 3 nodes but you can add more nodes as you like. 


## Installation

  >Install for the three node [using this tutuorial](http://www.tasdikrahman.com/2015/09/10/Install-Hadoop-1.0.3-Single-Node-on-Ubuntu-14.04.2/)



## Done? Let's continue then!

The very first step would be stop `hadoop` on all the three machines.

For that, you just need to do

{% highlight bash %}
hadoop@sys9:~/hadoop$ bin/stop-all.sh
{% endhighlight %}


on each of the nodes

Now to check whether hadoop processes are stopped in all the nodes or not!

**For sys9**

{% highlight bash %}
hadoop@sys9:~/hadoop$ jps
12792 Jps
hadoop@sys9:~/hadoop$
{% endhighlight %}



**For sys10**

{% highlight bash %}
hadoop@sys10:~/hadoop$ jps
12637 Jps
hadoop@sys10:~/hadoop$
{% endhighlight %}


**For sys8**

{% highlight bash %}
hadoop@sys8:~$ jps
2186 Jps
hadoop@sys8:~$ 
{% endhighlight %}

Now we have to choose which one node will be the master node, so that the other two nodes can be the slaves

In my case, I will chose `sys8` to be my `master` node and the others to be `slaves` 

## Master node Configuration : 

Add the the ip address of the slaves to `/etc/hosts` on `sys8`

{% highlight bash %}
hadoop@sys8:~$ sudo nano /etc/hosts
{% endhighlight %}


It should look something like this after you have added it.

{% highlight bash %}
### master node
hadoop@sys8:~$ cat /etc/hosts
192.168.103.26  sys9          ## slave
192.168.103.28  sys10         ## slave
hadoop@sys8:~$ 
{% endhighlight %}


For `sys9`

{% highlight bash %}
### for slave node
hadoop@sys9:~$ cat /etc/hosts
192.168.103.24  sys8          ## master
192.168.103.26  sys9          ## slave
192.168.103.28  sys10         ## slave
hadoop@sys9:~$ 
{% endhighlight %}


For `sys10`

{% highlight bash %}
### for slave node
hadoop@sys10:~$ cat /etc/hosts
192.168.103.24  sys8          ## master
192.168.103.26  sys9          ## slave
192.168.103.28  sys10         ## slave
hadoop@sys10:~$
{% endhighlight %}


## Changing the hadoop configurations : 

Shift to the `hadoop` directory first 

{% highlight bash %}
hadoop@sys8:~$ cd ~/hadoop/conf
hadoop@sys8:~/hadoop/conf$  
{% endhighlight %}


### Editing `conf/core-site.xml` (all nodes)

{% highlight bash %}
hadoop@sys8:~/hadoop/conf$ sudo nano core-site.xml 
{% endhighlight %}

After editing, it should look like

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
  <value>hdfs://192.168.103.24:54310</value>
  <description>The name of the default file system.  A URI whose
  scheme and authority determine the FileSystem implementation.  The
  uri's scheme determines the config property (fs.SCHEME.impl) naming
  the FileSystem implementation class.  The uri's authority is used to
  determine the host, port, etc. for a filesystem.</description>
 </property>
</configuration>
{% endhighlight %}


similarly for nodes `sys9` and `sys10`.


**Note**

Here `192.168.103.24` is the ip address of the master node. 
We just have replaced `localhost` with this ip address


### Editing `conf/mapred-site.xml` (All nodes)

After editing, the file should be

{% highlight xml linenos %}
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<!-- Put site-specific property overrides in this file. -->

<configuration>
 <property>
  <name>mapred.job.tracker</name>
  <value>192.168.103.24:54311</value>
  <description>The host and port that the MapReduce job tracker runs
  at.  If "local", then jobs are run in-process as a single map
  and reduce task.
  </description>
</property>
</configuration>
{% endhighlight %}


similarly for nodes `sys9` and `sys10`.

**Note** : 

The file `conf/hdfs-site.xml` remains the same for all the nodes 


### Editing the `conf/masters` (master node only)

{% highlight bash %}
hadoop@sys8:~/hadoop/conf$ sudo nano masters
{% endhighlight %}


It should look like

{% highlight bash %}
hadoop@sys8:~/hadoop/conf$ cat masters
192.168.103.24
hadoop@sys8:~/hadoop/conf$
{% endhighlight %}


### Editing the `conf/slaves` (master node only)

{% highlight bash %}
hadoop@sys8:~/hadoop/conf$ sudo nano slaves
{% endhighlight %}


It should look like

{% highlight bash %}
hadoop@sys8:~/hadoop/conf$ cat slaves
192.168.103.24
192.168.103.26
192.168.103.28
hadoop@sys8:~/hadoop/conf$
{% endhighlight %}



If you have additional slaves to add up, just add those in the `conf/slaves` file after a newline




***




So we have basically added ip's of all the nodes inside the `slaves` file.

###Generate the ssh keys for the master again

{% highlight bash %}
hadoop@sys8:~/hadoop/conf$ ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/hadoop/.ssh/id_rsa): 
/home/hadoop/.ssh/id_rsa already exists.
Overwrite (y/n)? y
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/hadoop/.ssh/id_rsa.
Your public key has been saved in /home/hadoop/.ssh/id_rsa.pub.
The key fingerprint is:
fc:73:90:c9:c9:cc:b7:13:e7:75:55:0b:94:b5:18:01 hadoop@sys8
The key's randomart image is:
+--[ RSA 2048]----+
|           Eo=+..|
|             .+ +|
|             . o.|
|       . = +    .|
|        S X o . o|
|         . o = ..|
|          o + .  |
|           o .   |
|                 |
+-----------------+
hadoop@sys8:~/hadoop/conf$ 
{% endhighlight %}

### Add this key to all the nodes (Including the master)

For `sys9`

{% highlight bash %}
hadoop@sys8:~/hadoop/conf$ ssh-copy-id hadoop@sys9
The authenticity of host 'sys9 (192.168.103.26)' can't be established.
ECDSA key fingerprint is e9:d3:5f:de:5b:0d:93:17:18:16:b8:9b:39:39:fa:62.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
hadoop@sys9's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'hadoop@sys9'"
and check to make sure that only the key(s) you wanted were added.

hadoop@sys8:~/hadoop/conf$
{% endhighlight %}


For `sys10`

{% highlight bash %}
hadoop@sys8:~/hadoop/conf$ ssh-copy-id hadoop@sys10
The authenticity of host 'sys10 (192.168.103.28)' can't be established.
ECDSA key fingerprint is 1b:e2:6c:bc:2d:41:12:4d:79:e1:60:5c:08:74:32:9a.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
hadoop@sys10's password: 
Permission denied, please try again.
hadoop@sys10's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'hadoop@sys10'"
and check to make sure that only the key(s) you wanted were added.

hadoop@sys8:~/hadoop/conf$
{% endhighlight %}


For `sys8` itself!

{% highlight bash %}
hadoop@sys8:~/hadoop/conf$ ssh-copy-id hadoop@sys8
The authenticity of host 'sys4 (192.168.103.24)' can't be established.
ECDSA key fingerprint is 1b:e2:6c:bc:2d:41:12:4d:79:e1:60:5c:08:74:32:9a.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
hadoop@sys4's password: 
Permission denied, please try again.
hadoop@sys4's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'hadoop@sys4'"
and check to make sure that only the key(s) you wanted were added.

hadoop@sys8:~/hadoop/conf$
{% endhighlight %}


## Format the `/app/tmp/` directory contents (master node)

{% highlight bash %}
hadoop@sys8:~$ sudo rm -rf /app/hadoop/tmp/*
hadoop@sys8:~$
{% endhighlight %}


## Format the namenode (master node)

{% highlight bash linenos %}
hadoop@sys8:~/hadoop$ bin/hadoop namenode -format
Warning: $HADOOP_HOME is deprecated.

15/09/13 23:11:22 INFO namenode.NameNode: STARTUP_MSG: 
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = sys8/192.168.103.24
STARTUP_MSG:   args = [-format]
STARTUP_MSG:   version = 1.2.1
STARTUP_MSG:   build = https://svn.apache.org/repos/asf/hadoop/common/branches/branch-1.2 -r 1503152; compiled by 'mattf' on Mon Jul 22 15:23:09 PDT 2013
STARTUP_MSG:   java = 1.7.0_79
************************************************************/
15/09/13 23:11:22 INFO util.GSet: Computing capacity for map BlocksMap
15/09/13 23:11:22 INFO util.GSet: VM type       = 64-bit
15/09/13 23:11:22 INFO util.GSet: 2.0% max memory = 932184064
15/09/13 23:11:22 INFO util.GSet: capacity      = 2^21 = 2097152 entries
15/09/13 23:11:22 INFO util.GSet: recommended=2097152, actual=2097152
15/09/13 23:11:22 INFO namenode.FSNamesystem: fsOwner=hadoop
15/09/13 23:11:22 INFO namenode.FSNamesystem: supergroup=supergroup
15/09/13 23:11:22 INFO namenode.FSNamesystem: isPermissionEnabled=true
15/09/13 23:11:22 INFO namenode.FSNamesystem: dfs.block.invalidate.limit=100
15/09/13 23:11:22 INFO namenode.FSNamesystem: isAccessTokenEnabled=false accessKeyUpdateInterval=0 min(s), accessTokenLifetime=0 min(s)
15/09/13 23:11:22 INFO namenode.FSEditLog: dfs.namenode.edits.toleration.length = 0
15/09/13 23:11:22 INFO namenode.NameNode: Caching file names occuring more than 10 times 
15/09/13 23:11:22 INFO common.Storage: Image file /app/hadoop/tmp/dfs/name/current/fsimage of size 112 bytes saved in 0 seconds.
15/09/13 23:11:22 INFO namenode.FSEditLog: closing edit log: position=4, editlog=/app/hadoop/tmp/dfs/name/current/edits
15/09/13 23:11:22 INFO namenode.FSEditLog: close success: truncate to 4, editlog=/app/hadoop/tmp/dfs/name/current/edits
15/09/13 23:11:23 INFO common.Storage: Storage directory /app/hadoop/tmp/dfs/name has been successfully formatted.
15/09/13 23:11:23 INFO namenode.NameNode: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at sys8/192.168.103.24
************************************************************/
hadoop@sys8:~/hadoop$ 

{% endhighlight %}


## Start the name node (in master node)

{% highlight bash linenos %}
hadoop@sys8:~/hadoop$ bin/start-all.sh 
Warning: $HADOOP_HOME is deprecated.

starting namenode, logging to /home/hadoop/hadoop/libexec/../logs/hadoop-hadoop-namenode-sys8.out
192.168.103.24: starting datanode, logging to /home/hadoop/hadoop/libexec/../logs/hadoop-hadoop-datanode-sys8.out
192.168.103.28: starting datanode, logging to /home/hadoop/hadoop/libexec/../logs/hadoop-hadoop-datanode-sys10.out
192.168.103.26: starting datanode, logging to /home/hadoop/hadoop/libexec/../logs/hadoop-hadoop-datanode-sys9.out
192.168.103.24: starting secondarynamenode, logging to /home/hadoop/hadoop/libexec/../logs/hadoop-hadoop-secondarynamenode-sys8.out
starting jobtracker, logging to /home/hadoop/hadoop/libexec/../logs/hadoop-hadoop-jobtracker-sys8.out
192.168.103.26: starting tasktracker, logging to /home/hadoop/hadoop/libexec/../logs/hadoop-hadoop-tasktracker-sys9.out
192.168.103.28: starting tasktracker, logging to /home/hadoop/hadoop/libexec/../logs/hadoop-hadoop-tasktracker-sys10.out
192.168.103.24: starting tasktracker, logging to /home/hadoop/hadoop/libexec/../logs/hadoop-hadoop-tasktracker-sys8.out
hadoop@sys8:~/hadoop$
{% endhighlight %}


## Check JPS (in all systems)

`sys8`

{% highlight bash %}
hadoop@sys8:~/hadoop$ jps
3526 Jps
2817 NameNode
3258 JobTracker
3153 SecondaryNameNode
2976 DataNode
3442 TaskTracker
hadoop@sys8:~/hadoop$
{% endhighlight %}


`sys9`

{% highlight bash %}
hadoop@sys9:~/hadoop$ jps
2440 TaskTracker
2519 Jps
hadoop@sys9:~/hadoop$
{% endhighlight %}

`sys10`

{% highlight bash %}
hadoop@sys10:~/hadoop$ jps
2549 Jps
2469 TaskTracker
hadoop@sys10:~/hadoop$
{% endhighlight %}

## Check the web interface in your browser

You can also check the web interface of hadoop in your browser

Using the url : `http://192.168.103.24:50030/`


<figure>
  <a href="/images/machine_list_hadoop.jpg"><img src="/images/machine_list_hadoop.jpg" alt="">
  <figcaption>List of all nodes in the Hadoop Cluster</figcaption>
</figure>

***

<figure>
  <a href="/images/machine_list_hadoop.jpg"><img src="/images/machine_list_hadoop.jpg" alt="">
  <figcaption>Mapreduce User Interface in the master node</figcaption>
</figure>


That's all Folks!
