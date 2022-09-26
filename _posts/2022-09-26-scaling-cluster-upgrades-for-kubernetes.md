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

Here are the slides, which you can take a peek over, to complement this post, if you would like to go through it before reading further.

<script async class="speakerdeck-embed" data-id="5c166502e7e0418dad72eb6c65849ca0" data-ratio="1.77725118483412" src="//speakerdeck.com/assets/embed.js"></script>

## Context

I will not repeat the content which is already there in the slides. Will also update this post with the talk link when the talk gets uploaded. But I do want to delve over into the idea of how I feel I would attempt to structure the upgrades next. This post is more on the infrastructure upgrade complexities arising from when managing double digit or more k8s clusters.

### What is the bottleneck at the end of the day

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">To just add to the last idea of the main tweet. Having seen and maintained the terraform setup to create/maintain clusters at a couple of places, and this way of doing it doesn&#39;t work very well after you start reaching cluster nos in 10&#39;s or 100&#39;s and more (1/n) <a href="https://t.co/C9cSaE812Q">https://t.co/C9cSaE812Q</a></p>&mdash; Tasdik Rahman (@tasdikrahman) <a href="https://twitter.com/tasdikrahman/status/1521420689206648832?ref_src=twsrc%5Etfw">May 3, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

We have a state for the cluster, which also ends up being managed by the way we add/delete/create the cluster with, this could be terraform or eksctl config or cloudformation, whatever you have chosen.

The way to introduce change in these types of setups is often more than not a human interaction happening over with them, it could be a terraform plan, apply for example in this case and the state of the change then gets stored in the state file.

Now imagine having to do the same over 10s or 100s of cluster.

I am increasingly leaning towards a way to manage/create the state of the k8s cluster, via some form of a control loop, whereas the whole terraform style of state is to have interrupts around it. While it can very well be engineered to reduce the interrupts, fundamentally I feel the difference in the way to operate the state is different from the terraform style and control loop style management.

This tweet further discusses this idea.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">In general I prefer control-loop style Management of infrastructure, especially at scale. It’s easier to manage drift and be proactive about modifications. One-shot stuff like terraform gets fragile over time and it’s harder to evolve, turns out software is OK too.</p>&mdash; Lincoln Stoll (@lstoll) <a href="https://twitter.com/lstoll/status/1521412277152460800?ref_src=twsrc%5Etfw">May 3, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

### Ways to think about efficiently managing this drift and human bottleneck problem

As described in the last tweet, the control loop managing the state of the infrastructure in question, makes us think more in the direction in which [cluster-api] tries thinking around managing infrastructure. I have come across [https://github.com/openshift/managed-upgrade-operator](https://github.com/openshift/managed-upgrade-operator), which is the closest I have seen an OSS implementation going towards a complete implementation of this style of operation. Reasonably well documented and how I would probably lean into solutioning this problem for a large scale cluster sprawl when trying to solve it the next time.

While there are more examples of automation of rolling upgrades for node groups out there in the form of [https://github.com/keikoproj/upgrade-manager](https://github.com/keikoproj/upgrade-manager), taking over the control loop approach for a specific subset of the problem statement of the upgrade process, in this case the node operations.

Along with [https://github.com/hellofresh/eks-rolling-update](https://github.com/hellofresh/eks-rolling-update) and [https://github.com/deliveryhero/k8s-cluster-upgrade-tool](https://github.com/deliveryhero/k8s-cluster-upgrade-tool)(P.S. is written by me), both of which are human operated, while although do solve the problem in case, a subset of it if not all, but the openshift's implementation is a more human interrupt free way to go about the cluster upgrade process, which is what will allow the cluster upgrade process to be sustainably executed at the end of the day, without a lot of toil and effort. Even though if we end up adding the operations of cluster upgrade process further to this way, operations like control plane upgrade, upgrade of the managed node groups and k8s components, we have to at the end of the day, wrangle with the state and modifying it further with the tools which we would have created/managing the specific k8s resources.

### Closing thoughts

The control loop strategy would make us the question or reconsider how we are creating and managing these k8s specific resources like control plane for the managed provider/self hosted cluster, managed node groups/self hosted node groups and the k8s components managed by helm. In any case, I foresee someone wanting to consolidate their automation or creation/management process back to the control loop automation which they have in order to have standardisation over time. More of this automation introduced would more more moving codepieces leading to maintenance creep, but the cost of which would be justifiable if you are measurably reducing the effort of the cluster upgrade process for the x number of clusters you own/manage/run.

The whole IAC being run via atlantis and having terragrunt to run plan/apply in the CI is a not so uncommon of a setup as of now, but the churn of cluster upgrades and the complexity which comes up with it, is the not the same as managing a couple of managed services via the terraform config, upgrade/delete of which is most of the times a one step plan/apply.

While to each to their own setup. But in general, reaching to a state where k8s clusters itself are not pets, is a fairly complex problem. If you have already reached that state, then your state of a new cluster creation is very mature, cleaning up of which might be the next logical step to think about, after which you can do a blue-green deployment of the new cluster setup. But again, I have not seen this work out successfully so far, hence, don't think of your k8s cluster as a pod by this extension of thinking. Save the state of the cluster in a way in which you can modify it over time with your tooling.

If you have implemented some other setup for a large sprawl of k8s clusters for yourself. Would love to hear more about it.

## References

- [https://github.com/openshift/managed-upgrade-operator/blob/master/docs/design.md](https://github.com/openshift/managed-upgrade-operator/blob/master/docs/design.md)
- [https://github.com/openshift/managed-upgrade-operator](https://github.com/openshift/managed-upgrade-operator)
- [https://github.com/hellofresh/eks-rolling-update](https://github.com/hellofresh/eks-rolling-update)
- [https://github.com/deliveryhero/k8s-cluster-upgrade-tool](https://github.com/deliveryhero/k8s-cluster-upgrade-tool)

