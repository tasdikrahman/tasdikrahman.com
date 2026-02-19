---
title: "Monoliths are just fine"
description: "Monoliths are just fine"
tags: [systemdesign]
comments: true
share: true
cover_image: '/content/images/2018/10/monolithic-vs-microservices.png'
---

A lot of great material has already been [written](https://martinfowler.com/articles/microservices.html) out there around what microservices are and what they are not.

What I would try putting down here is what I saw as we grew from a monolith to a microservices architecture over the period of time back here at [Razorpay](https://www.razorpay.com). Please take it with a grain of salt when you read this as this is going to be opinionated.

> Microservices are something which you grow into, not something you start with

Having a microservice(s) based architecture just for the sake of having one is kind of like using [kubernetes](https://kubernetes.io) to deploy a service or two on it. In the end, you will only overcomplicate things by having multiple things to manage. And trust me, [kubernetes is hard](https://codeengineered.com/blog/2017/kubernetes-is-hard/). I say that with at least a year and a half of running production workloads at Razorpay on self-hosted kubernetes and having scaled them to what it is now. It's complicated and can be hard to get right at first. But makes life much easier when you have multiple services to manage at the very least.

If you have a look at Shopify, they are one huge Ruby on Rails shop. And the last time I checked their scale is pretty huge. So the argument that microservices are the only way to scale is not entirely true.

There are reasons why I feel monoliths are the way one should start with, because

>  Microservices are complicated to manage

If you're just starting to write your app, you need to get market validation fast and get the features out before your competition does, instead of having to worry about 10 other different things. Your customers/investors probably won't care about how your service works or what is their underlying tooling here unless they are techies and would be curious about how is it working internally but at the end of the day, they just want things to work!

This is the part, where you are working on your prototype and you don't need the extra overhead of worrying about how the 10 different microservices you wrote are interacting with each other. As at the end of the day, the tooling and the architecture matters, but not if you have not even attained that critical mass. This is a little similar to fight over what language to choose over the other, of course, there would be obvious choices for some specific jobs but you can always go first with the one which you and the rest of the team are most comfortable with to [beat the averages](http://www.paulgraham.com/avg.html)

Plus the overhead of managing the health checks, monitoring and graceful shutdown for all the microservices is something which you need to put on your head around when starting with microservices apart from writing your main business logic.

They bring a lot of baggage that you might never have seen before (i.e. big learning curve) and the myth of isolated changes is just that, a myth. Unless it is some low-level thing, you cannot change it without impacting other services and this is no different than a monolith. It just replaces internal calls between services of your monolith with flaky and slower network calls.


> Architect your monolith right

When you start off with your monolith, there would be places where you feel you see things repeating. And that's where you separate that out into functions or modules. But you should not be afraid to repeat yourself at the start and try to modularise/abstract out everything, as you would not like to end up with [leaky abstractions](https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/)

When you understand the boundaries and functions well enough, that's when you would start modularizing your codebase. So as to have clear distinctions of which part of the codebase does what. This helps in a few things, specific teams being able to work on specific codebases and clear distinction between core functionality and helper functions.

When you are done dividing your codebase into modules, using a message queue for asynchronous communication between the modules would make more sense. This will enable you to debug faster, distribute the work of different components to different teams for things to start with.

With all the above in check, you already have a lot of things sorted which would not become a technical debt when trying to move to microservices.

From what I have noticed is, over-optimising from the start is just gonna bog you down. The priority at the start should be to
- Get it working.
- Get feedback about your product.
- Fix the bugs when they are reported or when you find any.
- If things go down, look why they went wrong and fix them. This is where you will learn from your experience.
- Repeat the whole process.

I think of it this way, your product is not a program like [ls](http://man7.org/linux/man-pages/man1/ls.1.html) command which is feature complete, you need to constantly iterate upon it, but even before that. You have to give out a working model for people out there to use it.

> Moving to a microservice?

The argument of moving to a microservice can be made when
- you have divided certain work among certain teams, by virtue of which there would be times when there would be friction, miscommunication happening over when contributing to certain parts of the codebase which come common when different teams are working. Even if you have managed to write something, it's quite possible that something which you added might have had a regression over something, and at this point, your automated tests in the CI should ideally catch them. But having a clear separation of work when you have multiple teams working on things, microservices can make sense.
- you would want to rewrite a piece of codebase into something more performant.
- Feature Velocity
- autonomy of teams to iterate faster with their own choice of technology
- make deployments quicker, impact on story structure.

The jump from monolith to service-oriented thinking is a huge one. But the jump from a few services to more is much easier.

Most of the times, I've found a push to microservices within an organization to be due to some combination of:

1) Business is pressuring tech teams to deliver faster, and they cannot, so they blame current system (derogatory name: monolith) and present microservices as solution. Note, this is the same tired argument from years ago when people would refer to legacy systems/legacy code as the reason for not being able to deliver.

2) Inexperienced developers proposing microservices because they think it sounds much more fun than working on the system as it is currently designed.

3) Technical people trying to avoid addressing the lack of communication and leadership in the organization by implementing technical solutions. This is common in the case where tech teams end up trying to "do microservices" as a way to reduce merge conflicts or other such difficulties that are ultimately a problem of human interaction and lack of leadership. Technology does not solve these problems.

4) Inexperienced developers not understanding the immense costs of coordination/operations/administration that come along with a microservices architecture.

5) Some people read about microservices on the engineering blog of one of the major tech companies, and those people are unaware that such blogs are a recruiting tool of said company. Many (most?) of those posts are specifically designed to be interesting and present the company as doing groundbreaking stuff in order to increase inbound applicant demand and fill seats. Those posts should not be construed as architectural advice or *best practices*

In the end, it's absolutely the case that a movement to microservices is something that should be evolutionary, and in direct need to technical requirements. For nearly every company out there, a horizontally-scaled monolith will be much simpler to maintain and extend than some web of services, each of which can be horizontally scaled on their own.

IMHO, the monoliths vs microservices debate is akin to monorepos vs multi-repos: they are both strategies used to share work when your organization grows. Both can work well, depending on your tooling and organization.

But do not forget that those abstractions layers you add, while very useful (say, for release velocity), might also be a direct application of [Conway's Law](https://en.wikipedia.org/wiki/Conway%27s_law)

Which means that refactoring some code, might sometime require refactoring your organisation, so if you lack the ability to do that incrementally, you might converge to an ossified system that stops evolving.

> So, Microservices or Monoliths?

It depends.

## References

- https://martinfowler.com/bliki/MonolithFirst.html
- https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/
- http://www.paulgraham.com/avg.html

