---
title: "My internship experience at Wingify (VWO team), New Delhi"
description: "My internship experience at Wingify (VWO team), New Delhi"
tags: [internships]
comments: true
share: true
cover_image: '/content/images/2016/08/Wingify_acquisition_main.jpg'
---

As I am sitting here at the Delhi Airport waiting for my flight back to Chennai, I could just not stop myself from thinking about my time as an intern back at Wingify which ended last week. Here’s what I wrote down after getting carried way with several cups of coffee (thanks for luring me with that smell costa coffee)

So here it is then! 

## Day 1

It was 5 o'clock in the morning and I was quite drowsy. Reason being the all nighter I pulled the other night for the last exam of our end sems. **phew**

Here I was at the Delhi airport just a day after my semester exams, ready to start with my internship. Talk about eagerness here!

A small part of me was also happy that I was moving out of Chennai! (at least for some time)

Joined them the next day in their main office at the heart of NSP, Delhi.

<center><img src="/content/images/2016/08/wingify_rig_mine.jpg"></center> 

Now I was naturally excited to work in a company which had grown and become one of the best startups in India in such a short span of time. On top of that, this was my first internship in a well-established product based start-up and I was hoping that I could learn all that I could and perform in accordance to their standards.

<center><img src="/content/images/2016/08/wingify_officespace_2.jpg"></center> 

I was introduced to Ankit Jain (Lead software Engineer at Wingify) and Ajay Sharma (Senior Software Engineer) by my HR. We had a brief chat where I was told I would be working with the Backend Development team for VWO, their flagship product.

<center><img src="/content/images/2016/08/wingify_officespace_1.jpg"></center> 

Talking about VWO, it's the world's easiest A/B testing tool. And we are quite good (read "The Best") at it! The month before I joined, we had monthly sales crossing a little over 1 million dollars.

After getting up and ready with my development environment, I was given my first project.

### Integration of Statsd and graphite (Project #1)

StatsD collects and aggregates metrics and then ships them off to Graphite which stores the time-series data and enables us to render graphs based on these data.

Graphite consists of three parts.

- carbon: a daemon that listens for time-series data.

- whisper: a simple database library for storing time-series data.

- webapp: a (Django) webapp that renders graphs on demand.

The setting up of the the overall stack was a bit archaic but I finally got it right and the metrics for our internal service were being graphed correctly by Graphite. And they looked pretty too!

Coming back, the service on which we integrated StatsD and graphite runs on several servers. So while plotting the graphs we wanted to know the server from where the stats are being pushed on to the buckets of statsd. Well that was much about it.

### Bumblebee - An experimental slack bot VWO (Project #2)

Wingify has this culture of organizing hackathons at the end of every month, where people from the engineering team come together to hack on something which they want to see at VWO.

To be honest, I was quite clueless on what to build for the first half an hour or so and after a little nudge from Ankit I decided upon bumblebee. Bumblebee makes use of the beautiful VWO API to provide functionalities (if not all) to the VWO account holder right at the comfort of his slack channel. Like you can get details of all the campaigns of your account, check their status (whether they are running, paused et el). Update status to Start/Stop/Pause a particular campaign. Share your campaign with someone else and some more things.

It was written in python and Ankit was too kind to let me open source it. Here is the link for the curious.

[https://github.com/wingify/bumblebee](https://github.com/wingify/bumblebee)

### Optimization much? (Project #3)

My 3rd project revolved around optimization of an internal service. I had to increase the efficiency (read performance). I implemented some rough 3 approaches and the last one bumped the performance by up to 23.6%. I could have tried for dropping it down further to a lower one but sadly the end to my internship was looming around the corner so I dropped it. And that was the 3rd and the last project I did as an intern at Wingify.

## How was my experience?

My experience? I loved it there!

- Solving hard engineering problems. Check

- Extremely talented engineering team. Check

- Approachable mentors. Check

Heck, here I am with Abhishek Batra, the Android guru here and Abhishek Garg, our DevOps pro. Turns out that we all came along quite well along each other along with the other people and yes these guys are quite senior to me :)

<center><img src="/content/images/2016/08/wingify_tasdik_abhishek.jpg"></center> 

- Awesome Work Culture. Check

- Delhi :P. Check

Jokes apart. I made some really good friends back there and learned a ton from everyone. I am proud that I was part of a team which is building something which people love and has an impact on thousands of customers.

## So what now?

Looking back at the time when I received a call from Nupur about my acceptance as an intern at Wingify. I was thinking about whether to join it over the other 6-7 odd companies which accepted my application as a summer intern. 

<center><img src="/content/images/2016/08/wingify_table_tennis.jpg"></center> 

After the two months that I have spent here at Wingify, I now believe that I did just the right thing on choosing Wingify over others!

And did I mention that they gave me a pre-placement offer :) ?

This was taken on my last day at office. And boy was I sad!

<center><img src="/content/images/2016/08/wingify_newdelhi_tasdik.jpg"></center> 

Ankit threw a huge party at a pub in Rajouri garden for all the interns along with the other guys and my last day turned out to be the best day in Delhi.

<center><img src="/content/images/2016/08/wingify_2016_summer_interns.jpg"></center> 

Until next time Delhi!

_This post was cross posted at [Wingify's team blog too](http://team.wingify.com/tasdik-talks-about-his-internship-experience-at-wingify)_