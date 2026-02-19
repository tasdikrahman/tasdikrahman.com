---
title: "Choosing between one big cluster or multiple smaller kubernetes clusters"
description: "Choosing between one big cluster or multiple smaller kubernetes clusters"
tags: [kubernetes]
comments: true
share: true
cover_image: '/content/images/2019/04/k8s-image.png'
---

This post is a continuation on the discussion which I was having with [@vineeth](https://twitter.com/VineethReddy02)

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">But why would someone choose one large cluster over multiple small clusters? Aren&#39;t multiple clusters already a pattern in enterprises?</p>&mdash; Vineeth Pothulapati (@VineethReddy02) <a href="https://twitter.com/VineethReddy02/status/1329656769900003329?ref_src=twsrc%5Etfw">November 20, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Context is when I came across a tweet which demonstrated the ability of kubernetes to scale uptill 15k nodes due to recent improvements.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">15k nodes cluster 🤯 <a href="https://t.co/VMWI7HeYHH">https://t.co/VMWI7HeYHH</a></p>&mdash; Tasdik Rahman (@tasdikrahman) <a href="https://twitter.com/tasdikrahman/status/1329615066405093379?ref_src=twsrc%5Etfw">November 20, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

The discussion was originally around costs and how much would it take to run one such large kubernetes cluster, but it went into a different direction altogether.

So what should the decision be? Rather, I would like to take the path, where we discuss a few things about both sides of the coin. While this post is not a recommendation on what one should do, but the idea is to guide you to take a more informed decision with the data points, trade-offs and constraints that you have.

Before we start, big, here will be relative, but for the sake of this conversation, let's say you plan to run a cluster, with worker nodes greater than ~50 (this is a big cluster for me right now, more on the why part of it in a bit) and a small cluster for the context of this post is say, 5-10 worker nodes.

#### Basic house keeping

Having separate cluster(s) for staging and production, similar to how you would have been following the same for your compute infrastructure on VM's.

#### Multi tenancy

Hard multi-tenancy is something, which might not work the way as expected as of now on k8s, even with [netpols](https://kubernetes.io/docs/concepts/services-networking/network-policies/)/[namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) and upcoming mechanisms like [Hierararchical namespaces (aka HNC)](https://github.com/kubernetes-sigs/multi-tenancy/blob/master/incubator/hnc), which is still in incubation at the time of writing this.

That's what I have last checked, but please correct me here if you came across something which tells otherwise.

So if your workloads do have a hard requirement for this, multiple k8s clusters would be a way here,

#### Upgrade charter

One operational aspect which is important to note here is the burden of upgrades. Keeping up with upgrades, with the release cycle of kubernetes is not a trivial task. I have written about our experiences in this thread sometime back, to give you an idea of what really goes into one such upgrade.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">A few notes on <a href="https://twitter.com/kubernetes?ref_src=twsrc%5Etfw">@kubernetes</a> cluster upgrades on GKE (1/n)</p>&mdash; Tasdik Rahman (@tasdikrahman) <a href="https://twitter.com/tasdikrahman/status/1285619368353726465?ref_src=twsrc%5Etfw">July 21, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Even on a managed platform provided by the cloud vendors (where they maintain the control plane for you), it's still an operational heavy task, and we are not even talking about upgrades on self hosted and managed clusters via tools for eg: [kops](https://github.com/kubernetes/kops/).

Multiply this effort with multiple such clusters, and you will end up needing to have people dedicated to just do this. There is no such thing as an [LTS](https://en.wikipedia.org/wiki/Long-term_support) release as of now, unless you are fine with running super old kubernetes installations which would have CVE's reported and fixed in the upcoming releases/you are ok with doing big bang upgrades. Both of which might not be a great idea to begin with. Even if someone decides to run an archaic installation for long, if you feel running a super old installation will fly with your cloud provider, you will be in for a rude shock, where they can literally force upgrade your cluster(yes, they do it).

All of the toil which goes into cluster upgrades might have been a factor, which would have led to the initiation of this discussion [here](https://github.com/kubernetes/sig-release/issues/1290) where folks discuss about modifying the kubernetes release cadence. I for one [did vote](https://github.com/kubernetes/sig-release/issues/1290#issuecomment-709774428), on moving to 3 releases than 4 every year. But there are also arguments against doing slower releases, which might make the next release bloated with features/deprecations and going against the philosophy of small incremental changes, but that discussion is for another post.

Given someone ends up with a large cluster, the upgrade is equally gonna be as operationally heavy, if not more than when someone was upgrading a bunch of clusters, as the operations to be done would remain more or less the same.

What might increase a side effect of the cluster being huge, is that someone would have to baby sit the whole upgrade operation longer, than what they would spend on a smaller cluster(given upgrade automation is not present/not mature enough to remove the human involved here). The upgrades here, have to be done one ASG group/node pool, at a time, and takes time even if done for a smaller cluster. Imagine having to do it with a large cluster size, having 100's of nodes in each node pool. But I am sure there are ways in which someone would have improvised here, but I am yet to come across something like that.

The eventual state is to have automation which does a bunch of domain specific operations, and the human has to only get paged/notified in case when the upgrade gets done and dusted/there was some failure which occured, which the decision tree wasn't able to resolve itself. [Keiko](https://github.com/keikoproj/) project's [upgrade manager](https://github.com/keikoproj/upgrade-manager) is one such tool which comes to mind, apart from [https://github.com/hellofresh/eks-rolling-update](https://github.com/hellofresh/eks-rolling-update).

And if there are non-standardized workloads in the cluster(folks hand applying yamls/no track of objects in cluster), there are various ways on how an upgrade can either get stuck due to pdb budgets of pods not being met when the node gets cycled/end up causing an outage. Which makes this whole operation for a focused cluster running x teams applications much more saner.

Partly due to the fact that, now if the cluster is owned by x team, you can at least ask them to be on call with you during the upgrade process, so that they can at least shadow the person doing the upgrade and monitor the business metrics of the team's products being served out of the cluster being upgraded.

Hence, upgrade strategy is something to consider seriously while choosing the multiple clusters vs single cluster strategy.

#### API deprecations

API's getting deprecated, for example [1.16](https://kubernetes.io/blog/2019/07/18/api-deprecations-in-1-16/) was a release which had a bunch of changes, which would have involved the cluster operators to upgrade their automation/helm charts to be compatible with those changes, if they had plans on upgrading from some x version to v1.16.

There's no one to blame here in case of API deprecations, an object getting stabler and getting promoted to a more stable API, is a natural progression. To enjoy the benefits of a stable kubernetes object, it only makes sense to move to the stable api rather than being stuck on one which is less stable/getting deprecated in the next release.

For a larger org having bandwidth, managing multiple clusters for teams, they will eventually have automation over time to reduce the toil.

But if you're a small org, with a small group of folks managing the k8s clusters, the manual toil will be quite high. The reasons are also obvious, the lack of bandwidht will attribute to them not being able to automate the redundant tasks required for the upgrade. Even if they manage to write some automation, the automation will become stale over time if not given prioritisation to maintain it, as the domain changes. Plus, an average operations team will also have developer requests coming in their way, prioritising all this along with tasks such as maintaing your k8s cluster? Definitely a hard task to begin with.

Although I have not personally used it, but I have heard good things about [kube-no-trouble](https://github.com/doitintl/kube-no-trouble), which takes a stab at telling the deprecated API's in a cluster.

#### Access management

Giving the right kind of access to the developers/operational folks/xyz person in the team is necessary problem to solve, unless you are giving admin privileges to only the operational folks/one groups of people. Even then, solving the same problem over for multiple clusters requires automation and a proper mechanism.

What is to be done when a person leaves/joins? How can access be granted in granular manner for certain rbac roles which are only to be given read access, but no deletion access. If you have been in a position, where someone from the team has deleted the [deployment object](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) of your service and you ended up with an outage. This can happen with anyone, but reducing the blast radius is not a bad idea.

If your workloads are sensitive (payments data etc.), you would require access for such environments to be tighly audited and managed.

There are a few tools out there which would help you in doing so, like [krane](https://github.com/appvia/krane), [audit2rbac](https://github.com/appvia/krane), which help you in this process (thanks to [rahul](https://rmenn.in) for pointing them out to me).

#### Deployment automation

With all honesty, hand applying yaml files will only go so far. It works, but is a recipe for disaster over the long run, creating more spaghetti in the cluster and creating more problems than solving them. Problems like, who applied x resource object, who changed x resource, who is using this x resource. And the list so goes on. Multiple clusters or a single large cluster doesn't matter here in this context, but someone editing something which they are not supposed to for some other teams product?

Namespace as a service, to teams, is a model which I have heard people doing, keeping all access to specific roles tied to a particular namespace.

Continous Delivery is hardly a requirement for anyone(mostly?), but the ability to reliably deploy something is a hard requirement in most cases though.

Having some form of CI, to safely modify the respective resources via a tool/process will prevent a lot of surprises in production. Audit trail logs in the CI pipeline for someone to see what happened, automated rollbacks too? That would be a sweet spot I would say. (We have built out our deployment platform similar to these practices described in my current org, but more on that in another post)

There are multiple tools out there which allow fine grained RBAC rules, CI/CD, progressive delivery to your kubernetes clusters, [flagger](https://flagger.app/), [argo](https://argoproj.github.io/argo-cd/) being a few ones to name.

#### Concentration risk

If we look at the flip-side, having one big cluster, concentrates the risk of failure.

If you are not on a vendor specific kubernetes installation, what happens when the control plane goes for a toss? 1 person having all the context is not scalable, what happens when the person who knows the operational know-how, to fix x problem, is not present to handle the pager?

What about zonal failures affecting the cluster? Do we have checks and balances to handle such an event?

What if the [requests and limits](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits) for memory and CPU were not set properly for foo service in foo namespace, that it ended up hogging more cpu and memory, affecting baz service from baz team(baz service's getting degraded). It's not a great place to be in. Overprovisioning pods for teams/services and then gradually checking the trendline usage over time and then setting them over in requests and limits is one way to have sanity in the cluster and sizing it accordingly. More on [resource quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/) in another post.

Having one cluster, per product group is also a model which people follow, helping de-risk the affects of failure to not affect other products/product groups. But then the operation problem/complexity of managing multiple clusters arise.

#### Ending notes

Either of the two options, one large cluster vs multiple smaller clusters, both are an opinionated way to run clusters, or for that matter any compute infrastructure. What works best, might not work out in another context. Someone else's best practice might turn out into a nightmare for another org/team to manage/run.

Given, most/all of these problems can be solved, the right trade-offs can be made when deciding for a solution and I hope this discussion helps you making the right decision in your context.

P.S. A good amount of discussion also happened on [/r/kubernetes](https://reddit.com/r/kubernetes), here in [this thread](https://www.reddit.com/r/kubernetes/comments/jy85ly/choosing_between_one_big_cluster_or_multiple/)

### References

- [https://kubernetes.io/docs/concepts/services-networking/network-policies/](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
- [https://blog.gojekengineering.com/how-we-upgrade-kubernetes-on-gke-91812978a055](https://blog.gojekengineering.com/how-we-upgrade-kubernetes-on-gke-91812978a055)
- [https://kubernetes.io/blog/2019/07/18/api-deprecations-in-1-16/](https://kubernetes.io/blog/2019/07/18/api-deprecations-in-1-16/)
- [https://github.com/kubernetes/sig-release/issues/1290](https://github.com/kubernetes/sig-release/issues/1290)
- [https://github.com/kubernetes-sigs/multi-tenancy](https://github.com/kubernetes-sigs/multi-tenancy)
- [https://github.com/kubernetes-sigs/multi-tenancy/blob/master/incubator/hnc](https://github.com/kubernetes-sigs/multi-tenancy/blob/master/incubator/hnc)
- [https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits)
- [https://github.com/kubernetes/community/blob/master/contributors/design-proposals/node/resource-qos.md](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/node/resource-qos.md)
