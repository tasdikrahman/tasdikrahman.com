---
title: "Specifying scheduling rules for your pods on kubernetes"
description: "Learnings on scheduling rules native to the default k8s scheduler"
tags: [kubernetes, devops]
comments: true
share: true
cover_image: '/content/images/2019/04/k8s-image.png'
---

This is more of an extended version of the tweet here 

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">If you haven&#39;t had a look at pod-affinity and anti-affinity, it&#39;s a great way which one can use to distribute the pods of their service across zones. <a href="https://t.co/iqhbyhruD8">https://t.co/iqhbyhruD8</a> (1/n)</p>&mdash; Tasdik Rahman (@tasdikrahman) <a href="https://twitter.com/tasdikrahman/status/1231635358363729920?ref_src=twsrc%5Etfw">February 23, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 


PodAntiAffinity/PodAffinity were released in beta some time back in 2017, in the [1.16 release](https://kubernetes.io/blog/2017/03/advanced-scheduling-in-kubernetes/) for k8s, along with node affinity/anti-affinity, taints and tolerations and custom scheduling.

Depending on your use case, you can definitely use a few of them for specific type of workloads, to achieve certain outcomes, for things running in your k8s cluster.

Will talk about how you can achieve distributing your pods across zones for your service using pod-affinity and anti-affinity.

One can use `preferredDuringSchedulingIgnoredDuringExecution`, which can be used for podAntiAffinity, for the scheduler to not be fixated on the constraints you put, rather it would give a best case effort to schedule the pods based on your constraints.

podAntiAffinity would use the pod labels to tell the scheduler, if pods of a particular label (eg: `app: foo`) are already scheduled on a node, then it would to schedule the pod in a node different than the one where the pod with above label is already scheduled. 

The pods can be spread across zones by specifying `topologyKey: http://failure-domain.beta.kubernetes.io/zone` under `podAffinityTerm`, this assumes that the nodes are already spread across different zones to prevent catastrophe arising from zonal failure, having a regional cluster is a good idea

If you have multiple rules under `podAffinityTerm`, you can specify weights to them to tell the scheduler the importance of each rule, if you have just one rule, you can specify `100` to it.

A very simple example of podAntiaffinity being specified for a deployment, where we let the scheduler take a best case approach

<script src="https://gist.github.com/tasdikrahman/b706b8e9c57d8b7ec3e311ab0dacb188.js"></script>

You can notice, that the pods present here are getting scheduled in the same node, while the scheduler was trying it's best to spread the pods out. If you had presumable 3 nodes, in 3 different zones, the rule above would have tried spreading out the pods in all the three zones, giving you resiliency over zone failures.

For the purpose of this example, I have a single node cluster in place.

```
$ k get nodes
NAME                 STATUS   ROLES    AGE   VERSION
kind-control-plane   Ready    master   97s   v1.14.10
```

```
$ k get pods -owide -l 'app=store'
NAME                          READY   STATUS    RESTARTS   AGE    IP           NODE                 NOMINATED NODE   READINESS GATES
redis-cache-566bcff79-2qrzr   1/1     Running   0          110s   10.244.0.6   kind-control-plane   <none>           <none>
redis-cache-566bcff79-grhwk   1/1     Running   0          110s   10.244.0.7   kind-control-plane   <none>           <none>
redis-cache-566bcff79-ktmxv   1/1     Running   0          110s   10.244.0.5   kind-control-plane   <none>           <none>
```

One thing to note here is that, you can specificy multiple `matchExpressions` inside `podAffinityTerm.labelSelector`.

So if you wanted the scheduler to take into consideration another label like `app_type: server`, you can do something like

```
- podAffinityTerm:
    labelSelector:
    matchExpressions:
    - key: app
        operator: In
        values:
        - store
    - key: app_type
        operator: In
        values:
        - server
```
so both the match expressions will be `AND`'d while the scheduler is evaluating the rules.

Now if we would want this rule to be enforced while being schduled you can use `requiredDuringSchedulingIgnoredDuringExecution` instead of `preferredDuringSchedulingIgnoredDuringExecution`

<script src="https://gist.github.com/tasdikrahman/927f44bf2efb1dacea51f316d4c53e42.js"></script>

You would notice that the other two pods go into `Pending` state, since there is only one node present to spread out, given we are trying to enforce a required during scheduling rule, the scheduler would keep trying to schedule the other two pods to a node which doesn't already have a pod with the label `app: store`, which isn't able to find as there is only one node. 

```
$ k get pods -owide -l 'app=store'
NAME                           READY   STATUS    RESTARTS   AGE   IP           NODE                 NOMINATED NODE   READINESS GATES
redis-cache-66b88fd4fc-7dc65   0/1     Pending   0          6s    <none>       <none>               <none>           <none>
redis-cache-66b88fd4fc-sbh5z   0/1     Pending   0          6s    <none>       <none>               <none>           <none>
redis-cache-66b88fd4fc-wszsq   1/1     Running   0          6s    10.244.0.8   kind-control-plane   <none>           <none>
```

Checking the logs of one of the pending pods would give the same

```
Events:
  Type     Reason            Age                  From               Message
  ----     ------            ----                 ----               -------
  Warning  FailedScheduling  18s (x3 over 3m16s)  default-scheduler  0/1 nodes are available: 1 node(s) didn't match pod affinity/anti-affinity, 1 node(s) didn't satisfy existing pods anti-affinity rules.
```

“IgnoredDuringExecution” means that the pod will still run if labels on a node change and affinity rules are no longer met. There were plans to have `requiredDuringSchedulingRequiredDuringExecution` which will evict pods from nodes as soon as they don’t satisfy the node affinity rule(s), but I haven't seen that in the release docs, so not sure when is it scheduled to be added.

### Why not use node anti-affinity? 

When node affinity/anti-affinity will be used to schedule pods in the specified nodes but it does not ensure the best effort to distribute pods evenly across nodes.


### Side effects of having anti-affinity rules? 

Inter-pod affinity and anti-affinity require a substantial amount of processing which can slow down scheduling in large clusters significantly. The docs do not recommend using them in clusters larger than several hundred nodes.

Empty `topologyKey` is interpreted as “all topologies” (“all topologies” here is now limited to the combination of `http://kubernetes.io/hostname`, `http://failure-domain.beta.kubernetes.io/zone`, and `http://failure-domain.beta.kubernetes.io/region`). Each can be used to spread the pods using different constraints. 

### How is it different from taints?

> Taints allow a Node to repel a set of Pods.

So in order for a pod to be scheduled on a node, a pod has to `tolerate` that taint. This comes useful for workloads, where say you only want to schedule pods of a certain type on a set of nodes. A straight away use case is used in the control plane nodes of k8s, where you would not want your workload pods to get scheduled, or say nodes, where you only want to run [prometheus](https://prometheus.io/) dedicatedly, given it can take quite a bit of memory.

### Using a custom scheduler

Albiet I haven't used it, but I feel it's something which you should do only if you have a very good understanding of what's gonna happen if you pluck out your default scheduler from you workloads podspec, as for in most cases, the default scheduler just works fine. Unless you are doing something really out of the usual.


### References

- [https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#inter-pod-affinity-and-anti-affinity](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#inter-pod-affinity-and-anti-affinity)