---
title: "Apache2 : Virtual Hosts"
description: "Setting up website on localhost using LAMP stack"
tags: [apache, webservers, lampstack]
comments: true
share: true
---

## What do we need?
The two most common tools for this are the Apache and nginx servers.

### Notes:

You'll need to edit a few system configuration files. If you're uncomfortable with vim, replace vim with nano, or gedit in the following commands. For example, sudo vim will become `sudo -H gedit` or `sudo nano`.

Once you're done setting it up, have a look at How to avoid using sudo when working in `/var/www`?
A more detailed guide is available from the Ubuntu LTS Server Guide.

### First, install Apache:

{% highlight bash %}
tasdik@Acer:~$ sudo apt-get install apache2
{% endhighlight %}

The Apache configuration files are located in `/etc/apache2`. You'll typically be interested in:

* `/etc/apache2/sites-available` - contains the Virtual Host definitions. Definitions are enabled and disabled using the `a2ensite` and `a2dissite`commands. The enabled site definitions are linked to `/etc/apache2/sites-enabled`.

* `/etc/apache2/conf-available` - contains custom configuration files. They are enabled and disabled using the `a2enconf` and `a2disconf` commands.
The enabled site configuration files are linked to `/etc/apache2/conf-enabled`.

* `/var/www/html` - the default directory that Apache serves.

* For most instructions, I'll assume we are in `/etc/apache2`.
 
### VirtualHost setup

Let us create a new site. There's a default configuration available in `sites-enabled/default.conf`. We will make a copy of this, and work on it:

This is where should be

{% highlight bash %}
tasdik@Acer:/etc/apache2$ ls
apache2.conf  conf-available  conf-enabled  envvars  magic  mods-available  mods-enabled  ports.conf  sites-available  sites-enabled
tasdik@Acer:/etc/apache2$ 
{% endhighlight %}


{% highlight bash %}
tasdik@Acer:~$ sudo cp sites-available/000-default.conf sites-available/my-name.conf
tasdik@Acer:~$ sudo nano sites-available/my-name.conf
{% endhighlight %}


It should look something like this

{% highlight bash linenos %}
<VirtualHost *:80>
	# The ServerName directive sets the request scheme, hostname and port that
	# the server uses to identify itself. This is used when creating
	# redirection URLs. In the context of virtual hosts, the ServerName
	# specifies what hostname must appear in the request's Host: header to
	# match this virtual host. For the default virtual host (this file) this
	# value is not decisive as it is used as a last resort host regardless.
	# However, you must set it for any further virtual host explicitly.
	ServerName myname.com

	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/my-name

	# Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
	# error, crit, alert, emerg.
	# It is also possible to configure the loglevel for particular
	# modules, e.g.
	#LogLevel info ssl:warn

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

	# For most configuration files from conf-available/, which are
	# enabled or disabled at a global level, it is possible to
	# include a line for only one particular virtual host. For example the
	# following line enables the CGI configuration for this host only
	# after it has been globally disabled with "a2disconf".
	#Include conf-available/serve-cgi-bin.conf
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
{% endhighlight %}


### Save the file, and enable it:

{% highlight bash %}
tasdik@Acer:~$ sudo a2ensite my-name
{% endhighlight %}

Now, we need to set up the directory for the site:

{% highlight bash %}
tasdik@Acer:~$ sudo mkdir /var/www/my-name
{% endhighlight %}


We'll set permissions for convenience:


{% highlight bash %}
tasdik@Acer:~$ sudo chown $USER:www-data /var/www/my-name
tasdik@Acer:~$ sudo chmod g+s /var/www/my-name
{% endhighlight %}

Add a few HTML files here.

Since the virtual host is to run locally, we need to map myname.com to a local address. To do this, we need to edit `/etc/hosts`:

{% highlight bash %}
tasdik@Acer:~$ sudo nano /etc/hosts
{% endhighlight %}


It should look something like this

{% highlight bash linenos %}
127.0.0.1       localhost
127.0.1.1       Acer
127.0.0.2       myname.com   myname


# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
{% endhighlight %}


### Save, and then restart Apache:

{% highlight bash %}
tasdik@Acer:~$ sudo service apache2 restart
{% endhighlight %}


Now, you can browse to `http://myname.com` or `http://myname`, and the contents of `/var/www/my-name` will be displayed.

