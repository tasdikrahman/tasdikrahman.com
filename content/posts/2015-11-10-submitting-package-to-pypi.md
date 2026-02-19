---
title: "Submitting python package to pypi"
description: "Submitting python package to pypi"
tags: [python, pypi, ubuntu]
comments: true
share: true
cover_image: '/content/images/2015/12/pypi.png'
---

Recently I had written a [thin wrapper around getziptastic's API](https://github.com/tasdikrahman/pyzipcode-cli/) and I wanted that to be availble as a [pypi package](pypi.python.org/pypi). 

## What is `PyPI`?

From the official website:

#### `PyPI` — the Python Package Index
The Python Package Index is a repository of software for the Python programming language.
Written something cool? Want others to be able to install it with easy_install or pip? Put your code on PyPI. It's a big list of python packages that you absolutely must submit your package to for it to be easily one-line installable.

The good news is that submitting to PyPI is simple in theory: just sign up and upload your code, all for free. The bad news is that in practice it's a little bit more complicated than that. The other good news is that I've written this guide, and that if you're stuck, you can always refer to the official documentation.

Create your accounts

On [PyPI Live](http://pypi.python.org/pypi?%3Aaction=register_form) and also on [PyPI Test](http://testpypi.python.org/pypi?%3Aaction=register_form). You must create an account in order to be able to upload your code. I recommend using the same email/password for both accounts, just to make your life easier when it comes time to push.

## Create a `.pypirc` configuration file

This file holds your information for authenticating with PyPI, both the live and the test versions. This file should be placed in your home directory. So do a 

{% highlight bash %}$ nano ~/.pypirc{% endhighlight %}

Where you can use your favorite text editor in place of `nano`

{% highlight bash %}
[distutils] # this tells distutils what package indexes you can push to
index-servers =
  pypi
  pypitest

[pypi]
repository: https://pypi.python.org/pypi
username: user_name
password: your_password

[pypitest]
repository: https://testpypi.python.org/pypi
username: user_name
password: your_password

[server-login]
repository: https://testpypi.python.org/pypi
username: user_name
password: your_password
{% endhighlight %}

This is just to make your life easier, so that when it comes time to upload you don't have to type/remember your username and password. 

## Prepare your package

Every package on PyPI needs to have a file called `setup.py` at the root of the directory. If your'e using a markdown-formatted read me file you'll also need a `setup.cfg` file. Also, you'll want a `LICENSE.txt` file describing what can be done with your code. So if I've been working on a library called mypackage, my directory structure would look like this:

{% highlight bash %}
.
├── LICENSE.txt
├── pyzipcode-cli
│   ├── countries.json
│   ├── __init__.py
│   └── pyzipcode-cli.py
├── README.md
├── requirements.txt
├── setup.cfg
├── setup.py
└── usage.gif
{% endhighlight %}

Here's a breakdown of what goes in which file:

#### `setup.py`

This is metadata about your library.

{% highlight bash %}
#!/usr/bin/env python
try:
  import os
  from setuptools import setup, find_packages
except ImportError:
  from distutils.core import setup

setup(
  name = 'pyzipcode-cli',
  version = '0.0.12',
  author = 'Tasdik Rahman',
  author_email = 'tasdikrahman@zoho.com',
  # packages = ['pyzipcode_cli'], 
  description = "a thin wrapper around getziptastic's API v2",
  url = 'https://github.com/tasdikrahman/pyzipcode-cli', 
  license = 'MIT',
  install_requires = [
    "docopt==0.6.1",
    "requests==2.8.1"
  ],
  ### adding package data to it 
  packages=find_packages(exclude=['contrib', 'docs', 'tests']),
  package_data={
      'pyzipcode_cli': ['*.json'],
  },

  ###
  download_url = 'https://github.com/tasdikrahman/pyzipcode-cli/tarball/0.0.12', 
  classifiers = [
      'Intended Audience :: Developers',
      'Topic :: Software Development :: Build Tools',
      'License :: OSI Approved :: MIT License',

      # Specify the Python versions you support here. In particular, ensure
      # that you indicate whether you support Python 2, Python 3 or both.
      'Programming Language :: Python :: 2.7',
      'Programming Language :: Python :: 3.4',
  ],
  keywords = ['api', 'geo-location', 'zipcode','devtools', 'Development', 'ziptastic'], 
  entry_points = {
        'console_scripts': [
            'pyzipcode = pyzipcode_cli.core:main'
      ],
    }
)
{% endhighlight %}

The `download_url` is a link to a hosted file with your repository's code. Github will host this for you, but only if you create a git tag. 

In your repository, type: 

{% highlight bash %}$ git tag 0.1 -m "Adds a tag so that we can put this on PyPI."{% endhighlight %}
 
Then, type `git tag` to show a list of tags — you should see 0.1 in the list. 

Type 

{% highlight bash %}
git push --tags origin master
{% endhighlight %}

 to update your code on Github with the latest tag information. Github creates tarballs for download at `https://github.com/{username}/{module_name}/tarball/{tag}`.

#### `setup.cfg`

This tells PyPI where your README file is.


{% highlight bash %}
[metadata]
description-file = README.md
{% endhighlight %}

This is necessary if you're using a markdown readme file. At upload time, you may still get some errors about the lack of a readme — don't worry about it. If you don't have to use a markdown `README` file, I would recommend using reStructuredText (REST) instead.

#### `LICENSE.txt`

This file will contain whichver license you want your code to have. I tend to use the [MIT license](prodicus.mit-license.org).

## Upload your package to PyPI Test

Run:

{% highlight bash %}
$ python setup.py register -r pypitest
{% endhighlight %}

This will attempt to register your package against PyPI's test server, just to make sure you've set up everything correctly.

Then, run:

{% highlight bash %}
$ python setup.py sdist upload -r pypitest
{% endhighlight %}

You should get no errors, and should also now be able to see your library in the test PyPI repository.

## Upload to PyPI Live

Once you've successfully uploaded to PyPI Test, perform the same steps but point to the live PyPI server instead. To register, run:

{% highlight bash %}
$ python setup.py register -r pypi
{% endhighlight %}

Then, run:

{% highlight bash %}
$ python setup.py sdist upload -r pypi
{% endhighlight %}

and you're done!

## Some shameless promotion

If you want to try my package out here is the 

- [github link to the repo](https://github.com/tasdikrahman/pyzipcode-cli/)
- [https://pypi.python.org/pypi/pyzipcode-cli/](https://pypi.python.org/pypi/pyzipcode-cli/)

My cool looking badge :D

[![PyPI version](https://badge.fury.io/py/pyzipcode-cli.svg)](https://badge.fury.io/py/pyzipcode-cli) 


## References:

* [http://peterdowns.com/posts/first-time-with-pypi.html](http://peterdowns.com/posts/first-time-with-pypi.html)
* [https://the-hitchhikers-guide-to-packaging.readthedocs.org/en/latest/quickstart.html#lay-out-your-project](https://the-hitchhikers-guide-to-packaging.readthedocs.org/en/latest/quickstart.html#lay-out-your-project)
* [http://stackoverflow.com/questions/3658084/how-to-send-a-package-to-pypi?rq=1](http://stackoverflow.com/questions/3658084/how-to-send-a-package-to-pypi?rq=1)
* [http://packages.python.org/an_example_pypi_project/](http://packages.python.org/an_example_pypi_project/)
* [https://wiki.python.org/moin/CheeseShopTutorial](https://wiki.python.org/moin/CheeseShopTutorial)
* [http://docs.python.org/distutils/index.html](http://docs.python.org/distutils/index.html)
* [http://zetcode.com/articles/packageinpython/](http://zetcode.com/articles/packageinpython/)
* [http://stackoverflow.com/questions/18787036/difference-between-entry-points-console-scripts-and-scripts-in-setup-py](http://stackoverflow.com/questions/18787036/difference-between-entry-points-console-scripts-and-scripts-in-setup-py)
* [http://stackoverflow.com/questions/23324353/pros-and-cons-of-script-vs-entry-point-in-python-command-line-scripts?lq=1](http://stackoverflow.com/questions/23324353/pros-and-cons-of-script-vs-entry-point-in-python-command-line-scripts?lq=1)
