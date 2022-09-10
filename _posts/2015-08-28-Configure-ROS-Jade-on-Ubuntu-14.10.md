---
layout: post
title: "ROS Jade : Configuration"
description: "Configuring ROS - Jade on Ubuntu 14.10"
tags: [ros, ubuntu, robotics]
comments: true
share: true
---


### Configuring ROS

If you have not installed ROS, I have written a short guide describing the process. It can be found here found here

> [Install ROS - Jade on Ubuntu 14.10](https://www.tasdikrahman.com/2015/08/28/Install-ROS-Jade-on-Ubuntu-14.10/)

The first step is to check whether the environment variables for ROS are setup properly
Do a 

{% highlight bash %}
tasdik@Acer:~$ export | grep ROS
{% endhighlight %}


Look for `ROS_ROOT` and `ROS_PACKAGE_PATH` whether they are set.

It should look something like this

<figure>
  <a href="/images/path_ros.jpg"><img src="/images/path_ros.jpg" alt="">
  <figcaption>List of all nodes in the Hadoop Cluster</figcaption>
</figure>

If not, just do a 


{% highlight bash %}
tasdik@Acer:~$ source /opt/ros/<distro>/setup.bash
{% endhighlight %}


where <distro> is <del>the distribution name of you OS.</del>  version of ROS installed (Jade Turtle for our case). 


### Creating ROS Workspace

I will be using [catkin](http://wiki.ros.org/catkin) to create my workspace.


{% highlight bash %}
tasdik@Acer:~$ mkdir -p ~/catkin_ws/src
tasdik@Acer:~$ cd ~/catkin_ws/src
tasdik@Acer:~$ catkin_init_workspace
{% endhighlight %}


Even though the folder is empty(we just have a file named CMakeLists.txt). We can still build the workspace.


{% highlight bash %}
tasdik@Acer:~$ catkin_make
tasdik@Acer:~$ cd ~/catkin_ws/
{% endhighlight %}


Do an `ls` and now you can see that we have folders like `build`, `devel` folder. 
Inside the `devel` folder, there are several `*.sh` files

## Source the setup file


{% highlight bash %}
tasdik@Acer:~/catkin_ws$ source devel/setup.bash
{% endhighlight %}

To make sure, everything is done correctly so far, do

{% highlight bash %}
tasdik@Acer:~/catkin_ws$ echo $ROS_PACKAGE_PATH
/opt/ros/jade/share:/opt/ros/jade/stacks
tasdik@Acer:~/catkin_ws$
{% endhighlight %}


If that is your Output for the prompt, you are good to go.
  

Till then Goodbye!

### References : 


* [http://wiki.ros.org/ROS/Tutorials/InstallingandConfiguringROSEnvironment](http://wiki.ros.org/ROS/Tutorials/InstallingandConfiguringROSEnvironment)