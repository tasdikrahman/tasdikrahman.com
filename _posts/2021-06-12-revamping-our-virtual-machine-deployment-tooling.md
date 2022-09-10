---
layout: post
title: "Revamping Vesemir: our virtual machine deployment service"
description: "Revamping Vesemir: our virtual machine deployment service"
tags: [softwaredevelopment, deployments]
comments: true
share: true
cover_image: '/content/images/2021/06/vesemir-revamp.jpg'
---

This is a continuation of the [post](https://www.tasdikrahman.com/2021/06/10/vesemir-our-virtual-machine-deployment-service/), which details into the working of vesemir and how it goes about introducing changeset. Give it a read before continuing reading this, to allow you to gather more context on the what and the why.

While this post will focus more more on how we went on with revamping vesemir for increasing it's reliability, maintainability and modernizing it.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Continuing the thread around how we did the same for Vesemir. (1/n) <a href="https://t.co/Xol0uRraJv">https://t.co/Xol0uRraJv</a></p>&mdash; Tasdik Rahman (@tasdikrahman) <a href="https://twitter.com/tasdikrahman/status/1403746484630159360?ref_src=twsrc%5Etfw">June 12, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

### Previous state of affairs

Trite as it is, everything in software I feel is improved over iterations and the same is true with vesemir here too. We have a functional piece of software which does things as expected and has been running for quite some time now. After the initial set of features, this project remained in a bug fix state for a bit.

When our team took ownership for the deployment ecosystem a year back, we started looking for ways in which we could better handle the support requests for deployment issues and to reduce the firefighting, given we were slowly starting to gain context bit by bit. The deployments for containers didn't have much issues, given it was stable enough to not let such issues creep in.

For starters, there were a couple of boxes(VM's) in which Vesemir was deployed, which would receive requests for deploying applications. We also noticed some teams independently cloning the initial vesemir VM's, where they had added their own set of changes and then using this Vesemir VM to deploy their applications.

Centralized logging was something which we needed to add. Devs would look inside the specific VM boxes of vesemir to see what was going on, otherwise when a deployment failed (a very huge source of time sink for devs in our team/Level 1 support folks).

Monitoring was missing on the boxes, if and when CPU would spike (continuing to stay that way), we would not know about it and the deployments being processed would get slower and take more time. This would cause deployment failures, a bit of that is explained here in this [post](https://www.tasdikrahman.com/2021/06/06/bug-which-would-cause-some-deployments-to-get-triggered-again-and-again/), as the workers picking up deployment requests were having a finite time to process the deployment request.

Integration testing for the changes before deploying them to the production set of vesemir VM's was very manual and haphazard, prone to manual errors. It involved, removing one of the vesemir boxes, from behind the HAproxy and then deploying the changes to this vesemir box, testing it all along, and if everything went well, doing the same for other boxes. Again a huge time sink.

The way we deployed vesemir to the vesemir VM was very manual, someone would have to remove the box from the HAproxy manually, pull the changes to the boxes, and then restart the service, being supervised by systemd. The whole process of deploying would take around ~10-15mins for the whole set of vesemir boxes.

Before starting with anything, we would want to solve these issues for basic sanity of vesemir.

### What did we fix first

As with everything, prioritization is crucial. Breaking down the tasks into tactical and strategic fixes so as to fix what was burning.

Fixes which would immediately give us value and would help us reduce, the firefighting were prioritised first. Which would help us get the breathing space.

Fixes/features which would help us long term value over a larger segment of users, re-architecting the setup, providing better reliability, refactoring which would help us maintain the codebase better were kept for the end.

### Baseline fixes

To solve the onboarding issue of other devs in the team and to reduce the backlog of support requests received by our team for deployment failures, we ended up creating a playbook which would consist of the different failure modes for vesemir and their solutions. Which we gave to our platform support team (more on this model later), who would then look at the tickets and then solve the customer deployment failure issues. If there would be something which they wouldn't be able to help with, we would come in as the second line of defence. This helped us immensely in freeing up time to solve the initial burning issues which we were seeing with vesemir.

Our team ended up picking, adding logging support to Vesemir, which would be then be pushed to a centralized logging platform. This would allow us, the support team, as well as the developers of the product teams to look into what really caused the issue and then give us that initial bit of information about what went wrong for further debugging.

Along with it, we also consolidated the vesemir VM's and put them behind an HAproxy box, to front the service and loadbalance the incoming requests, which was missing earlier.

We also picked up adding monitoring and alerting for the vesemir VM's which would allow us visibility into what was happening. This also allowed us to see the CPU going all the way upto 100%, without any signifant increase in deployment requests being received.

[Vidit](https://twitter.com/viditganpi/) has written a [great article](https://www.gojek.io/blog/how-we-reduced-skyrocketing-cpu-usage) on solving for the same , where we ended up going from python2 to python3, which helped fix the issue altogether of the CPU spiking and staying that way, helping us reduce the number of VM's which we had been using.

We ended up replicating the production setup for an integration setup, which allowed us to have an exact replica of the production setup. The integration environment of the deployment orchestrator(also our service registry), was updated to start pointing to the integration environment of vesemir which we just created. This helped us test end to end, by sending deploy requests from the cli which we give to the developers. Allowing us to catch the errors before them landing up on production.

There were a couple of major bug fixes which we added, helping decrease the defect rate and increasing the reliability of deployments going through.

Deleting dead code, adding missing tests, a linter, integration testing scenarios, along with refactoring the codebase next, helped us maintain the codebase better and helping new devs getting onboarded to it much faster then before.

Adding support for storing deployment metadata like which application is getting deployed, the number of VM's which we are deploying, the time it's taking for the deployment, the chef tags being used to search for the application etc were also a few things which we started persisting in the vesemir database, helping us track metrics for deployments.

### Deploying vesemir by the service which is used for deploying to kubernetes

The next big chunk of work which we wanted to tackle was to automate the manual deployment process of vesemir, which would not only be time consuming for a developer in our team but would also be error prone whenever someone would be doing it.

This is where normandy came into picture. More on normandy in another post, but it's a service written on top of go-client which would effectively be used for doing CRUD operations on kubernetes resources required by an application when it is getting deployed to a kubernetes cluster(s)

The immediate benefit coming out of deploying vesemir via normandy, was that now what used to take more than 10-15mins of manually deploying vesemir to the VM's. We now were having a system in place where we would be able to deploy vesemir with the help of our deployment tooling which we give out to the product developers, with the same deployment UX, along with rollback being available via the same interface. Helping us dogfeed our own service to us.

We ended up adding support for python stack based deployments via our deployment platform as part of this effort, which came as a strategic win for us.

Another big win for us, was that we now knew the exact dependencies for vesemir, on what it used and what it didn't, which made it's infrastructure reproducible and immutable.

### Outcome

The biggest outcome of this revamp, came in terms of reducing the defect rate by ~18% from what it was earlier.

Monitoring, logging and reliability changes, along with better testing mechanisms introduced, helped reduce the hesistation by devs diving deeper into vesemir's codebase, which helped in taking it from a codebase which people feared to introduce changes into to something where the changeset could now be introduced in a couple of minutes, with immutability baked in.

A lot of toil involved in maintaining/adding features/testing to vesemir was reduced, which helped with developer productivity.

We upgraded from an EOL'd version of python, which no longer was receiving any security fixes.

### Why didn't we just rewrite vesemir?

There were a couple of things, which we considered before deciding upon not to rewrite vesemir.

#### Con's of a rewrite

- the new piece will be missing the vesemir codebase which has been
  - well tested for the behaviour by end-users.
  - in used for a couple of years.
  - patched with a lot of bug fixes which have been found and fixed over time with usage.
- not taking into consideration the dev effort which will be required to maintain two systems during such a migration to the new tool, where-in making us maitain two systems at the same time.

#### Pro's of a rewrite

- Completely new codebase, following basic sanity checks and best practices, right from the start.
- Rewrite in a language more familiar with the rest of our teams codebases(majority being in ruby/golang) rather than being the single service written in python.
- Not necessary, that the shortcomings solved in this rewrite would overshadow the new bugs which would come in as part of the rewrite.

### Takeaways

Learned a bunch during the course of the whole revamp, while creating the project plan and cutting out user stories for this, but the biggest for me was learning prioritization, deciding on what is more important of a problem to solve which would create more impact, helping the end users achieve(the product developers) achieve the end result of introducing their changeset for their service, in a reliable manner.

This was also the time when fresh grads had joined our team, mentoring them to ramp up on the codebase and the operational side of things was something which helped me deepen my own understanding around our domain.

This also was a good experience in dealing with a really hairy, legacy piece of software, working around it while enhancing it at the same time without causing issues to the customers(product team devs) and was something new.

Thanks for reading this piece. More on around normandy, our kubernetes deployment service, the other part of our deployment platform in another post.

### Links

- [https://www.gojek.io/blog/how-we-reduced-skyrocketing-cpu-usage](https://www.gojek.io/blog/how-we-reduced-skyrocketing-cpu-usage)
- [https://www.tasdikrahman.com/2021/06/10/vesemir-our-virtual-machine-deployment-service/](https://www.tasdikrahman.com/2021/06/10/vesemir-our-virtual-machine-deployment-service/)
- [Cover image credits](https://unsplash.com/photos/8lvHMctQMrU)
