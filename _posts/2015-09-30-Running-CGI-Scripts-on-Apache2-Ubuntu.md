---
layout: post
title: "Running CGI Scripts on Apache2"
description: "CGI Scripts on Apache2"
tags: [apache, ubuntu, cgi, python]
comments: true
share: true
---

## Intro

Have you ever wanted to create a webpage or process user input from a web-based form using Python? These tasks can be accomplished through the use of Python CGI (Common Gateway Interface) scripts with an Apache web server. CGI scripts are called by a web server when a user requests a particular URL or interacts with the webpage (such as clicking a "Submit" button). After the CGI script is called and finishes executing, the output is used by the web server to create a webpage displayed to the user.

If you just want to test the waters in the CGI world, you might wanna test your scripts in a simple server like the one shipped with python default.

I have written an article on that. Here's the link.

**[Running-CGI-Sripts-with-CGIHTTPServer](http://www.tasdikrahman.com/2015/10/20/Running-CGI-Sripts-with-CGIHTTPServer/)**

## Configuring the Apache2 Web server to run CGI scripts

I am assuming that you are using `apache2` version `2.4.*`,  as in `Apache2.4`, the configuration was cleaned up considerably, and things in the default site definition have been moved to configuration files in conf-available. Among other things, this also includes the CGI-related configuration lines seen in the default site of older versions. These have been moved to `/etc/apache2/conf-available/serve-cgi-bin.conf`, which contains:

```sh
ScriptAlias /cgi-bin/ /usr/lib/cgi-bin/
```

Just to check whether you have `apache2` installed on your system and its up and running do a

```sh
tasdik@Acer:~$ apache2 -v
Server version: Apache/2.4.10 (Ubuntu)
Server built:   Mar  5 2015 18:13:03
tasdik@Acer:~$
```

I am currently on version `2.4`. If you don't get any output from the prompt its likely that you don't have it installed on your system. Just do a

```sh
tasdik@Acer:~$ sudo apt-get install apache2
```

Anyways, You just need to make changes on two configuration files, them being

* `/etc/apache2/apache2.conf`
* `/etc/apache2/conf-available/serve-cgi-bin.conf`

On the terminal

```sh
tasdik@Acer:~$ mkdir /var/www/cgi-bin
tasdik@Acer:~$ cd /var/www/cgi-bin/
tasdik@Acer:/var/www/cgi-bin$ sudo nano /etc/apache2/apache2.conf
```

And add the following at the end


```sh
###################################################################
#########     Adding capaility to run CGI-scripts #################
ServerName localhost
ScriptAlias /cgi-bin/ /var/www/cgi-bin/
Options +ExecCGI
AddHandler cgi-script .cgi .pl .py
```

This converts the `cgi-bin` address to the specified address and the `AddHandler` tells the server to treat all files with `.cgi`,`.pl` and `.py` extensions as cgi scripts.

Now for the second conf file

```sh
tasdik@Acer:~$ sudo nano /etc/apache2/conf-available/serve-cgi-bin.conf
```

The final file should look something like this

```sh
<IfModule mod_alias.c>
	<IfModule mod_cgi.c>
		Define ENABLE_USR_LIB_CGI_BIN
	</IfModule>

	<IfModule mod_cgid.c>
		Define ENABLE_USR_LIB_CGI_BIN
	</IfModule>

	<IfDefine ENABLE_USR_LIB_CGI_BIN>
		#ScriptAlias /cgi-bin/ /usr/lib/cgi-bin/
		#<Directory "/usr/lib/cgi-bin">
		#	AllowOverride None
		#	Options +ExecCGI -MultiViews +SymLinksIfOwnerMatch
		#	Require all granted
		#</Directory>

		## cgi-bin config
		ScriptAlias /cgi-bin/ /var/www/cgi-bin/
	    <Directory "/var/www/cgi-bin/">
	        AllowOverride None
	        Options +ExecCGI
	    </Directory>

	</IfDefine>
</IfModule>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

Now restart the `apache2`

```sh
tasdik@Acer:~$ sudo service apache2 restart
```

## Creating a simple CGI script

We have to first create the folder that will hold the cgi-scripts, so lets do that.

```sh
tasdik@Acer:~$ cd /var/www/cgi-bin
tasdik@Acer:/var/www/cgi-bin$ touch hello.py
##  make it executable for you and others

tasdik@Acer:/var/www/cgi-bin$ chmod o+x hello.py

##  Put the following content just for testing inside `hello.py`
tasdik@Acer:/var/www/cgi-bin$ sudo nano hello.py
```

Add the following to `hello.py`

```python
#!/usr/bin/env python

import cgitb
cgitb.enable()
print("Content-Type: text/html;charset=utf-8")

print "Content-type:text/html\r\n\r\n"
print '<html>'
print '<head>'
print '<title>Hello Word - First CGI Program</title>'
print '</head>'
print '<body>'
print '<h2>Hello Word! This is my first CGI program</h2>'
print '</body>'
print '</html>'
```

## Run the Script:

Open your browser and enter the following link

[http://localhost/cgi-bin/hello.py](http://localhost/cgi-bin/hello.py)

And the script should run just fine.

## Debugging :

If the script is not running, you can check the logs stored in

`/var/log/apache2/error.log`

You can also refer the official reference here : [http://httpd.apache.org/docs/2.0/howto/cgi.html](http://httpd.apache.org/docs/2.0/howto/cgi.html)

Hope it helped!

## References:

* [http://httpd.apache.org/docs/2.0/howto/cgi.html](http://httpd.apache.org/docs/2.0/howto/cgi.html)
* [http://askubuntu.com/questions/14763/where-are-the-apache-and-php-log-files](http://askubuntu.com/questions/14763/where-are-the-apache-and-php-log-files)
* [http://askubuntu.com/questions/679961/apache2-4-10-on-ubuntu-returning-internal-server-error-on-running-cgi-scripts/680039#680039](http://askubuntu.com/questions/679961/apache2-4-10-on-ubuntu-returning-internal-server-error-on-running-cgi-scripts/680039#680039)
