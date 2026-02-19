---
title: "To self host or to not self host kubernetes cluster(s)"
description: "To self host or to not self host kubernetes cluster(s)"
tags: [kubernetes]
comments: true
share: true
cover_image: '/content/images/2019/04/k8s-image.png'
---

A friend of mine asked this to me recently, about how was it to [self host](https://en.wikipedia.org/wiki/Self-hosting) [kubernetes](https://kubernetes.io) clusters. And I was cursing myself about why I did not write this post earlier (I mean, technically I have written about how we used to do self hosting [before](https://www.tasdikrahman.com/2019/04/04/self-hosting-highly-available-kubernetes-cluster-aws-google-cloud-azure/), but not the pros and cons of it), as this was not the first time I had been asked this question. So this post is dedicated to my friend and to others when they chance upon this question.

Just for context, self host here does not only mean, [kubernetes inside kubernetes](https://www.tasdikrahman.com/2019/04/04/self-hosting-highly-available-kubernetes-cluster-aws-google-cloud-azure/), but in a broader sense, would mean, managing the control plane of the kubernetes cluster too, along with the worker nodes (which is the usual case these days with the cloud vendor k8s options). You would preferably be using solutions like [kops](https://github.com/kubernetes/kops), or [typhoon](https://github.com/poseidon/typhoon) here to self host your kubernetes clusters. But there are a bunch of really great tools out there these days apart from these two, which I personally like.

I had to pry out this tweet thread which I wrote about sometime back and it's a good summary more or less.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">If you can, don&#39;t try managing your own <a href="https://twitter.com/kubernetesio?ref_src=twsrc%5Etfw">@kubernetesio</a> clusters. It can become a huge engineering effort in itself very quickly. If the core business product is not around providing infrastructure to others, using <a href="https://twitter.com/hashtag/kubernetes?src=hash&amp;ref_src=twsrc%5Etfw">#kubernetes</a> solutions provided by cloud providers is not a bad idea</p>&mdash; Tasdik Rahman (@tasdikrahman) <a href="https://twitter.com/tasdikrahman/status/1110927284926447617?ref_src=twsrc%5Etfw">March 27, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

### Don't self host unless you have a very good reason to

The very obvious answer if you ask why, is because with all honesty, it's a huge engineering effort to even run and manage vendor managed kubernetes clusters, let alone self hosted ones. I have already written quite a lot, on a bunch of reasons, one particular problem, from which there's no running away away from, is kubernetes upgrades, which I have covered in detail [here](https://www.tasdikrahman.com/2020/07/22/a-few-notes-on-gke-kubernetes-upgrades/) on what entails in one such upgrade, for you to understand the complexity of things.

And there are a bunch of things, which I have mentioned in this [post](https://www.tasdikrahman.com/2020/11/21/choosing-between-one-big-cluster-or-multiple-smaller-kubernetes-clusters/) on the complexities which entail with managing a kubernetes cluster, vendor managed or self-hosted alike and how effort multiplies as one goes into the direction of multiple clusters and different cluster sizes. If you have read the above two posts, you might just feel this post getting a bit repeated at this point, hence I will not delve into the exact details, as I have covered them in depth in the other two posts.

### But Tasdik, I need to customize my kubernetes installation, I will lose that power if I go ahead with a vendor

A lot of flexibility in terms of customisations, in a vendor managed kubernetes cluster is alreay being provided these days. But even then, if you are really trying to use/set something esoteric which is really important to you and is required for your workloads, do keep in mind the maintenance of the cluster. If you do end up self-hosting it, you would need to know how to operate it and the propblems which come and the corner/edge cases with it.

An event like switching out our [CNI](https://chrislovecnm.com/kubernetes/cni/choosing-a-cni-provider/), moving from kube-dns to [core-dns](https://kubernetes.io/docs/tasks/administer-cluster/coredns/), for example, is best abstracted if possible.

I get that people do run other things like postgresql or mysql on VM's, but postgresql doesn't need an upgrade every few months, and would just about run for years without an upgrade without too many hiccups for most part.

And not to forget kubernetes is a really fast moving project. [v1.4.6](https://github.com/kubernetes/kubernetes/releases/tag/v1.4.6) was released around in november, 2016. It was also a version (~1.8 to be precise) close to which we started running our production workloads back in 2017 for an org. Come 2020 november, we already have [v1.18.12](https://github.com/kubernetes/kubernetes/releases/tag/v1.18.12), and LTS not a thing as of now in the kubernetes world. And not to mention the amount of changes which happen each release. As with any fast moving project, things are bound to change, as [Joe](https://www.infoq.com/podcasts/joe-beda-kubernetes-cncf/) mentions here that kubernetes should be boring. Kubernetes will take some time, to become something similar to that. On a related note, this is a very good piece on that [topic](https://mcfunley.com/choose-boring-technology).

There's just a mouthful of things to take care of too, in case you decide to manage your own [etcd clusters](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/), along with the control plane components, if you decide to self host. There's great learning for sure in all this, but the idea is to provide a reliable peice of infrastructure over to your customers here first (assuming you work in the operations/platform team, your customers are your developers), unless the team is really rock solid in terms of their k8s knowledge and is able to keep up with self hosting, this sort of path can very quickly become a possible wrong decision for your team.

And not to forget the automation which needs to be maintained to bring up/upgrade parts if not all of the whole self-hosted cluster automation. The automation can vary based on the solution which you picked up to orchestrate the creation of your cluster and could be anything from shell scripts/terraform modules etc. The point is that, if it's not a standard solution out there and you have hand smashed a lot of automation yourself, the added overhead of maintaing that automation shows up over time.

### Ending notes

Unless providing kubernetes as a PaaS is your bread and butter and your main business, where you compete against others and have to absolutely innovate and stay on top of the game. In most cases you are better off just using a cloud vendor managed offering.

We ended up self-hosting back in the days, as the cloud vendored k8s offering was not present in our region and due to constraints of compute, being present on that particular region and no particular one, made us go the self-hosting route.

Having managed(both self hosted and vendor managed) and built platforms on top of multiple k8s clusters(cloud vendor managed control plane in this case), for the better part of 3 and a half years now, one thing for sure I love about k8s is the power it gives. The ease of deployments([rolling deployments](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/) out of the box), the immutable nature of deployments(containerized workloads), the programmatic interface for starters([kubernetes/client-go](https://github.com/kubernetes/client-go), need I even say more) and I am not even scratching the surface.

But at the same time, the right tradeoffs do need to made. Kubernetes absolutely solves some very core problems, but if you're just in the phase of building your product, with not much of bandwidth. Would highly recommend thinking twice in case of going the self hosting route or even kubernetes, if you must for it to not become a distraction from the core task, which is reliable infrastructure. You might just be better off with plain simple [ASG's](https://docs.aws.amazon.com/autoscaling/ec2/userguide/AutoScalingGroup.html) and [instance groups](https://cloud.google.com/compute/docs/instance-groups) for that matter with [AMI's](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html). Less moving part is good to start with. But again, depends on your team, if everyone has a good amount of production experience with k8s, then why not? But even then, vendor managed k8s installation will again be my personal pick.

If you are in a very specialised environment where you absolutely know what you are doing and you must, this post is obviously not for you, in which case I would love to hear more about your learnings :).

Coincidentally, my last post was also about the trade-offs in specific situation similar to this situation, when dealing with kubernetes. Hope you have found this piece informative.

### Links

- [https://www.tasdikrahman.com/2020/07/22/a-few-notes-on-gke-kubernetes-upgrades/](https://www.tasdikrahman.com/2020/07/22/a-few-notes-on-gke-kubernetes-upgrades/)
- [https://www.tasdikrahman.com/2019/04/04/self-hosting-highly-available-kubernetes-cluster-aws-google-cloud-azure/](https://www.tasdikrahman.com/2019/04/04/self-hosting-highly-available-kubernetes-cluster-aws-google-cloud-azure/)
- [https://www.tasdikrahman.com/2020/11/21/choosing-between-one-big-cluster-or-multiple-smaller-kubernetes-clusters/](https://www.tasdikrahman.com/2020/11/21/choosing-between-one-big-cluster-or-multiple-smaller-kubernetes-clusters/)

### Credits

- Thanks to [nemo](https://captnemo.in/) for proofreading and the folks at [kubernetes](https://kubernetes.io/) for the post image.
