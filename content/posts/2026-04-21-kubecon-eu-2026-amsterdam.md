---
title: "KubeCon EU 2026 - Amsterdam"
date: 2026-04-21
draft: false
tags: ["kubernetes", "kubecon", "devops", "cloud-native", "opentelemetry", "prometheus"]
categories: ["conferences"]
description: "Notes from KubeCon EU 2026 in Amsterdam, covering talks on cluster migrations, CAPI, OpenTelemetry, Prometheus 3.0, and more."
---

# KubeCon EU 2026 - Amsterdam

A few weeks ago, I attended CubeCon EU in Amsterdam in person. This was my third CubeCon EU in succession from the last few years, the first one being, CubeCon Paris, then CubeCon London.

This time. We had around 13,000 attendees meeting in person.

I'll try summarizing the talks on each day I attended and the notes from the same.

Putting my notes as is as I put them down while tending the talks

## Day 1

### Airbnb - Migration of 1000 Services from Regional Clusters to Zonal Clusters

- deployment platform
    - one touch system
    - DSL which devs use, deploy job, kubernetes-gen and then CD system
    - different environments are then promoted over time.
- cells and cellsets: The compute abstraction
    - cellset - collection of cells
        - cell - 1: cluster1, cluster2…. cluster-n
    - Applications would run in a cell set.
- so what they moved over was from a regional deployment to a cellset.
    - their deployment platform would then deploy to the cellsets.
- automating the service migration
    - best time to migrate would be when they actually do the deployment.
    - As they deploy the application
    - steps would be to first scale up the cellular deployments of the application being deployed
    - then each cell would be deployed gradually over course of time and the regional deployment would be scaled down (each regional cell would have multiple clusters deployed)
    - migration would be batched.
        - automated pull requests.
        - git clone - > config migrator -> create PR -> create slack thread.
        - Different tiers for migration of services - tier 0, tier1 and publishing a timeline for the same.
            - for example, they migrated everything on testing, staging for example first.
            - pre-prod: canary: would be then migrated next
            - and then only after all stages were migrated, then we would go to production for the same.
        - support requests from teams.
            - lots of requests from teams which would ask about the migration progress.
        - They were also tracking the migration
            - starting the year, they crossed half of the migration by mid of the year and the last set of services, they would be taking it over at the very end of the migration, so end of the year.
        - 3000 migration PR’s, 95% of the migration is done, they were deleting the regional clusters also slowly.
        - The whole point of this was to do traffic shifting easier.
            - so they would shift the traffic inside the cells across different zones.
    - They are moving into a format where the SRE team will do the whole migration for you.


### Celonis - Migrating Away from 1.16 in 2025

- why?
    - 70% of all large scale migrations fail
- 6 k8s flavors
- they wanted to reduce the number of flavors to 3, for each cloud provider.
- but fear was that they would have more flavors than before, but they avoided that.
- k8s setup history
    - How did they accumulate this much?
    - environments - full installation of the celonis platform
        - 1 env - multiple clusters
        - tenancy - multi tenant clusters and also the single tenant cluster.
        - data locality is also something important - customer requirements
        - ~160 clusters as of 2026
    - 2017
        - kops was used for staging, production
        - gardener was used in 2018 and then for azure
    - 2021
        - openshift was used.
        - AWS and Azure
        - custom operator and security focus.
    - 2023
        - target platform development
        - problems with current platform
            - high cognitive load
            - making changes is complicated and slow
            - newest features are not supported.
        - one flavor per cloud provider
            - EKS, AKS and GKE for cloud provider.
            - Modern, secure.
        - multi cloud support was required.
        - no application changes should not be required.
        - features
            - argocd appsets
            - cluster upgrades was given priority
            - karpenter as a feature.
        - Cluster fleet
            - in a cloud provider region - aws-us-east-1
            - shared networking
                - cluster main
                - cluster ML
                - cluster query processor
        - wormhole.
            - cross cluster service mesh.
        - Why?
            - workloads can be migrated independently
            - workloads can seamlessly talk to each other across clusters
            - off the shelf solutions not feasible.
        - How?
            - they deploy the svc to the new cluster.
            - the svc selector in the original cluster is then modified to point to the new cluster.
        - migration plan
            - stateless apps
                - scale up
                - cut traffic over
                - scale down
                - Migration plan
                    - duplicate the kustomize overlap
            - singletons
                - similar procedure
            - stateful apps
                - some ~9% of the apps, they had to do one of things for a couple of applications.
            - the tooling would create PRs for apps which would be reviewable and then would be able to be reviewed by everyone.
    - The plan was to consolidate the platform spread.
    - Lessons
        - Don’t change too many things at once.
            - application refactoring shouldn’t be something which is to be done.
        - Beware of scope creep
        - Snowflakes will slow you down
        - The key to scaling is repeatability.
        - Iterate and improve.



### OpenTelemetry Project Updates

- go metrics sdk performance
    - 4x to 30x performance improvements.
    - changes on most metric instruments are available today.
    - exponential histograms and attributes coming soon
- span events deprecation
    - the span event API is getting deprecated in habit of log based events
    - span events will be phased out over time
- stability
    - cross wide project to improve stability
    - obi progress
        - RC in next few months, aligning with declarative config and improving installation and docs.
    - profiling goes alpha
        - the profiling signal will be alpha on OTLP v1.11.0. Already available on the collector
    - collector stability
        - renewed focus on most used components, learn more in the next session on this room.
        - https://opentelemetry.io/blog/2026/kubecon-eu/
    - get involved
        - blueprints
        - zig sig
        - otel for beginners
        - browser sig
        - kotlin sig
        - php distro
        - ecosystem explorer
        - https://opentelemetry.io/blog/2025/otel-injector/



### Spotify Model of Resource Management Based Model of Operation

spotify model of resource management based model of operation
        - feature teams and platform teams both use the product
        - provide feature teams with less complexity and product teams with flexibility
        - managing 1M resources at scale
            - look at this talk later which was given in kubecon 2025
        - case study
            - spotify has lots of data.
            - 750M active users
            - every user generates a lot of data.
            - 28% of data is duplicated from two sources.
            - biglake tables to point to underlying data in GCS is used to solve this problem.
        - Interlude using cloud bigtable
            - low latency key value store
        - kro.run - platform developer can focus on stiching together the resources together.
            - Instead of the kubebulder setup
            - The underlying resources on different cloud providers are then also orchestrated in order
            - https://kro.run
            - https://kro.run/docs/overview
        - kpop - k8s protobuf operators
            - from spotify



## Day 2

### Cluster API In Place Updates

I personally was looking forward to attending this talk in person, having worked quite a bit around cluster API and
it's providers, this one was particularly interesting for me. 

cluster API in place updates
no human intervention during in place updates
- https://kubernetes.io/blog/2026/01/27/cluster-api-v1-12-release/
- cluster API
    - update extensions
        - perform in place updates
    - update extensions should perform only carefully validate and repeatable in place updates
    - cluster API does not have opinions on how updates should be perform
- In place updates for control plane machines
    - immutable control plane rollouts
        - replicas: 3
            - maxSurge: 1 (maxUnavailable: 0)
                - because maxUnavailable is 0 cluster API ensures that there are always 3 available replicas
            - maxSurge: 0 (maxUnavailable: 1)
                - because maxUnavailable is 1 cluster API only ensures that there are always 2 available replicas
    - in place updates
        - what is possible?
            - there is more than 1 replica
            - the control plane is in a healthy state.
        - replicas: 3, maxSurge: 1 (maxUnavailable: 0)
            - because maxUnavailable is 0 cluster API ensures that there are always 3 available replicas.
            - so we create a 4th machine, and then do in place updates on the 3 replicas, one by one and then delete the 4th replica.
- in place updates for worker machines
    - immutable machinedeployments rollouts
        - machineset - the replica set is the similarity
    - so when we create a new machine deployment spec, the machineset a new one gets created.
    - maxunavailable: 0
        - defines the speed at which the old machine is getting rid of
        - defines how much disruption cluster API should tolerate at any time
    - maxSurge: 1
        - defines the speed at which the new machineset is getting added
        - how much additional machines CAPI can create during rollouts
    - MD rollouts with in place updates
        - what is possible?
            - rollout strategy is rollingUpdate.
        - replicas: 3, maxSurge: 1, maxUnavailable: 0
            - so we create a 4th machine,
            - on replicaset 1, we do in place update and then we update the owner ref for that machine to be in the newer machineset which was created.
            - so, process is update in place -> update machineset ownership for that machine.
        - replicas: 3, maxSurge:0, maxUnavailable: 1
            - we tell CAPI to disrupt more
            - so what we do is, we do a move and then update in place and so on.
    - Skip worker rollouts when upgrading multiple minors
        - we can now go multiple versions of the upgrade now.
            - earlier only one version at a time was possible.
        - eg: 1.31.0 -> 1.34.0 is updated by user.
        - CAPI will take care of the whole update flow from 1.31 till 1.34 of the control plane nodes and the worker machines
        - We can also optimise further on how to do the upgrades for worker nodes all the time.
            - For example, we can
    - The sweet spot between immutable and mutable infrastructure
        - immutability is and always will be the core of CAPI
        - CAPI will continue to make immutable rollouts faster
    - two cents from CAPI maintainers
        - in place updates should not be used as a workaround to avoid fixing foundational problems in your platform
            - if your app cannot handle drain nicely you must look into it and try fixing it.
            - Drain is a foundational primitive of k8s trying to avoid drain is fighting k8s
        - if your infra is slow, you must look into it and try to improve speed.
            - you cannot avoid creating or deleting machines
        - in place updates are best suited for rolling out changes that don’t require node drain or pod restarts
            - if the application will be disrupted anyways, just do an immutable rollout.
            - for eg: rollout for a new ssh key is okay for in place updates.
            - not okay to do in place updates: eg: upgrade kernel

### Improving Pod Disruption and Node Lifecycle

Improving pod disruption and node life cycle - 14:35 pm
— eviction request ideas/roadmap
 - configurable heartbeat deadline
- cancellation policy
- unification of termination mechanisms
- eviction types
- pre-emotion support
- kubelet soft eviction support
- integration and adoption in the ecosystem
- dynamic responders
- new target types, podgroup workloads

life cycle management
- improve lifecycle management of the the node
    - node maintenance
    - graceful shutdown
- declarative node maintenance
- kubectl cordon and kubectl drain
- lots of tools
    - no standardised way to implement a solution
- there’s a lot of thoughts on how to do drain
- ecosystem is fragmented
    - aws-node-terminatio-handler
    - karpenter
    - cluster-autoscaler
    - kured
    - draino
    - kubevirt
    - maintenance-operator
    - cluster-api
- approach
    - observability
        - specialised lifecycle management
            - create a standardised declarative API for co-ordinations lifecycle management
    - meaningful states
        - eg: draining
        - what are the meaningful states of the drain operation, for eg: what is are we draining in the node

            - https://github.com/kubernetes/enhancements/pull/5769
    - ecosystem adoption
- kubelet forgets it’s doing a graceful shutdown

kubelet starts workloads on a node that is currently in process of ...

Slides : https://hosted-files.sched.co/kccnceu2026/a7/The%20Future%20of%20Kubernetes%20Node%20Lifecycle%20-%20Google%20Slides.pdf?_gl=1*w2weem*_gcl_au*MTEwMTE0OTk3OC4xNzc0NTEzMTA5*FPAU*MzE2ODkxOTUuMTc3NDUxMzEwMQ..

Slides: https://hosted-files.sched.co/kccnceu2026/f3/Improving%20Pod%20Disruption%20and%20Node%20Lifecycle.pdf?_gl=1*1phc9ru*_gcl_au*MTEwMTE0OTk3OC4xNzc0NTEzMTA5*FPAU*MzE2ODkxOTUuMTc3NDUxMzEwMQ..

Pod group/ task group new primitives in k8s


### OpenAPI Meets Kubernetes - Autogenerating CRDs and Operators the Smart Way - MongoDB

- what are we trying to solve
    - k8s is not a workload scheduler
    - used as a foundation to build platforms
    - used to connect external systems
    - api centric use cases becoming more relevant
    - the classic operator pattern here doesn’t work well
- operator types
    - workload operators
        - run something inside k8s
            - stateful for stateless workloads
            - databases
    - configuration operators
        - extend what the cluster can do
            - certificates
            - policies, RBAC
            - ingress rules
    - external resource operators
        - connect k8s to other systems
        - mongodb atlas clusters
        - aws s3 systems
        - external dns records
        - atlas k8s operator
            - provision mongodb resource on atlas
        - ACK
            - aws controllers for k8s
        - azure service operator
        - crossplane providers
        - prior art
            - terraform wrapping
            - eg: crossplane/upjet
            - padok-team/burrito
            - pros:
                - leverage existing terraform providers
            - cons:
                - terraform providers are not tailored for kube
                - you must have a terraform provider
                - feels heavyweight, eg: state file management
        - prior art - api to api translation
            - fybrik/openapi2crd
            - ant31/crd-validation
            - pros:
                - lightweight
                - leverage existing API’s
            - cons:
                - cannot leverage existing providers
        - so how do you implement one
            - download controller-runtime
            - sin up kind
            - warm up vscode
            - Reconcile()
        - why manual doesn’t scale
            - manual toil
                - every new atlas feature implies hand crafted CRDs, controllers etc.
            - feature lag
                - k9s support trails behind the atlas API releases
            - inconsistent UX
                - human in the loop development leads to inconsistencies across resources
            - IDP integration demand
                - platforms like crossplane/backstage
        - from manual to automated
            - core: an automated pipeline transforms openAPI specs into fully functional k8s resources
            - the paradigm shift
                - old way: go code - > CRD (kubebuilder)
                - new way: open api - > CRD -> go code + controllers
            - benefits
                - eliminates the curated abstraction layer
                - achieves 1:1 alignment of openapi
            - automation pipeline
                - openapi spec is taken by openapi2crd -> generates the CRD
                - crd2go -> CRD yaml -> then this takes the yaml to go structs
        - universal state machine
            - all possible states are pre-defined
            - resources can use all or a subset of states.
        - crd2go
            - takes the crd yaml and then will give go types.
            - features
                - smart type naming (WIP)
                - custom renames and reservations
                - adhoc imports
                - kubebuilder covers deep copy generation
                - plugins for extra code generation
                - optional apply side configuration annotations
        - x-kubernetes-preserve-unknown-fields: true without a declared openapi type which created an error
        - ^ this was faced by me before, but it’s not something which is recommended to use as you could put whatever state inside.


### GitLab Automation - Building a Platform for K8s Which Scales from Self Managed to SaaS

- open standards ensure both internal and self managed experiences share one foundation
- key takeaways
  - vendored solutions are a genuine trade off, not automatically a mistake.
  - revisit trade off when business value changes
  - open standards help avoid vendor lock in for customers
  - k9s as an API enables a common architectural layer
  - a shared arch streamlines internal and customer tooling

## Day 3

### Uber Talk on K8s Lifecycle Management

https://hosted-files.sched.co/kccnceu2026/88/scaling-k8s-ecosystem.pdf


### K8s Readiness Controller - Deterministic Ready State - Digital Ocean

- node problem detector
- node readiness controller would check the checks based on the node conditions and doesn’t do any of it’s own checks
- https://kubernetes.io/blog/2026/02/03/introducing-node-readiness-controller/
- https://github.com/kubernetes-sigs/node-readiness-controller


### Prometheus 3.0 Updates

- experimental promql
  - better outer joins with fill(), fill_left(), fill_right()
  - access timestamps via ts_of_(first|….)_over_time
  - duration arithmetics: rate (http_requests_total[10m+2s]
  - type and unit labels {__type__="historgram", __unit__="seconds"}
  - new keywords
    - smoothed
- plenty of optimisations & fixes
  - tsdb - 25.5%
  - promql - 19.1%
  - scraping - 12.8%
- experimental start timestamp storage
  - gauge semantics
  - coming in 3.11 prom.
  - what's next
    - histograms
    - promql support
- experimental XOR2 encoding
  - xor2 chunk encoding
  - --enable-feature=xor2-encoding
  - cannot be then read by older prometheus versions that do not support the encoding. Once enabled and data is written, you need to manually delete blocks from the disk, otherwise prom will return error on all queries.
- what's baking (new features which need help, you can join)
  - delta temporality
    - main source - OTLP service
    - OTLP compatibility
    - Improved monitoring of edge, server less, batch jobs, browser, mobile. etc.
  - in prometheus/proposals
  - parquet storage in future
    - accelerating thanks at scale - talk by vinted team
  - metadata storage
    - pr/17740
  - towards composite sample storage
    - compatible query
    - scalability
  - classic histograms (explicit)
    - native histograms
    - native histograms with custom buckets
  - open metrics 2.0 (create.0)
- what else?
  - optional metric (otel) schematisation #18349, was a talk on kubecon EU 2026
  - piped promql + variables (#17487)
  - scrape memory limits (PROM-76)
- community achievements
  - governance 2.0 merged (opened by gouthamve)
  - promcon2025, munich happened.
  - promcon.io for future plans
  - 3.0 rush, was maxed out 2024 for it.
  - alert manager
    - now we have maintainers for alertmanager
  - AI has entered the chat
    - the project feels the pressure of it like many other projects
    - use AI tools, responsibly, they do too.
    - create personal trust
    - disclose AI use
    - demonstrate insight
    - stick around, triage, review.

### How Telemetry Data Moves in a Real System

- by eduardo - creator and maintainer of fluentbit
- this is about
  - how data moves
  - how it is buffered
  - how it's processed at scale
- core path
  - concepts
  - input/sources -> processors -> outputs/destinations.
- OS 101
  - kernel and user space
    - unsafe - user space - applications
    - safe - kernel space - hardware - system calls
    - application - system call -> kernel space -> hardware -> hardware -> kernel space -> system calls -> user space
    - data moves from user space to system space using system calls
  - our resources
    - hardware
    - cpu cores
    - memory
    - disk/storage
- data -> analysis
  - multiple sources of data -> data analysis
  - source of data A -> environment OS -> network hardware -> data analysis
- data normalisation
  - source of data A -> format A
  - source of data B -> format B
  - ….
  - format A, format B -> data parsing and serialisation (encoding) -> message pack.
- why message pack?
  - a binary json which has been around for 12-13 years
  - json: 27 bytes
    - {"compact": true, "Schema": 0}
  - message pack: 18 bytes
    - 82 A7 compact C3 A6 schema 00
  - protobuf is similar
  - but message pack is schemaless
  - but protobuf is with schema
- fluent bit chunk, a message pack
  - 4 byte header
  - TAG
  - data
- moving data out of the box
  - input/sources -> processors -> outputs/destinations
- sending data
  - multiple system calls are expensive
    - chunk -> write() -> system calls (N)
    - chunk -> write() -> system calls (N)
- data movement can be expensive
  - multiple system calls are expensive
  - so we buffer
    - chunk1, chunk2, chunk3… -> write() -> system calls (1)
- threading
  - take advantage of your architecture
    - thread 1 - main event loop (where we are doing buffering)
      - scheduler can schedule the data on when it is to be sent.
      - retries
      - filters
    - thread 2:
      - read data
      - encode events
    - thread 3
      - output plugin/destination
      - decode events
      - format
      - deliver
- event loops
  - an event - something that happens
    - in the event loop, you can implement asynchronous io
    - if I am trying to reach out from one to another machine
    - and if the machine is not reachable, they let the other know if they are able to reach back.
    - coroutine - light weight user space thread.
- event loops + co-routines


### Reducing Wasteful Telemetry

- weaver from otel
  - Enforce service.name rules before they get broadcasted.
  - A sdk wrapper
  - Wrapper sdk on top of otel sdk
  - Best is to do it in application side
  - Create wrappers around otel
  - https://github.com/open-telemetry/weaver
- commodities like auto instrumentation that expedite transition from day 0 to day 1 may hurt you on day 2 if used irresponsibly
- Use collector level attribute filtering and PII masking
- Use auto instrumentation responsibly
- Use weaver sdk wrappers and agentic instrumentation review for telemetry governance

## Ending notes

I had a great time meeting folks from the community. This time I was able to also meet again in person a lot of 
cluster API and its providers maintainers, and committers. Great conversations overall with them and also with some 
of my friends who had traveled back from far. Meeting them was great. In general, I think KubeCon always is amazing 
as an experience to attend. Looking forward to the next time in Barcelona in 2027. 
