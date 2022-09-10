---
layout: post
title: "Creating a gif of the current window"
description: "creating a screen cast"
tags: [ubuntu, gui]
comments: true
share: true
cover_image: '/content/images/2014/11/style-color-colorstory-01_large_mdpi.jpeg'
---

## BackDrop: 

Everybody at some time of their life as netizen would have seen something like this

![http://i.stack.imgur.com/0B664.gif](http://i.stack.imgur.com/0B664.gif)

How about we create one? 

Well recently when I was building a Calculator app, I wanted a `gif` image to be there in the `README.md` so as to show the usage of the app.

>I wrote a [How hard can Building a calclator be right?](http://www.tasdikrahman.com/2015/11/06/Building-a-calculator/) for the same some time back.

Did some googling and found out `byzanz-record` as the tool perfect for me

## Installation

Beginning for 14.04 and above, it is available in the `universe` repository 

{% highlight bash%}
$ sudo apt-get install byzanz
{% endhighlight %}

If you are on a system older than that

{% highlight bash%}
$ sudo add-apt-repository ppa:fossfreedom/byzanz
$ sudo apt-get update && sudo apt-get install byzanz
{% endhighlight %}

## Usage:

We are gonna be using this tool from the command prompt itself as GUI's would just slow down the process. Now for that we just need four things

* `--x=<your_value>`
* `--y=<your_value>`
* `--width=<your_value>`
* `--height=<your_value>`

Now how do we get that?

run
{% highlight bash%}
$ xwininfo
{% endhighlight %}

and point on the window which you want to record. And it will return you the required values and a little extra information too!

{% highlight bash %}
tasdik@Acer:~/Desktop/pyCalc$ xwininfo

xwininfo: Please select the window about which you
          would like information by clicking the
          mouse in that window.

xwininfo: Window id: 0x740003f "Calculator"

  Absolute upper-left X:  984
  Absolute upper-left Y:  509
  Relative upper-left X:  0
  Relative upper-left Y:  0
  Width: 284
  Height: 169
  Depth: 24
  Visual: 0x20
  Visual Class: TrueColor
  Border width: 0
  Class: InputOutput
  Colormap: 0x22 (installed)
  Bit Gravity State: NorthWestGravity
  Window Gravity State: NorthWestGravity
  Backing Store State: NotUseful
  Save Under State: no
  Map State: IsViewable
  Override Redirect State: no
  Corners:  +984+509  -98+509  -98-90  +984-90
  -geometry 284x169-88-80

tasdik@Acer:~/Desktop/pyCalc$
{% endhighlight %}

There now, I have got my values

{% highlight bash%}
tasdik@Acer:~/Desktop/pyCalc$ byzanz-record --duration=45 --x=984 --y=509 --width=290 --height=170 out.gif
{% endhighlight %}

This will immediately start the recording for the `Calculator` window until 45 seconds. So be sure to finish whatever you want to do before that.

Here's is what I got from the above script

![https://raw.githubusercontent.com/tasdikrahman/pyCalc/master/pyCalc_usage.gif](https://raw.githubusercontent.com/tasdikrahman/pyCalc/master/pyCalc_usage.gif)


## References:

* [http://askubuntu.com/questions/107726/how-to-create-animated-gif-images-of-a-screencast](http://askubuntu.com/questions/107726/how-to-create-animated-gif-images-of-a-screencast)
* [http://askubuntu.com/questions/4428/how-to-create-a-screencast](http://askubuntu.com/questions/4428/how-to-create-a-screencast)