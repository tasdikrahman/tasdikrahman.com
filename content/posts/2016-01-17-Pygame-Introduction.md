---
title: "Getting started with Pygame"
description: "Pygame Introduction"
tags: [gamedevelopment, pygame, python]
comments: true
share: true
cover_image: '/content/images/2016/1/pygame_logo.png'
---

## Pygame intro

As with every other kid out there, I spent long hours sitting in front of the computer playing Games like [Super Mario](https://en.wikipedia.org/wiki/Super_Mario), [Dangerous Dave](https://en.wikipedia.org/wiki/Dangerous_Dave) and the likes. So when I got to know about [Pygame](www.pygame.org/). I was really getting the itch on creating something in the lines of these games.

Of cource, we have better game engines written in other languages like `C++`, but since I liked `python`. So I mean what the heck right?

Lets Get Started then shall we

## Installation

### Ubuntu/Debian

{% highlight bash%}
$ sudo apt-get install python-pygame
{% endhighlight %}

### OS X

If you DON'T have `homebrew` installed, then 

{% highlight bash%}
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
{% endhighlight %}

If you have it installed, then

{% highlight bash%}
$ brew install caskroom/cask/brew-cask
$ brew cask install xquartz
$ brew install python3
$ brew install python
$ brew linkapps python3
$ brew linkapps python
$ brew install git
$ brew install sdl sdl_image sdl_ttf portmidi libogg libvorbis
$ brew install sdl_mixer --with-libvorbis
$ brew tap homebrew/headonly
$ brew install smpeg
$ brew install mercurial
$ pip3 install hg+http://bitbucket.org/pygame/pygame
{% endhighlight %}

## See if it works?

Open the terminal 

{% highlight bash%}
$ python
Python 2.7.8 (default, Jun 18 2015, 18:54:19) 
[GCC 4.9.1] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import pygame
>>>
{% endhighlight %}

If there is no Error on that, then you are good to go.

## A simple Pygame Boilerplate

I made a dead simple `Pygame` boilerplate for creating a base for your `pygame` programs. 

Here is the Github link for you.

- link : [tasdikrahman/pygame-boilerplate](https://github.com/tasdikrahman/pygame-boilerplate)

A sneak peak

<center><img src="http://i.imgur.com/p7PxMZl.jpg"></center>

## What can you do with this?

I created [Space Shooter](https://github.com/tasdikrahman/spaceShooter) using this boilerplate. Here's a demo screen for you

<center><img src="http://i.imgur.com/I5mTBFB.png"></center>

## References

- [http://askubuntu.com/questions/399824/how-to-install-pygame](http://askubuntu.com/questions/399824/how-to-install-pygame)
- [https://inventwithpython.com/pygame/chapter1.html](https://inventwithpython.com/pygame/chapter1.html)
- [http://kidscancode.org/blog/2015/09/pygame_install/](http://kidscancode.org/blog/2015/09/pygame_install/)