---
layout: page
title: Projects
customjs:
 - https://asciinema.org/a/35557.js
---

A selected list of projects that I have contributed to  or have made. Many more can be found out over at my [Github](https://github.com/tasdikrahman) profile.

## <a name="index"/>Index

_**Click on any link to jump to that project**_

#### Web Apps

- [SRM Search Engine: As the name says](#srmsearch)
- [Plino: RESTful spam filtering API powered by spammy](#plino)

#### Machine Learning

- [Spammy: Spam filtering made easy for the masses](#spammy)
- [Movie Reviews Analysis](#moviereview)

#### API's

- [easyrbac: Role Based Access Control (version 0) implementation](#easyrbac)
- [Markovipy: Random sentence generator based on Markov Chains](#markovipy)
- [vocabulary: Similar to Wordnet, comes with a better API](#vocabulary)
- [pyzipcode: All things possible with a zipcode](#pyzipcode)

#### GUI's

- [thanos: SQL injection demo on SQLite3](#thanos)
- [pyCalc: Cross platform Calculator](#pycalc)

#### Bot's

- [Margo: An opiniated slack bot](#margo)

#### Games anyone?

- [Space Shooter: 2D cross platform FPS](#spaceshooter)

#### Command line apps

- [tnote: Note taking for the command line](#tnote)
- [xkcd-dl: Download all the xkcd's, like ever](#xkcddl)

#### Hackathons

- [Foodoh - Startup Weekend Chennai, 2016](#foodoh)

***

### <a name="srmsearch"/>[SRM Search Engine](http://srmsearchengine.in/se.html)

<center><a href="#" target="_blank"><img src="http://i.imgur.com/rg45z9w.jpg"></a></center>

A general purpose Search Engine which is funded by the [National Internet Exchange](http://nixi.in/), Govt. of India

Covered thrice by Times Of India, titled

- ["College team develops search engine"](http://timesofindia.indiatimes.com/city/chennai/College-team-develops-search-engine/articleshow/45129062.cms)
- ["SRM University to launch search engine"](http://timesofindia.indiatimes.com/city/chennai/SRM-University-to-launch-search-engine/articleshow/48748309.cms)
- ["Smart search"](http://timesofindia.indiatimes.com/home/education/news/Smart-search/articleshow/45253861.cms)

The TV channel ["Vasanth TV"](https://www.youtube.com/watch?v=k-tvEKqdIQ4) aired us once. Yeah thats much about it!


<center><a href="#index"><i class="fa fa-chevron-circle-up fa-2x"></i></a></center>
<!-- [<i class="fa fa-chevron-circle-up fa-2x"></i>Back to top](#index) -->

***

### <a name="plino"/>[Plino](https://plino.herokuapp.com)

<iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=plino&type=star&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=plino&type=watch&count=true&v=2" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=plino&type=fork&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe>

<center><a href="https://plino.herokuapp.com"><img src="http://i.imgur.com/jJfnz7f.jpg"></a></center>

<a href="https://news.ycombinator.com/item?id=11491949"><img src="https://raw.githubusercontent.com/wingify/across-tabs/master/images/hn.png" width="150" height="20"/></a>
<a href="https://www.producthunt.com/posts/plino"><img src="https://raw.githubusercontent.com/wingify/across-tabs/master/images/product_hunt.png" width="100" height="20"/></a>


A high accuracy spam filtering system built on top of a custom Naive Bayes Classifier
to extract specific feature sets. Designed and developed RESTful APIs using Flask for users to integrate
our service with their apps. Implemented Caching for server load reduction. Achieved over 3000 users in
the first version of the application. Classifier was trained against 33,000 emails.

<i class="fa fa-code fa-2x"></i> `python`, `nltk`, `Machine Learning`, `flask`, `REST APIs`, `dill`

As covered by

- [Product Hunt](https://www.producthunt.com/tech/plino)

<center><a href="#index"><i class="fa fa-chevron-circle-up fa-2x"></i></a></center>


***

### <a name="spammy"/>[Spammy](https://github.com/tasdikrahman/spammy)

<iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=spammy&type=star&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=spammy&type=watch&count=true&v=2" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=spammy&type=fork&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe>

<center><a href="https://github.com/tasdikrahman/spammy"><img src="http://i.imgur.com/L8moQ2U.jpg"></a></center>

Custom `Naive Bayes classifier` in the form of pip module , which can be trained on your own dataset for
classifying emails into spam/ham. **Accuracy achieved: 80%-90%**. Blazingly fast once trained. “Dill” was
used to serialize the classifier object for later use. Powers the web app [**Plino**](https://plino.herokuapp.com/)

As covered by

- [Import Python's issue #68](http://importpython.com/newsletter/no/68/)
- ['Awesome Machine Learning' Github](https://github.com/josephmisiti/awesome-machine-learning#natural-language-processing-9)

<i class="fa fa-code fa-2x"></i> `python`, `nltk`, `Machine Learning`, `dill`

<center><a href="#index"><i class="fa fa-chevron-circle-up fa-2x"></i></a></center>

***

### <a name="moviereview"/>[Movie Reviews Analysis](https://github.com/tasdikrahman/movieReviewsAnalysis)

<iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=movieReviewsAnalysis&type=star&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=movieReviewsAnalysis&type=watch&count=true&v=2" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=movieReviewsAnalysis&type=fork&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe>

<center><a href="http://srmsearchengine.in/se.html"><img src="http://i.imgur.com/d7pwSTe.jpg"></a></center>

An analysis of the `movie_reviews` corpus present in the `nltk` corpus by applying various machine learning algorithms (like `MultinomialNB`, `LogisticRegression`, `LinearSVC` and some more)

<i class="fa fa-code fa-2x"></i> `python`, `nltk`, `pickle`, `scikitlearn`

<center><a href="#index"><i class="fa fa-chevron-circle-up fa-2x"></i></a></center>
<!-- [<i class="fa fa-chevron-circle-up fa-2x"></i>Back to top]() -->

***

### <a name="vocabulary"/>[vocabulary](../vocabulary)


<iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=vocabulary&type=star&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=vocabulary&type=watch&count=true&v=2" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=vocabulary&type=fork&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe>


[![vocabulary](https://raw.githubusercontent.com/tasdikrahman/vocabulary/master/assets/usage.gif)](https://github.com/tasdikrahman/vocabulary)


A python module using which you can get `Meaning`, `Synonyms`, `Antonyms`, `translations` and what not for a given word!

<i class="fa fa-code fa-2x"></i> `python`, `requests`

As covered by

- [pycoders issue #195](http://us4.campaign-archive2.com/?u=9735795484d2e4c204da82a29&id=06f1263282)
- [python weekly issue #220](http://us2.campaign-archive2.com/?u=e2e180baf855ac797ef407fc7&id=c3a5d1d4a8)
- [importpython issue #53](http://importpython.com/newsletter/no/53/)


<center><a href="#index"><i class="fa fa-chevron-circle-up fa-2x"></i></a></center>
<!-- [<i class="fa fa-chevron-circle-up fa-2x"></i>Back to top]() -->

***

### <a name="spaceshooter"/>[Space Shooter](https://github.com/tasdikrahman/spaceShooter)


<iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=spaceShooter&type=star&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=spaceShooter&type=watch&count=true&v=2" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=spaceShooter&type=fork&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe>


<center><a href="https://github.com/tasdikrahman/spaceShooter"><img src="/content/images/2016/1/spaceShooter.gif"></a></center>


<a href="https://news.ycombinator.com/item?id=10925168"><img src="https://raw.githubusercontent.com/wingify/across-tabs/master/images/hn.png" width="150" height="20"/></a>
<a href="https://www.producthunt.com/posts/space-shooter"><img src="https://raw.githubusercontent.com/wingify/across-tabs/master/images/product_hunt.png" width="100" height="20"/></a>


Go back to the memory lane by playing **spaceShooter** on your system.

<i class="fa fa-code fa-2x"></i> `python`, `pygame`, `cxFreeze`, `pyinstaller`, `sprites`

As covered by

- [Product hunt](https://www.producthunt.com/games/space-shooter)
- [pythondigest.ru on issue no #109](http://pythondigest.ru/issue/109/)
- [importpython.com on issue no #58](http://importpython.com/newsletter/no/58/)
- [weekly.pychina.org on issue #58](http://weekly.pychina.org/importpython/importpython-58.html)
- [dighub.ru on issue #35](http://dighub.ru/digests/35/)
- [oschina.net](http://www.oschina.net/p/spaceshooter): An open source community supported by Ministry of OSS promotion Union
- [mybridge.co](http://www.mybridge.co/view/28020): A `San Francisco` based news aggregator.
- [recordnotfound.com](https://recordnotfound.com/spaceShooter-tasdikrahman-5374): A curated list of open source projects
- [neuropuff.com](https://neuropuff.com/post/show-hn-space-shooter-retro-game-recreated-using-python): a programming blog

**Wanna play? Requires no installation! Just unzip it and you are good to go!**

| <i class="fa fa-linux fa-2x"></i>   | <a href="https://github.com/tasdikrahman/spaceShooter/releases/download/v0.0.3/spaceShooter-v0.0.3_linux.zip" target="_blank"><i class="fa fa-cloud-download fa-2x"></i></a>  |
|:-------------------------------------:|:-----------------------------------------------------------------------------------------------------------------------------------------:|
| <i class="fa fa-windows fa-2x"></i> | <a href="https://github.com/tasdikrahman/spaceShooter/releases/download/v0.0.3/spaceShooter-v0.0.3_windows.zip" target="_blank"><i class="fa fa-cloud-download fa-2x"></i></a> |


<center><a href="#index"><i class="fa fa-chevron-circle-up fa-2x"></i></a></center>
<!-- [<i class="fa fa-chevron-circle-up fa-2x"></i>Back to top]() -->

***

### <a name="foodoh"/>[Foodoh](https://github.com/foodoh)

<center><a href="https://github.com/tasdikrahman/foodoh" target="_blank"><img src="http://i.imgur.com/GwraWSw.jpg"></a></center>

Was the team leader for **Startup Weekend Chennai, 2015** where we made a Food recommendation system.

<i class="fa fa-code fa-2x"></i>  `python`, `beautifulsoup4`, `Image Processing`, `tesseract-OCR`, `Mongo-DB`, `HMTL`, `CSS`, `Bootstrap`, `Javascript`, `PIL`

**Startup weekend Team page**: [https://www.f6s.com/foodoh](https://www.f6s.com/foodoh)

<center><a href="#index"><i class="fa fa-chevron-circle-up fa-2x"></i></a></center>
<!-- [<i class="fa fa-chevron-circle-up fa-2x"></i>Back to top]() -->

***

### <a name="tnote"/>[tnote](https://github.com/tasdikrahman/tnote)


<iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=tnote&type=star&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=tnote&type=watch&count=true&v=2" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=tnote&type=fork&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe>


<!-- <center><a href="https://asciinema.org/a/35378"><img src="https://asciinema.org/a/35378.png"/></a></center> -->
<center><script type="text/javascript" src="https://asciinema.org/a/35557.js" id="asciicast-35557" async></script></center>


A cross platform, command line note taking app built using `python`. **peewee** was used as the **ORM** choice.

- **Secure**: Encrypts your database using standard `AES-256 in CBC mode`.
- Title, content(duh!), timestamp and tags support.
- Delete/add notes tags.
- Supports full text search for notes using content as well as tags as search parameter.
- Query highlighting when a search result is found

<i class="fa fa-code fa-2x"></i>  `python`, `pycrypto`, `pysqlcipher3`, `peewee`, `sqlite3`, `clint`, `args`


As covered by

- [importpython's issue #60](http://importpython.com/newsletter/no/60/)
- [pythonweekly issue #229](http://us2.campaign-archive1.com/?u=e2e180baf855ac797ef407fc7&id=c78bf6d519)


<center><a href="#index"><i class="fa fa-chevron-circle-up fa-2x"></i></a></center>
<!-- [<i class="fa fa-chevron-circle-up fa-2x"></i>Back to top]() -->

***

### <a name="markovipy"/>[markovipy](https://github.com/tasdikrahman/markovipy)

<iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=markovipy&type=star&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=markovipy&type=watch&count=true&v=2" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=markovipy&type=fork&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe>

<center><a href="https://github.com/tasdikrahman/markovipy"><img src="/content/images/2017/05/markovipy.png"></a></center>

Simply put, a markov chains based sentence generator

She tries striking conversations with you with her cohesive sentences after you have given her fill of text to her. And no she won’t complain about how big your corpus is. Also, don’t ask her if she can pass the turing test. She might not talk to you again.

The results are often just nonsense, but at times can be strangely poetic - the sentences below were generated from the text of “The Hamlet” by Shakespeare:

> If his occulted guilt, Do not it selfe vnkennell in one speech, It is most retrograde to our desire And we beseech you, bend you to remaine Heere in the cheere and comfort of our eye, Our cheefest Courtier Cosin, and our whole Kingdome To be contracted in one brow of woe Yet so farre hath Discretion fought with Nature, That we with wisest sorrow thinke on him, Together with remembrance of our selues.

- [Blog post going through it's making](http://www.tasdikrahman.com/2017/05/06/Making-of-trumporate-using-markovipy-generating-sentences-using-markov-chains-part-1/)

<i class="fa fa-code fa-2x"></i>  `python`, `markov chains`

<center><a href="#index"><i class="fa fa-chevron-circle-up fa-2x"></i></a></center>
<!-- [<i class="fa fa-chevron-circle-up fa-2x"></i>Back to top]() -->

***

### <a name="easyrbac"/>[easyrbac](https://github.com/tasdikrahman/easyrbac)

<iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=easyrbac&type=star&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=easyrbac&type=watch&count=true&v=2" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=easyrbac&type=fork&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe>

[![Demo](http://www.tasdikrahman.com/content/images/2017/05/easyrbac.png)](https://github.com/tasdikrahman/easyrbac)

> [Blog post](http://www.tasdikrahman.com/2017/06/01/Implementing-role-based-access-Control-easyrbac-python/) explaining how it works

Role based access control (RBAC0) implementation

As covered by

- [importpython's issue #127](http://importpython.com/newsletter/no/127/)

<i class="fa fa-code fa-2x"></i>  `python`

<center><a href="#index"><i class="fa fa-chevron-circle-up fa-2x"></i></a></center>
<!-- [<i class="fa fa-chevron-circle-up fa-2x"></i>Back to top]() -->

***

### <a name="thanos"/>[thanos](https://github.com/tasdikrahman/thanos)

<iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=thanos&type=star&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=thanos&type=watch&count=true&v=2" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=thanos&type=fork&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe>

[![Demo](http://i.imgur.com/qlzSCuP.jpg)](https://github.com/tasdikrahman/thanos)

A GUI demonstration of SQL injection and prevention techniques on a local `SQLite` database following
the guidelines layed out by OWASP.

<i class="fa fa-code fa-2x"></i>  `python`, `tkinter`, `sqlite3`

<center><a href="#index"><i class="fa fa-chevron-circle-up fa-2x"></i></a></center>
<!-- [<i class="fa fa-chevron-circle-up fa-2x"></i>Back to top]() -->

***

### <a name="pyzipcode"/>[pyzipcode](https://github.com/tasdikrahman/pyzipcode-cli)


<iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=pyzipcode-cli&type=star&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=pyzipcode-cli&type=watch&count=true&v=2" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=pyzipcode-cli&type=fork&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe>


[![Demo](https://raw.githubusercontent.com/tasdikrahman/pyzipcode-cli/master/assets/pyzip_demo.gif)](https://github.com/tasdikrahman/pyzipcode-cli)


`Python` module to extract every possible meta data from a Zip Code. Meta data like what?

- `latitude` and `longitude`
- `city`, `county`, `state`
- boundaries (in latitude and longitude)

<i class="fa fa-code fa-2x"></i>  `python`, `requests`

<center><a href="#index"><i class="fa fa-chevron-circle-up fa-2x"></i></a></center>
<!-- [<i class="fa fa-chevron-circle-up fa-2x"></i>Back to top]() -->

***

### <a name="xkcddl"/>[xkcd-dl](../xkcd_dl)


<iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=xkcd-dl&type=star&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=xkcd-dl&type=watch&count=true&v=2" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=xkcd-dl&type=fork&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe>


[![Usage](https://raw.githubusercontent.com/tasdikrahman/xkcd-dl/master/assets/usage.gif)](../xkcd_dl)

A command line application to **download all xkcd's** which have been uploaded till date. Ever!


<i class="fa fa-code fa-2x"></i>  `python`, `requests`, `beautifulsoup4`, `docopt`, `python-magic`

As covered by

- [lamiradadelreplicante.com on their blog post named "Download the geeks strips from the terminal xkcd xkcd-d"](http://lamiradadelreplicante.com/2015/11/15/descarga-los-tiras-geeks-de-xkcd-desde-la-terminal-con-xkcd-dl/) a spanish Linux blog.

<center><a href="#index"><i class="fa fa-chevron-circle-up fa-2x"></i></a></center>
<!-- [<i class="fa fa-chevron-circle-up fa-2x"></i>Back to top](#index) -->

***

### <a name="pycalc"/>[pyCalc](https://github.com/tasdikrahman/pyCalc)


<iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=pyCalc&type=star&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=pyCalc&type=watch&count=true&v=2" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=pyCalc&type=fork&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe>


<center><a href="https://github.com/tasdikrahman/pyCalc"><img src="https://raw.githubusercontent.com/tasdikrahman/pyCalc/master/assets/pyCalc_usage.gif"></a></center>

A cross platform, GUI calculator.

The making of this project was covered in a [blog post](http://www.tasdikrahman.com/2015/11/06/Building-a-calculator/) of mine if you are curious.

<i class="fa fa-code fa-2x"></i>  `python`, `tkinter`

<center><a href="#index"><i class="fa fa-chevron-circle-up fa-2x"></i></a></center>

***

### <a name="margo"/>[margo](https://github.com/tasdikrahman/margo)


<iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=margo&type=star&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=margo&type=watch&count=true&v=2" frameborder="0" scrolling="0" width="200px" height="20px"></iframe> <iframe src="https://ghbtns.com/github-btn.html?user=tasdikrahman&repo=margo&type=fork&count=true" frameborder="0" scrolling="0" width="200px" height="20px"></iframe>


<center><a href="https://github.com/tasdikrahman/margo"><img src="https://raw.githubusercontent.com/tasdikrahman/margo/master/assets/demo.gif"></a></center>

An opiniated slack bot for [SRMSE's](http://srmsearchengine.in/) slack channel

Related blog [post of mine](http://www.tasdikrahman.com/2016/06/25/Margo-An-opiniated-Python-based-slack-bot-for-SRM-Search-Engine/)

<i class="fa fa-code fa-2x"></i>  `python`, `slackclient`, `requests`

<center><a href="#index"><i class="fa fa-chevron-circle-up fa-2x"></i></a></center>


If you have something in your mind, you can reach me at **hey AT tasdikrahman DOT me**

You can find me on twitter at [@tasdikrahman](http://twitter.com/tasdikrahman)
