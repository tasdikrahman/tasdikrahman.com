---
layout: post
title: "ContainerDays 2024, Hamburg"
description: "ContainerDays 2024, Hamburg"
tags: [conferences]
comments: true
share: true
cover_image: ''
---

This was my first time at containerdays, Hamburg and I feel I missed out definitely on the previous ones by not attending!

This year's conference, was a 2 days conference and I met a bunch of familiar faces from KCD Munich, which I attended this year and the last year too.

# Takeaways

## KCP

There was a great talk by Marvin around KCP.

kcp talk - building a platform engineering api layer with kcp

- cncf sandbox project, [https://www.kcp.io/](https://www.kcp.io/)
- project maintainer presenting
    - agenda
        - k8s as an API layer
           -  primarily built to support container orchestration.
           -  CRDs help extend k8s and allow us to use k8s resource model.
           -  the k8s API is pretty awesome.
           - has developed over the last 10 years.
        - lightweight clusters to rescue
           - CRD is cluster scoped.
           - maybe we don’t need the full thing where we just need the API’s of k8s and not orchestration.
           - Hosted Control planes.
              - each cluster has it’s own control plane components
           - What if?
                - we partitioned an API server.
           - Similar to hosted control planes
                - but difference being, only a partition is created in an existing data store, the existing control plane components.
        - what is KCP
            - workspaces
               - multi-tenancy unit of isolation in kcp.
               - each workspaces has it’s own available API resource types.
               - API objects are not shared across workspaces.
               - cheap.
               - delegation of administrative permissions to workspace owners.
               - organised as trees.
        - the API marketplace
            - container orchestration has been removed.
            - API management can be done here since the above is not there.
            - Create APIs with APIExports.
                - a k8s resource model resource,
                - specific to kcp
            - `k get api-resources`
                - would show the workspaces.
            - `k get apibinding`
        - wrapping up.
            - API’s are eating the world.
            - build on top of k8s research and work
            - KCP allows you to use that focus on the implementation.
            - KCP is building a global control plane for API driven platforms.
            - Workspaces allow to mirror organisational hierarchy.

## fireside chat with Kelsey Hightower

When I attended my first kubecon back in 2017, in Austin. I met Kelsey Hightower and it was highly inspiring to meet him back then and it wasn't any different this time around too.

Some thoughts which I was able to scribble down during the fireside chat, where Kelsey was taking questions.

    - How did everything start?
        - best answer is luck.
        - was working since k8s was public with coreos being there.
    - Why k8s started off?
        - MSFT betting big on it, when brendan joined.
        - and they bet on it, which was big coming from a competitor
    - People say k8s is complex, because they don’t understand it.
        - Linux is also complex, but no one says it’s complex explicitly.
    - K8s gave the infrastructure a type system
        - similar to dynamic programming different with static programming languages.
        - the CRD is what separated it out from all the systems.
    - minimalism
        - Do your work at the best level.
    - separation of states
        - LB - apps - databases
        - to just separate the state from the application and upgrade life cycle is easier than.
    - k8s is it going away?
        - mesos didn’t know if it was going away.
        - k8s has a lot of problems, but later on, we might find something which makes easier and builds something new on top.
        - if 10, 20 years goes by, if we are still doing the same thing, that would be not good as that means no progress.
        - someone somewhere is trying to replace this
            - but it needs to be better.
        - Apache2 vs nginx.
          - Lineage will continue, don’t be afraid, embrace it.
        - AI to be used together to successfully integrate with your workflow.
          - but AI to replace everything? Maybe not as of now.

## Open Cluster Management

Some notes on a discussion with someone I had over at the conference about OCM [https://github.com/open-cluster-management-io/ocm](https://github.com/open-cluster-management-io/ocm) and how they are using it.

open cluster management

    - can be used to deploy dependencies to the child clusters from the management cluster.
    - there’s an agent running on the child cluster which talks to the management cluster’s API server.
    - There’s a CRD for managing the cluster information present in the child cluster.
        - but it doesn’t mostly show up the machine pools which you have in the child clusters all the way to the cluster CRD which is there for the child cluster in the management cluster.
        - Is a cncf project which is backed by redhat.


## My Presentation

Last, but not the least, I was able to present my presentation [" How to make pod assignment to thousands of nodes every day easier"](https://www.containerdays.io/containerdays-conference-2024/agenda/).

The excerpt of the talk.

```
New Relic operates tens of thousands of nodes across hundreds of Kubernetes clusters. Pod assignment to these thousands of nodes is done every day, as applications get deployed. I'll share our experience in abstracting out the Kubernetes scheduling primitives from users, discuss their limitations and describe the solution. I'll cover:
* the complexity for end user, in specifying scheduling rules to Kubernetes at scale.
* how we built a scheduling engine by extending Kubernetes via mutating admission webhooks, to translate declarative requirements by user into native Kubernetes scheduling constraints.
* tradeoffs made in the system.
After this talk, attendees will be better prepared to deal with the complexity of extending Kubernetes, to abstract pod assignment to nodes, especially at scale, for end users.
```

Here are the slides and the recording if you would like to go through them.

<script defer class="speakerdeck-embed" data-id="71861a83ee134294b880974e838680e5" data-ratio="1.7772511848341233" src="//speakerdeck.com/assets/embed.js"></script>

<iframe width="560" height="315" src="https://www.youtube.com/embed/BDdzvM83iYc?si=hUgLUDzt6bk4CVn7" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<center><img src="/content/images/2024/09/container-days-picture-2.png"></center>

<center><img src="/content/images/2024/09/container-days-picture-1.png"></center>

## Ending notes

Learned a lot from attending ContainerDays this year and meeting the community, and I would like to attend ContainerDays, along with kubernetes days Munich again next year.

Until next time!
