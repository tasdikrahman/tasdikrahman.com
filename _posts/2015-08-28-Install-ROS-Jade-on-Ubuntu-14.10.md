---
layout: post
title: "ROS Jade : Installation"
description: "Install ROS - Jade on Ubuntu 14.10"
tags: [ros, ubuntu, robotics]
comments: true
share: true
---

### What the heck is ROS anyway : 

Robot Operating System (ROS) is a collection of software frameworks for robot software development, providing operating system-like functionality on a heterogeneous computer cluster. 

So in layman terms, it just helps us build robot applications.

> Previous post on ROS : [Configuring ROS - Jade on Ubuntu 14.10](http://www.tasdikrahman.com/2015/08/28/Configure-ROS-Jade-on-Ubuntu-14.10/)

### Note : 

ROS Jade ONLY supports Trusty (14.04), Utopic (14.10) and Vivid (15.04) for debian packages. 
If you are on any other version of Ubuntu or have a different flavor of linux installed, I suggest you head over 
to [wiki.ros.org](http://wiki.ros.org/).

Note that this guide is written keeping in mind that we are on Ubuntu 14.10 !



### Requirements : 

* Supported OS : 
  * Ubuntu Trusty(14.04)
  * Ubuntu Utopic (14.10)
  * Ubuntu Vivid (15.04)

* Minimum requirements : 
  * C++03
  * C++11 features are not used, but code should compile when -std=c++11 is used
  * Python 2.7
    *Python 3.3 not required, but testing against it is recommended
  * Lisp SBCL 1.1.14
  * CMake 2.8.12
  * Boost 1.54


### Installation : 


1. **Configure your Ubuntu repositories** :
  * Configure your Ubuntu repositories to allow "restricted," "universe," and "multiverse." You can 
    follow [follow the Ubuntu guide ](https://help.ubuntu.com/community/Repositories/Ubuntu) for completing this work.


2. **Set up your sources.list** 
   
    This can be done by running the following command in the terminal 
    
    {% highlight bash %}
    tasdik@Acer:~$ sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
    {% endhighlight %}

    
3. **Set up your keys**
  
    {% highlight bash %}
    tasdik@Acer:~$ sudo apt-key adv --keyserver hkp://pool.sks-keyservers.net --recv-key 0xB01FA116
    {% endhighlight %}

    
    Note that if you are behind a proxy, this command wont work. You will have to workaround that I guess(use a dongle maybe!)


4. **Installation**
  
  Make sure that Debian package index is up-to-date.
  
  To make sure just do a 

  {% highlight bash %}
  tasdik@Acer:~$ sudo apt-get update
  {% endhighlight %}

  and you are good to go
  

#### For those running 14.04.2


  If you are on 14.04.2, you have to manually fix the dependy issues, 

  >DO NOT INSTALL THE BELOW PACKAGES IF YOU ARE ON 14.04, THIS WILL DESTROY YOUR X-SERVER. IN SHORT, YOU WONT BE ABLE TO SEE THE GUI OF YOUR OS AGAIN!


  {% highlight bash %}
  tasdik@Acer:~$ sudo apt-get install xserver-xorg-dev-lts-utopic mesa-common-dev-lts-utopic libxatracker-dev-lts-utopic libopenvg1-mesa-dev-lts-utopic libgles2-mesa-dev-lts-utopic libgles1-mesa-dev-lts-utopic libgl1-mesa-dev-lts-utopic libgbm-dev-lts-utopic libegl1-mesa-dev-lts-utopic
  {% endhighlight %}

  >DO NOT INSTALL THE ABOVE PACKAGES IF YOU ARE ON 14.04, THIS WILL DESTROY YOUR X-SERVER. IN SHORT, YOU WONT BE ABLE TO SEE THE GUI OF YOUR OS AGAIN!

  Alternatively you can try, 


  {% highlight bash %}
  tasdik@Acer:~$ sudo apt-get install libgl1-mesa-dev-lts-utopic
  {% endhighlight %}

  to fix the dependency issues.

#### For those running 14.04, 14.10 and 15.04


  After that you can run the command to install the **Desktop-Full Install** which is the **recommeded** one by
  doing.
  

  {% highlight bash %}
  tasdik@Acer:~$ sudo apt-get install ros-jade-desktop-full
  {% endhighlight %}

  
  If everything is good uptil here, you should see the package manager downloading the required packages.
  

### **Get yourself a coffee or two**

  because it can take a helluva a time depending on the speed of your internet connection.
  
  
### Initialize rosdep
  
  Before you can use ROS, you will need to initialize rosdep. rosdep enables you to easily install system 
  dependencies for source you want to compile and is required to run some core components in ROS.
  

{% highlight bash %}
tasdik@Acer:~$ sudo rosdep init
tasdik@Acer:~$ rosdep update
{% endhighlight %}


### Environment setup
  
  It's convenient if the ROS environment variables are automatically added to your bash session every time 
  a new shell is launched:
  

{% highlight bash %}
tasdik@Acer:~$ echo "source /opt/ros/jade/setup.bash" >> ~/.bashrc
tasdik@Acer:~$ source ~/.bashrc
{% endhighlight %}
  
  
### Getting rosinstall
  
  [rosinstall](http://wiki.ros.org/rosinstall) is a frequently used command-line tool in ROS that is 
  distributed separately. It enables you to easily download many source trees for ROS packages with one command.
  
  For ubuntu users, just run


{% highlight bash %}
tasdik@Acer:~$ sudo apt-get install python-rosinstall
{% endhighlight %}
  
  
### Finally

  The next step would be to [configure ROS, the steps of which can be found here](http://www.tasdikrahman.com/Configure-ROS-Jade-on-Ubuntu-14.10/)

  Now that you have installed ROS on your system, you can look forward to the 
  [ROS tutorials](http://wiki.ros.org/ROS/Tutorials)


  
Till then Goodbye!

### References : 

I could not have written this guide if the documentation had not been so crisp and straight forward.
Kudos to the ROS development team!

* [http://wiki.ros.org/jade/Installation/Ubuntu](http://wiki.ros.org/jade/Installation/Ubuntu)
