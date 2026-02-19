---
title: "Bug which would cause some deployments to get triggered again and again"
description: "Bug which would cause some deployments to get triggered again and again"
tags: ["softwaredevelopment"]
comments: true
share: true
cover_image: '/content/images/2021/06/cover-image.jpg'
---

This post is a continuation of this tweet here.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">We recently encountered a bug in our deployment flow, which we were completely oblivious to. (1/n)</p>&mdash; Tasdik Rahman (@tasdikrahman) <a href="https://twitter.com/tasdikrahman/status/1401225434835001345?ref_src=twsrc%5Etfw">June 5, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Bugs are present in every system, waiting to be discovered. As such, this one was no different.

## What did the bug do?

Would cause an application to be deployed again and again, even though when it had been triggered only once to be deployed.

## Context about the system involved

The part which handles requests to deploy an application to a set of VM's is a custom service which we have written and maintained over the years. The exact workings of it is the subject for another discussion. But gist is that, it has an API which listens to requests and does the intended work of deploying a change set to a set of VM's for the application and would return back an appropriate response.

Depending on whether the deployment was successful or not, the response body gets added with further information for the same for the client here to decipher whether the deployment was a success or not.

The deployment service runs behind gunicorn with a couple of sync workers handling the incoming requests, only other thing being that the workers are configured with a [timeout configuration](https://github.com/benoitc/gunicorn/issues/1493#issuecomment-321331753), which acts as request timeout configuration.

The current setup being sync, there is a timeout configuration also present on the reverse proxy which sits between the client and the service which deploys an application to VM's. The proxy timeout configuration was set to the same value as the gunicorn worker timeout.

Orchestration for the whole deployment, right from sending the details of the deployment request (which app to deploy, which environment along with other metadata), is handled by a separate service, which also doubles up as the internal service registry (more on this later)

The deploy request which this broker service receives, eventually ends up being queued as a background job. One detail here being that the job processing framework used here, retries failures with an exponential backoff by default.

Which would mean that the retry will keep on happening for a couple of times, which ends up being a not so good number for the nature of the job beind handled, which is not an intended behavior for this flow. After the max no's retries the job would be pushed to the deadset

## Immediate fix

After killing this specific job manually, to immediately resolve the issue, we went a bit further into checking the cause because of which the job processing logic for the deployment job was causing an error, leading to the retry.

## What caused it

Now the way the service handling deployments works is such that, out of a set of application VM's, where it has to deploy, it will do it either 1 at a time or the level of concurrency being passed in the deployment request.

More on the workings of the series of steps taken while deploying this changeset later. The point to note here being that the deployment time increases as a factor of the number of VM's present for the application, in the particular environment where it is being deployed.

What ended up happening was, the orchestrator would wait for the response from the deployment service, after x minutes set on the gunicorn worker, set same as the reverse proxy connection timeout too, the request would be abruptly closed by the proxy if it exceeded x mins.

All of this, even before a valid response could be received by the orchestrator from the deployment service.

The block handling the response from the deployment service, was having a guard to handle an HTTP error while making the request, but it was after the block where we would check for the status of a deployment after the response was received.

Given the timeout configuration, the HTTP error flow not being handled before the parsing the deployment response flow, it would end up error-ing out the deployment job, leading it to be retried by the background job processor.

## What did we do to prevent this from happening again

The first fix added was to increase the response timeout in the reverse proxy to be comfortably more than the timeout set on the gunicorn timeout, so that if the worker times out, the response would be sent through, without the connection getting timed out by the proxy.

The second thing which we ended up doing was to keep a check on the deployment flow, before initiating the deployment on whether the deployment is already in a terminal state (failed/succeeded) which was easy to check given the state transitions being maintained.

We could have also made use of the background job processor's `max_retry` setting in this case by setting it to 0, which would have not retried the job at all if it failed once. Another option would have been to use `discard_on` here [https://edgeapi.rubyonrails.org/classes/ActiveJob/Exceptions/ClassMethods.html#method-i-discard_on](https://edgeapi.rubyonrails.org/classes/ActiveJob/Exceptions/ClassMethods.html#method-i-discard_on)

Although the root cause is not this, but an obligatory plug, of the fallacy of ["Network is reliable"](https://web.archive.org/web/20171107014323/http://blog.fogcreek.com/eight-fallacies-of-distributed-computing-tech-talk/)

## Links

- [https://github.com/benoitc/gunicorn/issues/1493#issuecomment-321331753](https://github.com/benoitc/gunicorn/issues/1493#issuecomment-321331753)
- [https://docs.gunicorn.org/en/stable/settings.html#timeout](https://docs.gunicorn.org/en/stable/settings.html#timeout)
- [Credits for the cover photo to Hitanshu](https://unsplash.com/photos/ZmFjEJq-x9k)
