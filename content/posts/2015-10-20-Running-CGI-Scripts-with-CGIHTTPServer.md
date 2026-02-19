---
title: "Running CGI Scripts with CGIHTTPServer"
description: "CGI Scripts with pythons built in server"
tags: [apache, ubuntu, cgi, python]
comments: true
share: true
---

## Intro : 

So why should we be interested in CGI which stands for common gateway interface. Well for instance, try to recall the websites that you have been in the last 1 hour. Now out of those websites, some where static and some were dynamic. The latter would mean that the contents of that website kept changing in real time. 

CGI scripting is helpful when you want to generate content from the data residing in a database. This is not only convenient but also cuts a lot of time.

In my previous article, I showed how to run CGI scripts in an `apache2` webserver. Here is the link, if you wanna take a look at it

>**[Running-CGI-Sripts-on-Apache2-Ubuntu](http://www.tasdikrahman.com/2015/09/30/Running-CGI-Sripts-on-Apache2-Ubuntu/)**

## CGIHTTPServer

I hope that you have python installed in your system. Just to be sure. 

{% highlight bash %}
tasdik@Acer:~$ python --version
Python 2.7.8
tasdik@Acer:~$
{% endhighlight %}

We will be making use of a super simple Web server shipped by default with `python` instead of using a full blown web server software for the sake of understanding.

Now cgi scripts are executable files inside the `cgi-bin` or `htdocs` directory which the web server executes. After which the output of the program is captured in the standard output to be displayed back by the server.

It is the `cgi-bin` directory first where all our executable scripts will reside. 

{% highlight bash %}
tasdik@Acer:~$ cd cgi_demo/
tasdik@Acer:~/cgi_demo$ tree
.
├── cgi-bin
│   └── retrieval.py
└── forms.html

1 directory, 2 files
tasdik@Acer:~/cgi_demo$ chmod +x cgi-bin/retrieval.py
{% endhighlight %}

**NOTE: ** `Don't forget to make your cgi-script Executable`


`/forms.html`

{% highlight html linenos %}
<!DOCTYPE html>
<html>
<head>
    <title>A simple form demonstration</title>
</head>
<body>
    <div style="text-align:center;">
        <h1>User login</h1>
        <form action="/cgi-bin/retrieval.py" method="get">
            username  :  <input type="text" name="username" style="text-align:center;">
            <br><br>
            password   :  <input type="password" name="password" style="text-align:center;">
            <br><br><br>
            <input type="submit" value="Submit">
        </form>
    </div>
</body>
</html>
{% endhighlight %}


`/cgi-bin/retrieval.py`

{% highlight bash linenos %}
#!/usr/bin/env python3.4

import cgi, cgitb
cgitb.enable()		## allows for debugging errors from the cgi scripts in the browser

form = cgi.FieldStorage()

## getting the data from the fields 
first = form.getvalue('username')
last = form.getvalue('password')


print("Content-type:text/html\r\n\r\n")
print("<html>")
print("<head><title>User entered</title></head>")
print("<body>")
print("<h1>User has entered</h1>")
print("<b>Firstname : </b>" + first + "<br>")
print("<br><b>Lastname : </b>" + last + "<br>")
print("")
print("</div>")
print("</body>")
print("</html>")
{% endhighlight %}

## Running the webserver

We will be running our webserver in the `cgi_demo` directory. 

{% highlight bash %}
tasdik@Acer:~/cgi_demo$ python -m CGIHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
{% endhighlight %}

At this point. Our web server is up and running. To check on it.

Go to the link **`http://localhost:8000/forms.html`** in your browser.

You will be displayed with the forms to enter `name` and `password`. At this point, if you have a look at the terminal. You will see

{% highlight bash %}
tasdik@Acer:~/cgi_demo$ python -m CGIHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
127.0.0.1 - - [21/Oct/2015 14:26:32] "GET /forms.html HTTP/1.1" 200 -

{% endhighlight %}

`200` is the response code for the request made by the browser for the file `forms.html`, which was present. So it was served back by the server to the browser.

After filling up the form and submitting it. The form calls `retrieval.py` program and passes the data entered by the user to it using a `GET` request.

You will be redirected to page with a url looking something like

**`http://localhost:8000/cgi-bin/retrieval.py?username=tasdik&password=admin123`**

You will notice that the form data is appended with the url of the program itself. This is the standard way of how the `GET` method passes data onto functions. 

The data part starts after the `?`

After you are redirected. You will notice a change in your terminal window from where you had started the web server

{% highlight bash %}
tasdik@Acer:~/cgi_demo$ python -m CGIHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
127.0.0.1 - - [21/Oct/2015 17:17:32] "GET /forms.html HTTP/1.1" 200 -
127.0.0.1 - - [21/Oct/2015 17:17:37] "GET /cgi-bin/retrieval.py?username=tasdik&password=admin123 HTTP/1.1" 200 -


{% endhighlight %}

The program `retrieval.py` was served successfully to the browser.

That's all for this article. In the next one, I will write about how to talk to an `sqlite3` database using CGI scripts

Till then. Goodbye!