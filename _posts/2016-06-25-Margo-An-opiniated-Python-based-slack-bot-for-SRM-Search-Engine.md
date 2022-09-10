---
layout: post
title: "Margo: An opiniated Slack Bot for SRMSE's Slack channel"
description: "Margo:  Slack bot to tell whether a site is up or not"
tags: [oss, chatbot, python]
comments: true
share: true
cover_image: '/content/images/2016/6/cover.png'
---

## Bots: Before and Now

When was the last time you were having a conversation with a computer before? Most of us will come into the category where we haven't done so or maybe you did but you didn't like the experience! The days aren't far when customer support will be provided by dedicated bots built using cutting edge ML techniques and backed by state of the art NLP research.

If you have been following the latest trends in software industry. Bots and VR are *the thing* as echoed by a lot of big shot companies and tech evangelists. I mean just look at [these](https://techcrunch.com/2016/04/07/rise-of-the-bots-x-ai-raises-23m-more-for-amy-a-bot-that-arranges-appointments/) [crazy](https://techcrunch.com/2016/03/17/facebooks-messenger-in-a-bot-store/) [articles](https://techcrunch.com/2016/05/11/kik-already-has-over-6000-bots-reaching-300-million-registered-users/) [on](https://techcrunch.com/2016/05/10/facebook-chatbot-analytics/) [techcrunch](https://techcrunch.com/2016/05/07/bots-messenger-and-the-future-of-customer-service/). Bots have been the talk of the town since some time now and developers are taking a fair advantage of this rise in attention for promoting their own bots in the market.

## What got me into bots

Well to be honest. I am quite a regular to [techcrunch's](https://techcrunch.com/) website. And when I got to know that they had released a [telegram](https://telegram.org/) bot for their service, it didn't take me too long to setup the bot for my telegram ID.

I was quite cynical at first about how the experience would turn out. But surprisingly! The interface was really intuitive and didn't make any assumptions on their side about what the user knew or didn't.

The bot helps you stay on top of the topics, stories and people you care about the most. You can subscribe to different topics, authors or sections of the site, and the bot will send you news articles on a daily basis or when you explicitly ask for it.

In short I was really impressed and didn't regret installing the bot.

![techcrunch](https://tctechcrunch2011.files.wordpress.com/2016/03/mar-15-2016-1020.gif)

_image courtesy: TechCrunch_

## So what's the deal with Margo

Back at [SRM Search Engine](http://srmsearchengine.in/), we manage our own single dedicated server for providing our search service. And sometimes, it goes down due to some or the other reasons. Sometimes for the insane amount of data that we crunch on it or when we are playing around with a new technology which requires some downtime.


The idea for [Margo](https://github.com/tasdikrahman/margo) came to me while having a chat with one of my college mates on our slack channel. Oh no, we were not talking about bots! Something totally different, but well it came out of the blue to me!

Weekend ahead! Weather in Delhi (ok. it was raining that day!), along with some aloo tikkas. What a bliss!

Reading the docs for the [Slack API](https://api.slack.com/) simultaneously and working on my bot. I had a working prototype ready in about 2 hours. The next 1 hour was spent refactoring the app.

Here's a glimpse to what it does

![Margo demo](https://raw.githubusercontent.com/tasdikrahman/margo/master/assets/demo.gif)

Pretty basic for now. I plan to automate the pinging process. But the current deployment of the bot forbades me on doing so. You see, I have deployed it to a basic dyno on [heroku](https://heroku.com/). The thing is that, the dyno goes to sleep if it does not recieve any `HTTP requests` afer some time. Moreover, they have a fixed 6 hour downtime for any basic dyno. So yeah, as I am pretty much broke right now. I cannot afford a Digital Ocean/Linode server. But I mean what the heck right, at least it works for now!

More functionalities are coming through over.

>Here's a link to the project : [Margo](https://github.com/tasdikrahman/margo)

All in all, I enjoyed building [Margo](https://github.com/margo/) for this was a side project after some weeks (several if you may) of break. Reason being, I hadn't had much time to indulge in side projects due to my commitments as an intern at [Wingify](http://wingify.com/). And man, I am loving it here!

***

Back there, I just finished working on an internal backend service which is a [rabbitMQ](https://www.rabbitmq.com/) consumer handling the consumed messages (based on type of the queue) from the numerous queues and processing them accordingly. Did some refactoring of the service and then integrated [statsd](https://statsd.readthedocs.io/en/v3.2.1/index.html) with it to graph the IO operations done by it. [Graphite](https://graphite.readthedocs.io) along with it's components [carbon](https://graphite.readthedocs.io/en/latest/carbon-daemons.html) and [whisper](https://graphite.readthedocs.io/en/latest/whisper.html) were used to visualize the data in a human readable graph format. Last step was to deploy the setup to a test server on Digital Ocean.

Maybe what I just wrote is quite abstract. But I plan on writing a blog post about my experience with, but let's see when I get time to write about it!

***

So this weekend, me and some of my friends header over to [Bot builder workshop meetup, Delhi](http://meetup.com/Bot-Builder-Delhi/) for the fun of it. We had Beerud Sheth, the CEO of Gupshup give a talk about how bots were the next big thing.

All in all we had a really good time and met some really interesting guys.

![Gupshup](https://raw.githubusercontent.com/tasdikrahman/www.tasdikrahman.com/gh-pages/content/images/2016/6/gupshup.jpg)

![innov8](https://raw.githubusercontent.com/tasdikrahman/www.tasdikrahman.com/gh-pages/content/images/2016/6/innova8.jpg)

Cheers!
