---
title: "Converting python script into an executable"
description: "Converting python script into an executable"
tags: [python, gui]
comments: true
share: true
cover_image: '/content/images/2014/11/theme-vs-style.png'
---

## BackDrop: 

So recently I was building a Calculator using `tkinter`. 

>I wrote a [blog post](http://www.tasdikrahman.com/2015/11/06/Building-a-calculator/) for the same some time back.

Now I thought, how awesome it would be if I could distribute it to my friends and let the use it. Problem was that some of them not being CS grads would not know head or tails about how to run it!

So I thought the best way and the easiest way was to convert the `pyCalc.py` into a `.exe` file.

That way both my purposes were solved.

- non-cs people would get an interface to run it which was familiar to them. Heck even a granny who knows how to use chrome to watch cookery shows can use it now. Ok, that was a little bit too much
- They didn't have to install anything in this process

## Enter [pyinstaller](https://github.com/pyinstaller/pyinstaller/)

>Note: Before installing PyInstaller on Windows, you will need to install `PyWin32`. You do not need to do this for GNU/Linux or Mac OS X systems.

To install it. You just have to do 

`$ sudo pip install pyinstaller` for `python2.*`

or 

`$ sudo pip3 install pyinstaller` for `python3.*`

If you are behind a proxy server, just add `-E` flag like this `sudo -E pip3 ..`

## Creating the executable

I have my `pyCalc.py` which I want to make an executable of, in 

{% highlight bash%}
tasdik@Acer:~/Desktop/pyCalc$ tree
.
└── pyCalc.py

0 directories, 1 file
tasdik@Acer:~/Desktop/pyCalc$
{% endhighlight %}

To build the executable 

{% highlight bash%}
tasdik@Acer:~/Desktop/pyCalc$ pyinstaller --onefile --windowed pyCalc.py
{% endhighlight %}

Yes, it's that easy!

If you are not haunted with any errors. You should see two folders being placed in `pyCalc`

{% highlight bash %}
tasdik@Acer:~/Desktop/pyCalc$ tree
.
├── build
│   └── pyCalc
│       ├── base_library.zip
│       ├── localpycos
│       │   ├── pyimod01_os_path.pyc
│       │   ├── pyimod02_archive.pyc
│       │   ├── pyimod03_importers.pyc
│       │   └── struct.pyo
│       ├── out00-Analysis.toc
│       ├── out00-EXE.toc
│       ├── out00-PKG.pkg
│       ├── out00-PKG.toc
│       ├── out00-PYZ.pyz
│       ├── out00-PYZ.toc
│       ├── out00-Tree.toc
│       ├── out01-Tree.toc
│       └── warnpyCalc.txt
├── dist
│   └── pyCalc
├── pyCalc.py
└── pyCalc.spec

4 directories, 17 files
tasdik@Acer:~/Desktop/pyCalc$ 
{% endhighlight %}

For a successful build , the final executable, `pyCalc`, and any associated files, will be placed in the `dist` directory, which will be created if it doesn’t exist.

Let me briefly describe the options that are being used:

* `--onefile` is used to package everything into a single executable. If you do not specify this option, the libraries, etc. will be distributed as separate files alongside the main executable.
* `--windowed` prevents a console window from being displayed when the application is run. If you’re releasing a non-graphical application (i.e. a console application), you do not need to use this option.
* `pyCalc.py` the main source file of the application. The basename of this script will be used to name of the executable, however you may specify an alternative executable name using the `--name` option.


See the PyInstaller Manual for more configuration information.

Sadly, we can't make `windows` executables from `pyinstaller`! It supported it some time back in the earlier versions but not in the newer versions.

## Alternatives to `pyinstaller`

* [http://www.py2exe.org/](http://www.py2exe.org/)
* [http://nuitka.net/](http://nuitka.net/)
* [http://wiki.python.org/moin/Py2Exe](http://wiki.python.org/moin/Py2Exe)
* [http://cx-freeze.sourceforge.net/](http://cx-freeze.sourceforge.net/)

Till then. Goodbye!

## References:

* [https://mborgerson.com/creating-an-executable-from-a-python-script](https://mborgerson.com/creating-an-executable-from-a-python-script)
* [http://stackoverflow.com/questions/5458048/how-to-make-a-python-script-standalone-executable-to-run-without-any-dependency](http://stackoverflow.com/questions/5458048/how-to-make-a-python-script-standalone-executable-to-run-without-any-dependency)