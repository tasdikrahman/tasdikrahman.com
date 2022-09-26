---
layout: post
title: "Scaling cluster upgrades for kubernetes"
description: "Scaling cluster upgrades for kubernetes"
tags: [kubernetes]
comments: true
share: true
cover_image: ''
---

This post is more of a continuation of the talk I gave over at [kubernetes bangalore k8s september 2022 meetup](https://www.meetup.com/kubernetes-openshift-india-meetup/events/288277755/).

Here are the slides, which you can take a peek over, to complement this post.

## Context

I will not repeat the content which is already there in the slides. Will also update this post with the talk link when the talk gets uploaded. But I do want to delve over into the idea of how I feel I would attempt to structure the upgrades next.

### What is the bottleneck at the end of the day

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">To just add to the last idea of the main tweet. Having seen and maintained the terraform setup to create/maintain clusters at a couple of places, and this way of doing it doesn&#39;t work very well after you start reaching cluster nos in 10&#39;s or 100&#39;s and more (1/n) <a href="https://t.co/C9cSaE812Q">https://t.co/C9cSaE812Q</a></p>&mdash; Tasdik Rahman (@tasdikrahman) <a href="https://twitter.com/tasdikrahman/status/1521420689206648832?ref_src=twsrc%5Etfw">May 3, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

We have a state for the cluster, which also ends up being managed by the way we add/delete/create the cluster with, this could be terraform or eksctl config or cloudformation, whatever you have chosen.

It will increasingly become more manual since the state store is still managed via terraform, which by design will now tie you down to having to either edit these state files and modifications via terraform. Which is all good till you don't hit the above number of clusters

I am increasingly leaning towards a way to manage/create this state of the cluster which has a control loop, whereas the whole terraform style of state is to have interrupts around it, while it can very well be engineered to manage the drift which is hard to manage via terraform in a large enough inventory, the control loop style approach would be rather more efficient I feel.

The same is echoed in this tweet here.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">In general I prefer control-loop style Management of infrastructure, especially at scale. It’s easier to manage drift and be proactive about modifications. One-shot stuff like terraform gets fragile over time and it’s harder to evolve, turns out software is OK too.</p>&mdash; Lincoln Stoll (@lstoll) <a href="https://twitter.com/lstoll/status/1521412277152460800?ref_src=twsrc%5Etfw">May 3, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

### Ways to think about efficiently managing this drift and human bottleneck problem

As described in the last tweet, the control loop managing the final state of the infrastructure in question, makes us think more in the direction in which cluster-api is going. I have come across [https://github.com/openshift/managed-upgrade-operator](https://github.com/openshift/managed-upgrade-operator), which is the closest I have seen an OSS implementation so far to being well documented and how I would probably lean into solutioning this problem for a large scale cluster sprawl.

While there are more examples of automation of rolling upgrades for node groups out there [https://github.com/keikoproj/upgrade-manager](https://github.com/keikoproj/upgrade-manager) taking over the control loop approach for a specific subset of the problem statement of the upgrade process.

Along with [https://github.com/hellofresh/eks-rolling-update](https://github.com/hellofresh/eks-rolling-update) and [https://github.com/deliveryhero/k8s-cluster-upgrade-tool](https://github.com/deliveryhero/k8s-cluster-upgrade-tool)(P.S. is written by me), both of which are human operated, while although do solve the problem in case, but the openshift's implementation is a more complete autopilot way of doing the upgrade.

If you have implemented some other setup for a large sprawl of k8s clusters for yourself. Would love to hear more about it.

## References

- [https://github.com/openshift/managed-upgrade-operator/blob/master/docs/design.md](https://github.com/openshift/managed-upgrade-operator/blob/master/docs/design.md)
