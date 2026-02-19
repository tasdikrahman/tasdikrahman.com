---
title: "Route missing in kubernetes node with kuberouter as the CNI"
description: "Route missing in kubernetes node with kuberouter as the CNI"
tags: [kubernetes, docker, devops]
comments: true
share: true
cover_image: '/content/images/2020/01/kuberouter-logo-full.svg'
---

Anyone who is evaluating into having a networking solution for their kubernetes cluster without having a lot of moving parts in the cluster, [kuberouter](https://www.kube-router.io/) provides [pod networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/), ability to enforce [network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/), IPVS/LVS service proxy among other things.

The problem which we faced specifically while running this in our clusters was missing routes upon restart of the node, or sometimes in the case when the node was joining the cluster as part of the worker node.

For us, the issue would come around as a the [kiam](https://github.com/uswitch/kiam) (which we were using for identity management for pods inside the k8s clusters) pod would go into [CrashLoopBackOff](https://stackoverflow.com/questions/44702715/kubernetes-pod-fails-with-crashloopbackoff) as described by me in the github issue [https://github.com/uswitch/kiam/issues/49](https://github.com/uswitch/kiam/issues/49), as the dns resolution would fail (more on that later)

We were using the latest version of [Coreos](http://coreos.com/), but we found out that the version 1576.5.0 of Coreos was not plagued by this problem.

This has been defined in detail in the [github issue](https://github.com/cloudnativelabs/kube-router/issues/370)

The problem was that there was race condition caused by [systemd-networkd.service](https://www.freedesktop.org/software/systemd/man/systemd-networkd.service.html) trying to manager the tunnels and modigying the routes causing the missing routes. Whenever networkd was  restarting, all the tunnels would go away with it.

It is best described by [Niel here](https://github.com/cloudnativelabs/kube-router/issues/370#issuecomment-399850110).

To fix this, a file in the networkd dir `/etc/systemd/network/` so it starts ignoring those interfaces and doesn't manage them as described by [Lomkju](https://github.com/cloudnativelabs/kube-router/issues/370#issuecomment-463967949)

```
[Match]
Name=tun* kube-bridge kube-dummy-if

[Link]
Unmanaged=yes
```

It was tested to be working for the following coreos version as mentioned by [Lomkju](https://github.com/cloudnativelabs/kube-router/issues/370#issuecomment-463967949)

```
Container Linux by CoreOS 1967.6.0 (Rhyolite)
Kernel: 4.14.96-coreos-r1
```
