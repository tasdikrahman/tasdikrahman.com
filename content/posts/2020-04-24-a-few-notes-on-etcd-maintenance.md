---
title: "A Few Notes on Etcd Maintenance"
description: "Learnings on provisioning, compaction, defragmentation, and more"
tags: [databases, devops, etcd]
comments: true
share: true
cover_image: '/content/images/2020/04/etcd-header.jpeg'
---

> This was originally published under [Gojek's engineering blog](https://blog.gojekengineering.com/a-few-notes-on-etcd-maintenance-c06440011cbe) by me, this post is a repost.

If you have worked around managing [Kubernetes](https://kubernetes.io/) clusters on your infrastructure — instead of going with a managed version provided by cloud providers — chances are that you already are managing an [etcd](https://etcd.io/) cluster. In case you are new to it, this post is for you.

We’ll get the basics out of the way first, and define what Etcd is.

> Etcd, is just a distributed key-value store, which uses [raft](https://raft.github.io/) consensus algorithm in the back of it, to provide a fully replicated, highly available key-value store.

A telling point of its stability is the fact that Kubernetes (API server) uses it as it’s key-value store for storing state of the whole cluster. It uses Etcd’s ‘watch’ function to monitor this data and to reconfigure itself when changes occur. The ‘watch’ function stores values representing the actual and ideal state of the cluster and can initiate a response when they diverge.

As to choosing Etcd over other databases like Redis, Zookeeper or Consul, I feel that it’s out of scope for this blog post considering there a lot of posts out there which go about detailing this. This post will try listing down a few things about the maintenance activities which we are currently running for our production Etcd databases, to keep them healthy.

### Monitoring

One of the first things which you can do is, adding a few basic alerts which will serve as the foundation for our maintenance activities.

<script src="https://gist.github.com/tasdikrahman/26ac5d37a86a9360d4c361226308017b.js"></script>

I will be only repeating myself here if I list down any more alert rules, as the official alert docs [here](https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/etcd3_alert.rules.yml) mention most of the important alerts. The one rule to notice apart from leader election and process being down is the DB size. This rule is not mentioned in the official rules doc, as this is something which people tune based on their DB size.

The metrics emitted by Etcd are in [Prometheus](https://prometheus.io/) format. A few things on our side which we add while emitting is to add the environment label, depending on which environment the ETCD VM is running, as you can see in the alert above. You can remove that while trying it out on Alertmanager.

To avoid running out of space for writes, the keyspace history has to be compacted. Once reached, this would be obvious from the errors received by your client using Etcd.

Track `etcd_mvcc_db_total_size_in_bytes`, as etcd also emits `etcd_debugging_mvcc_db_total_size_in_bytes_gauge` in some versions. The reason to track the former, is that it can get dropped in later releases. It’s not a bad idea, to make sure to not track anything with *debugging* with it in the metric name


### Compaction

As the space quota is limited, one would need to clear out the keyspace history, which can be achieved through compaction.

> Compaction truncates the per-key change log.

Using one of these auto-compaction modes is usually recommended:

For periodic compactions, pass `--auto-compaction-retention` to the Etcd process while starting, eg: `--auto-compaction-retention=1` would run compaction every one hour. The mode picked up here would be periodic, this is similar to passing `--auto-compaction-mode=periodic`

The other mode of compaction is revision-based, which is similar to passing `--auto-compaction-mode=revision`. We don’t use this as the use case for us is having a large keyspace rather than having a huge number of revisions for a key-value pair.

There’s a single revision counter which starts at 0 on etcd. Each change made to the keys and their values in the database is manifested as a new revision, created as the latest version of that key and with the incremented revision counter’s value associated with it.

For revision-based auto compaction mode, the behaviour should ideally be similar to when one runs something like `$ etcdctl compact 4`, after which the revisions prior to the compaction revision version become unavailable. So in this case, if you would do a `$ etcdctl get --rev=3 some-key` would fail in this case.

If you need more headspace, one can pass `--quota-backend-bytes` with the desired space to set space quota, while starting up the Etcd process. The default is 2GB, but you can max it up till [8GB](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.2.0/manage_cluster/manage_etcd_clusters.html). It is ideal to keep this size lower.


### Defragmentation

Compaction is not enough, as the internal DB exhibits fragmentation after compaction, leaving gaps in the backend database, which would still cause disk space to be consumed. This is fixed by running defragmentation on the Etcd DB.

> Defragmentation operation removes the free space holes from storage

Combining compaction and defragmentation, along with the right set of monitoring can build the base for maintenance of your etcd cluster/node.

One thing to note here is that defragmentation should be run rather infrequently, as there is always going to be an unavoidable pause. Defragmentation to a live member blocks the system from reading and writing data while rebuilding its state.

This is it is recommended to keep the DB size smaller — the default again being 2GB. The larger the DB size being consumed, the more time it will take to defragment. Depending on how critical Etcd is to your app, you can take this into consideration.

### Would an HA setup help here?

An [HA](https://en.wikipedia.org/wiki/High_availability) Etcd setup won’t help in reducing the pause while defragmenting, as Etcd leader re-election will only happen when the defrag takes longer time than the leader election timeout, which shouldn’t happen if your DB is below 2GB and if compaction runs regularly.

In a clustered setup, defrag can be run per node, or can be directly run on the whole cluster by running `$ etcdctl defrag --cluster` to do it for all members of the cluster.

### Running defrag operations

Personally, I prefer a systemd.timer to run the defrag operation, compared to a traditional cronjob, as one would get the logs by default for each and every trigger by passing the timer service to journalctl, which is far more intuitive to someone logging onto the Etcd box.

<script src="https://gist.github.com/tasdikrahman/d064d35d734d802e80a20e5138c839b9.js"></script>

<script src="https://gist.github.com/tasdikrahman/b450236232c048dd9aa0d1f173d7db18.js"></script>

Using the above two systemd services, you can run the defrag operations based on when you would want it to run.

<script src="https://gist.github.com/tasdikrahman/d2236b54c4d17f1df37cdfbdebc042a2.js"></script>

### Provisioning Etcd

We use the [Etcd community cookbook](https://github.com/chef-cookbooks/etcd/) to provision the Etcd servers, and haven’t noticed any issues so far.

<script src="https://gist.github.com/tasdikrahman/6eccff66192b81e6f264391caa7bdb9f.js"></script>

The sample systemd service is for a single node Etcd database. Depending on if you require an Etcd cluster or don’t want etcdv2 API to be available — among other things — the parameters to your `ExecStart` will change. (This is well-documented in the cookbook repo).

We haven’t tried running Etcd on Kubernetes. Although statefulSets are something slowly gaining traction, we didn’t feel running this on top of Kubernetes was something we wanted to do. That being said, we have heard good things about [etcd-operator](https://github.com/coreos/etcd-operator), although the project has been archived.

### So how do we automate provision of Etcd?

We do it via a [proctor script automation](https://github.com/gojek/proctor), paired with a chef cookbook on top of [https://github.com/chef-cookbooks/etcd](https://github.com/chef-cookbooks/etcd) with a few extra things, to create the defragmentation related systemd services.

### A few optimizations you can do

It is preferable to have the storage device attached to the Etcd VM to be in the same network, as Etcd is extremely dependent on storage latency, and some operations are blocked until most nodes have accepted a change. If latency passes certain critical numbers, one can end up with leader election storms, where no leader keeps the lease for long enough to actually make changes. It’s preferable to never use remote block storage for Etcd, as there are just too many places it could go wrong in weird ways.

Layering multiple layers of fault tolerance that aren’t aware of each other (in this case the remote storage) might lead you to end up with no fault tolerance at all.

Since Etcd is highly I/O dependant, it would make sense to have an SSD as the disk type attached to the ETCD instance, as this is something which is very critical to have lower disk write latency.

Sizing your VM type depending on your workload is crucial to not have performance issues (more discussion [here](https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/hardware.md)).


### References

- [https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/maintenance.md](https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/maintenance.md)
- [https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/etcd3_alert.rules.yml](https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/etcd3_alert.rules.yml)
- [https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/hardware.md](https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/hardware.md)
- [https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/etcd3_alert.rules.yml](https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/etcd3_alert.rules.yml)
- [https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/grafana.json](https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/grafana.json)
- [https://freedesktop.org/software/systemd/man/systemd.timer.html](https://freedesktop.org/software/systemd/man/systemd.timer.html)

Thanks a ton to [@youngnick](https://twitter.com/youngnick) for being super active on #etcd channel on Kubernetes Slack and helping us dig deeper, to [Shubham](https://twitter.com/Shubham5830) for seeing this through with [me](https://twitter.com/@tasdikrahman), and my teammates for sticking with us.
