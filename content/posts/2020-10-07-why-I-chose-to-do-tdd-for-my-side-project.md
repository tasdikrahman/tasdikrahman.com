---
title: "Why I chose to do TDD for my new side project"
description: "Why I chose to do TDD for my new side project"
tags: [testing, tdd]
comments: true
share: true
cover_image: '/content/images/2019/03/painter.jpeg'
---

This post is more of a continuation to this tweet

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">One thing which I tried doing differently this time with one of my side projects is to do TDD from the start. Someone may ask why? It&#39;s just a side project no? (1/n)</p>&mdash; Tasdik Rahman (@tasdikrahman) <a href="https://twitter.com/tasdikrahman/status/1312622418230296576?ref_src=twsrc%5Etfw">October 4, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

I have been building [bhola](https://github.com/tasdikrahman/bhola) in my free time, and one thing which I tried doing differently this time with it, was to practice [TDD](https://en.wikipedia.org/wiki/Test-driven_development) from the start.

## But why?

Someone may ask why? It's just a side project no? True, yes. It is, but let me explain why I tried this out.

One reason is that, for some of my past side projects, when someone creates an issue/submits a PR. I wouldn't necessarily remember everything which I did/why I did x instead of y, when I would have authored it (more on how this can be improved later)

Taking the liberty to quote [Ajey](https://twitter.com/AjeyGore).

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">What ever code you write, it will be out of context in 18 months, write tests along with it, so at least people know what you meant</p>&mdash; Ajey Gore (@AjeyGore) <a href="https://twitter.com/AjeyGore/status/865555853423673344?ref_src=twsrc%5Etfw">May 19, 2017</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Coming back to say reviewing a bugfix/feature PR. Having no coverage for those specific routines which were modified, would mean I either would have to rely on my gut feeling, or I would have to test it by pulling the changes.

This in turn would do two things, for one, it would create a form of resistance, as to even review the PR, it would mean me having to also manually test out things and see if changes are not having any regression/the feature works as expected. Which would mean, I would either get swamped by the things to do to just review something, making the requests pile up one by one and then ending up in a position where there are multiple stale PR's which have been just lying there. (If you have ever experienced this with any of my repositories, I sincerely apologise, I will strive to be better.)

The 2nd thing which would be a by-product of this, is that for these changes, I am doing the testing manually, which would mean I would have spent say x amount of time doing it which could have been used for something else.

This x amount of time, would vary wildly, depending on many factors. Some can be, how good are the docs, which would allow 1 to replicate the setup quickly(another reason why I really love 1 step dev setup commands)? Familiarity with the codebase so as to remember all the cases, corner cases included, so that you don't miss them.

It's natural for someone to not remember minute details of the codebase, when they are looking at it again after weeks/months/years. Naturally, they will need some time to again get acclimatized to the codebase which they had interacted/authored.

This is where tests for the routines bring in value. It's your 1st level of safety net which you have spread out to weed out changes which would break your expected flow/behaviour.

## Will this solve all my problems?

As luck will have it, I have an example from the side project which I was working on itself, where the coverage was high and covered the specific flow, but would ultimately fail when trying to run it!

There's one specific flow, where the service reads an env var from the environment variable via [Figaro](https://github.com/laserlemon/figaro). The value is a plain boolean var of `true`. Now to simulate this in the spec, what I did was simply stub the call to the method of the figaro lib, to return the value I wanted for the flow. The problem being here, that I was stubbing the wrong type for the value! Figaro, when it reads the env var, it reads it as a string rather than a boolean(or any other type for that matter, all will be read as a plain string), which is where I was going wrong. This in turn would also affect the way, the implementation would happen.

Here's a small snippet from the changelog of [https://github.com/tasdikrahman/bhola/pull/65](https://github.com/tasdikrahman/bhola/pull/65) for reference, to give you an idea of what I am trying to depict here and what I changed to fix the same in the spec as well as the implementation.

```patch
diff --git a/app/jobs/check_certificate_job.rb b/app/jobs/check_certificate_job.rb
index 93fb16b..76f995b 100644
+++ b/app/jobs/check_certificate_job.rb
--- a/app/jobs/check_certificate_job.rb
@@ -10,7 +10,7 @@ class CheckCertificateJob < ApplicationJob
     Domain.all.each do |domain|
       if domain.certificate_expiring?
         Rails.logger.info("#{domain.fqdn} is expiring within the buffer period")
+        if (Figaro.env.send_expiry_notifications_to_slack == 'true') && !Figaro.env.slack_webhook_url.empty?
-        if (Figaro.env.send_expiry_notifications_to_slack == true) && !Figaro.env.slack_webhook_url.empty?
           message = "Your #{domain.fqdn} is expiring at #{domain.certificate_expiring_not_before}, please renew your cert"
           slack_notifier = SlackNotifier.new(Figaro.env.slack_webhook_url)
           begin
diff --git a/spec/jobs/check_certificate_job_spec.rb b/spec/jobs/check_certificate_job_spec.rb
index f4fa49b..71c40c8 100644
+++ b/spec/jobs/check_certificate_job_spec.rb
--- a/spec/jobs/check_certificate_job_spec.rb
@@ -48,7 +48,7 @@ RSpec.describe CheckCertificateJob, type: :job do

           it 'will not call SlackNotifier#notify' do
             allow_any_instance_of(Domain).to receive(:certificate_expiring?).and_return(true)
+            allow(Figaro).to receive_message_chain(:env, :send_expiry_notifications_to_slack).and_return('false')
-            allow(Figaro).to receive_message_chain(:env, :send_expiry_notifications_to_slack).and_return(false)
             allow(Figaro).to receive_message_chain(:env, :slack_webhook_url).and_return(slack_webhook_url)
             expect_any_instance_of(SlackNotifier).not_to receive(:notify).with(anything)
```

So as you see, it's not necessary that following the above practices, will allow you to create bug free software.

Bhola had [~99.63%](https://github.com/tasdikrahman/bhola/pull/65/checks?check_run_id=1203022679) coverage at the time this bug was present in it, but it didn't stop it from having this bug.

100% code coverage doesn't mean that your software is bug free/free of issues. The only real test is when your software is getting used by someone. This is where it should behave/perform as it is expected out of it. There's [no silver bullet](https://en.wikipedia.org/wiki/No_Silver_Bullet).

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Being proud of 100% test coverage is like being proud of reading every word in the newspaper. Some are more important than others.</p>&mdash; Kent Beck (@KentBeck) <a href="https://twitter.com/KentBeck/status/812703192437981184?ref_src=twsrc%5Etfw">December 24, 2016</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## So what's the use then?

But having a high coverage would also mean, that you can refactor without fear, and have a faster feedback cycle than before, i.e testing for changes manually.

The 2nd level of safety net can be end-to-end integration tests for your codebase, which would run with each commit, the same way your unit tests would run with each commit.

The value here out of these 2 safety nets, is that you will be able to ship with more confidence, compared to not having these 2 safety nets at all

## Why I chose to do TDD here?

There's a lot of literature around this, but for me personally I feel it allows me to think in terms of contract and how a routine should behave. As the behaviour is what we would really like to test for routine rather than the exact mechanics.

To add to it, the tests would act as documentation when I would go through them, telling me how a particular routine behaves under different scenarios. It also encourages baby steps and a faster feedback loop for something functional as fast as possible.

For reference, a few years ago, I wrote this thing called [plino](https://github.com/tasdikrahman/plino)(spam filtering as an API) back in college days. I wasn't aware of the testing literature back then (still learning), but what I ended up writing was an [integration test](https://github.com/tasdikrahman/plino/blob/master/tests/test_plino_app_api_response.py
) for the api.

It has absolutely no coverage for other routines which are present. It's just by luck, that the codebase is small and someone will be able to quickly grok it and understand what is happening, but the overload of the same happening in larger codebase does affect maintenance.

If I have to compare it with bhola, I ended up having [coverage for even a small routine which just does a POST to an external API](https://github.com/tasdikrahman/bhola/blob/master/spec/services/slack_notifier_spec.rb). Someone might think it's an overkill, why do we need all this if barely anyone is using this?

Another question which comes is, at the end it would be the functioning lines of code which your consumer of the software would be interacting with, not the tests. So why write tests? But would skipping these mean, taking a hit on maintainability, I feel the answer is yes.

As for [bhola](https://github.com/tasdikrahman/bhola), I feel I would definitely have more confidence and a faster feedback cycle when adding changes to it in future.

If you liked this piece, I have written a few more under [#testing](https://www.tasdikrahman.com/blog/tag/testing) and [#tdd](https://www.tasdikrahman.com/blog/tag/tdd).

