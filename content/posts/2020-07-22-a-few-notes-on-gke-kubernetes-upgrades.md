---
title: "A few notes on GKE kubernetes upgrades"
description: "A few notes on GKE kubernetes upgrades"
tags: [kubernetes, devops]
comments: true
share: true
cover_image: '/content/images/2020/07/k8s-upgrades-gojek-header.jpeg'
---

> This post was originally published in [Gojeks engineering blog, here](https://blog.gojekengineering.com/how-we-upgrade-kubernetes-on-gke-91812978a055), this post is a cross post of the same

This post is more of a continuation to this tweet

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">A few notes on <a href="https://twitter.com/kubernetes?ref_src=twsrc%5Etfw">@kubernetes</a> cluster upgrades on GKE (1/n)</p>&mdash; Tasdik Rahman (@tasdikrahman) <a href="https://twitter.com/tasdikrahman/status/1285619368353726465?ref_src=twsrc%5Etfw">July 21, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

If you are running kubernetes on [GKE](https://cloud.google.com/kubernetes-engine), chances are that you are already doing some form of upgrades for your kubernetes clusters, given that their release cycle is quarterly, which means you will have a minor version bump every quarter in the upstream. That is really a high velocity for version releases, but that's not the focus of this post, the focus is on how you can attempt to keep up with this release cycle.

Quite a few things are GKE specific, but overall at the same time, there are also a lot of things which apply to in general any kubernetes cluster, whether self hosted or a managed one.

That being said. Let's quickly set context, on what exactly do we mean by when we say a kubernetes cluster.

# Components of a kubernetes cluster

In any kubernetes cluster, it would consist of your master and worker nodes. Two sets of nodes for different kind of workloads to run on them. More on the kind of workloads which run on them.

The master nodes in the case of GKE are managed by googlecloud
itself, now what does it entail? It means, the components like api-server, controller-manager, etcd, scheduler etc, will not have to be managed by you in this case, which would have been an added operational burden.

<center><img src="/content/images/2020/07/components-of-kubernetes.png"></center>

I will not go into what each and every component goes into detail, as the docs do a good justice on what do they do, but just to summarise, the scheduler schedules your pods to nodes, the controller manager consists of a set of controllers used to control the existing state of the cluster and reconcile it with the state stored in etcd, the api-server is your entry point to the cluster, which is where each and every component comes to interact with other components.

## How should I create a cluster

We use [terraform](https://www.terraform.io/) along with gitops to manage the state of everything related to gcp, although I have heard good things about [pulumi](pulumi.com/), whatever works for you at the end of the day, but having the power of being able to declaratively configure the state of your infrastructure cannot be understated.

We have a bunch of cluster creation modules inside our private terraform repository, which makes creation of our GKE cluster, literally just a call to the module along with some sane defaults and the custom arguments which vary along with any cluster, a git commit and push, and then the next thing, which one sees is the terraform plan, right in the comfort of the CI, if all looks good, they do a terraform apply as the next step in the same pipeline stage.

With a few contextual things, about how we are managing the terraform state of the cluster, let's move on to a few defaults which we set.

By default, one should always choose [regional clusters](https://cloud.google.com/kubernetes-engine/docs/concepts/regional-clusters). The advantage of this is that, then GKE will maintain replicas of your control plane across zones, which makes your control plane resilient to zonal failures. Since the api-server is the entry to all the communication and interaction, this going down is basically you losing control/access to your cluster, but your workload will continue to run unaffected (if they don't depend on the api-server or k8s control plane, more on this in some)

Components like Istio, prometheus operator, etc and even good old kubectl which depend on the api-server, may not function momentarily as the control-plane is getting upgrades in case your cluster is not a regional cluster.

Although, in the case of regional clusters, I haven't personally seen any service degradation/downtime/latency increase while the master upgrades itself.

## Master upgrades come first before upgrading anything

The reason for that is, the control plane needs to be upgraded first and then the rest of the worker nodes.

When one is upgrading the master nodes(you will not see the nodes in GKE, but this would be running somewhere as part of VMs/borg pods et al. whatever google is using as an abstraction), the workloads running on it like the controller-manager, scheduler, etcd, the api-server are the components which are getting upgraded to the version of k8s which you are setting it to.

Master upgrades need to happen and then one can move on the worker node upgrades, the process of master node upgradation is very opaque in nature, as GKE manages the upgrade for you and not the cluster operator, which might not give you a lot of visibility on what exactly is happening. But nevertheless, if you just want to learn on what's happening inside, you can try [typhoon](https://github.com/poseidon/typhoon) and try upgrading the control plane of the cluster brought up using that, which I used to live upgrade the control plane of a self hosted k8s cluster in [devopsdays 2018 india's talk](https://www.youtube.com/watch?v=3WgqFoo9eek&feature=youtu.be).


## GKE cluster master upgraded, what next

The next obvious thing after you have done your GKE master node upgrade, is to upgrade your worker nodes, in the case of GKE, you would have node pools, which would in turn be having nodes being managed by the node pools.

Why different node pools? One can use separate node pools, to run different kind of node pools, which can then be used to segregate the workloads which run on the nodes of that node pool, for example. One nodepool can be tainted to run only prometheus pods, and then the prometheus deployment object can then tolerate that taint to get scheduled on that node.

## What consists of the worker nodes

This is the part of the compute infra, which is what you get to interact with if you are on GKE.

These are the node pools, where your workloads will run at the end of the day.

As to components which make up the worker nodes, (excluding your workload) would be
- kube-proxy
- container-runtime (docker for example)
- kubelet

Again, not going very deep into what each thing does, but on a very high level, kube-proxy is responsible for the translation of your service's clusterIP to podIP at the end of the day, along with that also nodePort.

Kubelet is the process, which actually listens to the api-server for incoming instructions to schedule/delete pods to the node in which it is running. This instruction is in turn translated to the api instruction set which the container runtime (docker, podman for example) understands.

These 3 components, are again managed by GKE, and whenever you are upgrading your nodes, kube-proxy and kubelet gets upgraded, the container runtime need not receive and update while you upgrade. GKE would have it's own mechanism to do it, but on a very high level, it does so by changing the image versions of the control plane pods.

We haven't seen a downtime/service degradation happening due to these components getting upgraded on the cluster.

One good thing to note here is that, the worker nodes can run a few versions behind the version of the master nodes, the exact versions can be tested out on your staging clusters, to have more confidence while doing your production upgrade. But for example, I have seen if master is on 1.13.x, the nodes run just fine while even if they are on 1.11.x. Recommended is only a 2 minor version skew.

## Anything one should check while upgrading to a certain version?

Since the major [release cycle](https://github.com/kubernetes/kubernetes/releases) is quarterly for kubernetes, one thing for sure which operators have to check is the release notes and the [changelog](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/) for each version bump, as they usually entail quite a few api deletions and major changes.

## What happens if my cluster is regional while upgrading it

If your cluster is regional, the node upgrade happens zone by zone. You can control the number of nodes which can be upgraded at once using the surge configuration for the node pool, turning autoscaling off for the node pool is also recommended during the node upgrade upgrade.

If surge upgrade is enabled, a surge node with the upgraded version is created and it waits till the kubelet registers itself to the api-server, marking it ready after the kubelet reports the node as healthy back to the api-server, at which point, the api-server can direct the kubelet running on the surge node to schedule any workload pods

In case of a regional cluster, another node from the same zone is picked up to be drained, after which the node is cordoned, it's workload rescheduled and then the end of this, the node gets deleted and removed from the nodepool.

## Release channels

Setting a [release channel](https://cloud.google.com/kubernetes-engine/docs/concepts/release-channels) is something which is highly recommended, we set it to stable for the production clusters, and the same for our integration clusters, with that being set, the nodes will always run the same version of kubernetes as the master nodes (excluding the small amount of time when the master is getting upgraded.)

There are 3 release channels, depending on how fast you want to keep up with the kubernetes versions released upstream
- rapid
- regular (default)
- stable

Setting maintenance windows will allow you to control when these upgrade operations are supposed to kick in, once set, the cluster cannot be upgraded/downgraded manually, so if one doesn't really care about granular control, they can not choose this option.

I haven't personally downgraded a master version, please try this out on a staging cluster if you really need to. Although if you look at the docs, downgrading master is not really [recommended](https://cloud.google.com/kubernetes-engine/docs/how-to/upgrading-a-cluster#downgrading_limitations).

Downgrading a node pool version is not possible, but you can always create a new node pool, with the said version of kubernetes and delete the older node pool.

## Networking gotchas while upgrading to a version 1.14.x or above

If you are running a version lesser than 1.14.x and don't have the ip-masq-agent running and if your destination address range falling under the CIDR's

- 10.0.0.0/8
- 172.16.0.0/12
- 192.168.0.0/16

the packets in the egress traffic will not be masqueraded, which means that the node IP will be seen in this case.

The default behaviour after 1.14.x (and on [COS](https://cloud.google.com/container-optimized-os/docs)), packets flowing from the pods stop getting NAT'd. This can cause disruption as you might not have whitelisted the pod address range.

One way is to add the [ip-masq-agent](https://cloud.google.com/kubernetes-engine/docs/how-to/ip-masquerade-agent) and add the config for the nonMasqueradeCIDRs list the destination CIDR's like 10.0.0.0/8 (for example if this is where your destination component like postgres lies), in this case the packets will use the podIP as the source address when the destination (postgres) receives the traffic and not the nodeIP.

## Can I upgrade multiple node pools at once?

No you can't, GKE doesn't allow you to do so, even when you are upgrading one node pool, the node which get's upgraded and picked for upgraded is not something which you will have control over.

So far we have discussed on what happens when the node pools get upgraded and the master node pools getting upgraded by GKE

## Would there be a downtime for my services when we do an upgrade?

Let's start with the master component, if you have a regional cluster, while upgrading, since it happens zone by zone, even if your service is making use of the k8s api-server to do something, it will not get affected, although you should definitely try replicating the same for your staging setup assuming both have similar config.

Coming to the worker nodes. Well it depends.

## How do I prevent/minimise downtime for my services deployed?

For stateless applications, the simplest thing to do is to increase the replicas to reflect the number of zones in which your nodes are present. But it need not to be necessary that scheduling of pods happen on each node across zone, kubernetes by default doesn't handle this but gives you the primitives to handle this case.

If you want to distribute pods across zones, you can apply [podantiaffinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity) in the deployment spec for your service, with the topologyKey set to `http://failure-domain.beta.kubernetes.io/zone` for the scheduler to try scheduling it across zones. (You can read a more detailed post on scheduling rules which you can specify which I wrote sometime back [here](https://www.tasdikrahman.com/2020/05/06/specifying-scheduling-rules-for-pods-with-podaffinity-and-podantiaffinity-on-kubernetes/))

Distribution across zones will make your service resilient to zonal failures, the reason we increase the replicas to greater than 1 is that, when the nodes get upgraded, the node gets drained and cordoned and the pods get bumped out from that node.

In case the service which has only 1 replica, and it is scheduled on the node which is set for upgrade by GKE, whilst the scheduler tries finding itself a new node, there would be no other pod which is serving requests, which would cause a temporary downtime in this case

One thing to note here is that, if you are using [PodDisruptionBudget(PDB)](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/), and if your running replicas are the same as the minAvailable specified in the PDB rule, the upgrade will just not happen, this is because of the fact that the node will not be able to drain the pod(s), as it respects the PDB budget in this case, hence you have to either
- increase the pods such that the running pods are > minAvailable specified in your PDB
- remove the PDB rule specified.

For [statefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/), you might have to take a small dowtime while you are upgrading, as the pods on which the stateful set is scheduled, that gets bumped out, the pvc claim will again be made by another pod, when it gets scheduled on the other node.

## But Tasdik these steps of upgrade are so mundane

Agreed, it is mundane, but there's nothing stopping anyone from having a tool do these things for you, have a look at [eks-rolling-update](https://github.com/hellofresh/eks-rolling-update), GKE is way easier if you look at it. Which makes fewer touch points and cases where things can go wrong

- one being the pdb budget being the showstopper for your upgrade if you don't pay attention
- replicas being 1 or so for services
- quite a few replicas being in pending or crashloopbackoff
- statefulsets being an exception and needing to be handholded.

For most of the above, you can initially start with having a fixed process (playbook) which one needs to follow and run through for each cluster whenever you are upgrading, so even though if the task is mundane, one knows which checks to follow and what to do to check the sanity of the cluster after the upgrade is done.

Replicas being set to 1 is just being plain naive, let your deployment tool have sane defaults of having replicas of 3 for minimum (3 zones in 1 region assuming you have podantiaffinity and a best case effort gets logged by the scheduler)

For the pods being in pending state, it means, you either are trying to request cpu/memory which is not available in any of the nodes present in the node pools, which again means, you are either not sizing your pods correctly, or there are a few deployments which are hogging resources, either way, it's a smell that you are not having enough visibility into your cluster.

For statefulsets, I don't think you can prevent not taking a downtime. So that's there.

After all the upgrades are done, you can backfill the upgraded version numbers and other things back to your terraform config in your git repo.

Once you have rinsed and repeated these steps above, you can very well start with automating a few things.

## What we have automated

We have automated the part, where the whole analysis of what pods are running in the cluster, we extract this information out in an excel sheet. Information like
- replicas of the pods
- age of the pod
- status of the pods
- which pods are in pending/crashloopbackoff
- node cpu/mem utilization

The same script handles inserting the team ownership details of the service, by querying our service registry and storing that info.

So all of the above details, at your tips, by just running the script in your command line and switching context to your clusters context.

As of now, the operations like
- upgrading the master nodes to a certain version
- disabling surge upgrades/autoscaling the nodes
- upgrading the node pool(s)
- reenabling surge upgrades/autoscaling
- setting maintenance window and release channel if already not set

All the above being done via the CLI.

The next parts would be to automate the sequence in which these operations are done and codify the learnings and edge cases to the tool.

This is a bit mundane, no doubt. But this laundry has to be done, there's no running away from it. Until one has automated the whole/major chunk of it, what we are currently doing in our team is to rotate people around doing cluster upgrades.

One person gets added to the roster, while there is a person who is already in the roster from the last week who will drive the upgrade for the week, also while giving context to the person who has just joined.

This helps in quick context sharing as well as the person who has just joined, they get to upgrade the clusters by following the playbooks, hence filling the gaps as we go forward.

The important part is that, you always come out of the week, with something improved, some automation added, some docs added. While also allocating dev time for automation explicitly in your sprint.

## Ending notes

All in all, GKE really has a stable base which in turn allows us to focus more on building the platform on top of it rather than managing the underlying system and improve the developer productivity by building out tooling on top of the primitives k8s gives you.

If you compare this to something like running your own k8s cluster on top of VMs, there is a massive overhead of managing/upgrading/replacing components/nodes of your self managed cluster which in itself does require dedicated folks to handhold the cluster at times.

So if you really have the liberty, a managed solution is the way to go, take this from someone who has managed prod self hosted k8s clusters, and I will be honest, it's definitely not easy, and something which if you can, should be delegated to focus on other problems

### References

- [https://cloud.google.com/kubernetes-engine/docs/concepts/regional-clusters](https://cloud.google.com/kubernetes-engine/docs/concepts/regional-clusters)
- [https://github.com/kubernetes/kubernetes/releases](https://github.com/kubernetes/kubernetes/releases)
- [https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/)
- [https://cloud.google.com/kubernetes-engine/docs/concepts/release-channels](https://cloud.google.com/kubernetes-engine/docs/concepts/release-channels)
- [https://cloud.google.com/container-optimized-os/docs](https://cloud.google.com/container-optimized-os/docs)
- [https://cloud.google.com/kubernetes-engine/docs/how-to/ip-masquerade-agent](https://cloud.google.com/kubernetes-engine/docs/how-to/ip-masquerade-agent)
- [https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity)
- [https://www.tasdikrahman.com/2020/05/06/specifying-scheduling-rules-for-pods-with-podaffinity-and-podantiaffinity-on-kubernetes/](https://www.tasdikrahman.com/2020/05/06/specifying-scheduling-rules-for-pods-with-podaffinity-and-podantiaffinity-on-kubernetes/)
- [https://kubernetes.io/docs/concepts/workloads/pods/disruptions/](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/)
- [https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
- [https://kubernetes.io/docs/tasks/run-application/configure-pdb/](https://kubernetes.io/docs/tasks/run-application/configure-pdb/)
- [https://cloud.google.com/kubernetes-engine/docs/how-to/upgrading-a-cluster#downgrading_limitations](https://cloud.google.com/kubernetes-engine/docs/how-to/upgrading-a-cluster#downgrading_limitations)

## Credits

- Image credits to [wikipedia](https://en.m.wikipedia.org/wiki/File:Chrome_Vanadium_Adjustable_Wrench.jpg) and [kubernetesio](https://kubernetes.io)
