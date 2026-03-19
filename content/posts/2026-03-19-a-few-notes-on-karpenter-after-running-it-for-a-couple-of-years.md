---
title: "A few notes on Karpenter after running it for a couple of years"
description: "A couple of tidbits and thoughts on Karpenter after having used it for the last couple of years"
tags: [kubernetes, karpenter, autoscaling, cost-optimisation]
comments: true
share: true
cover_image: ''
---

Having used Karpenter for the last couple of years, I wanted to share a few tidbits and thoughts on how Karpenter itself has evolved.

Without trying to repeat the docs too much, Karpenter is an autoscaler which focuses primarily on cost efficiency and efficient bin packing. It tries to satisfy the requests of your containers deployed in a cluster by fitting the cheapest and most cost-effective nodes to satisfy your applications' requirements.

## Can you run Karpenter alongside Cluster Autoscaler?

The first question that probably comes to mind is whether you can run Karpenter together with Cluster Autoscaler in the same cluster. The answer is yes, and it is entirely possible.

How this is made possible is through specific node labels and taints. One model of operation is that you have a bunch of auto-scaling groups backed by machine pools — potentially if you are using Cluster API and its provider ecosystem for your cloud provider. Each of these machine pools could have taints and specific node labels which your applications would be targeting. At the same time, you would also have multiple Karpenter node pools, where one of them has no taints present on the nodes it creates.

What ends up happening in this scenario when a workload is deployed without any specific affinity constraints and tolerations? It gets scheduled by default to Karpenter, because the nodes coming up from the default Karpenter node pool would be able to satisfy the scheduling requirements of this workload. At the same time, the machine pools you have created would not scale up for this workload, because the kube-scheduler would not see them as a fit — the taint would not be tolerated and the node labels would not be targeted via affinity.

Notice that you can effectively decide what the default landing place of workloads would be in such a setup.

A very clean approach here is to have a Karpenter node pool which has no taints and only has a label present. All workloads which land in this cluster would go to that Karpenter node pool by default. For all your auto-scaling group backed machine pools managed by Cluster Autoscaler, you would add taints so that no workload, unless explicitly configured to tolerate them, gets scheduled there.

The real benefit here is that your default setup allows workloads to directly land on Karpenter and get the benefits of faster autoscaling as well as a wide variety of node shapes — letting Karpenter decide the instance type and family while keeping costs in mind.

<pre class="mermaid">
flowchart TD
    W["Workload\n(no affinity / tolerations set)"]
    W --> S["kube-scheduler"]

    S -->|"✓ scheduled\n(no taint conflict)"| KP_DEF
    S -. "✕ blocked\n(taint not tolerated)" .-> KP_GPU
    S -. "✕ blocked\n(taint not tolerated)" .-> CAS_A
    S -. "✕ blocked\n(taint not tolerated)" .-> CAS_B

    KP_DEF["**Karpenter NodePool** *(default)*\nTaints: none\nLabel: pool=default\nLabel: pool-provider=karpenter\nInstances: any family / type"]
    KP_GPU["**Karpenter NodePool** *(gpu)*\nTaint: team=gpu:NoSchedule\nLabel: pool=gpu\nLabel: pool-provider=karpenter\nInstances: GPU families"]
    CAS_A["**Cluster Autoscaler MachinePool A** *(gpu)*\nTaint: team=gpu:NoSchedule\nLabel: pool=gpu\nLabel: pool-provider=cluster-autoscaler\nInstances: GPU families"]
    CAS_B["**Cluster Autoscaler MachinePool B**\nTaint: team=batch:NoSchedule\nLabel: pool=batch"]

    KP_DEF --> N["Nodes created by Karpenter\n(cost-optimised instance selection)"]

    style KP_DEF fill:#e8f5e9,stroke:#43a047,color:#1b5e20
    style KP_GPU fill:#fce4ec,stroke:#e91e63,color:#880e4f
    style CAS_A  fill:#f3e5f5,stroke:#8e24aa,color:#4a148c
    style CAS_B  fill:#f3e5f5,stroke:#8e24aa,color:#4a148c
    style N      fill:#c8e6c9,stroke:#43a047,color:#1b5e20
    style W      fill:#e3f2fd,stroke:#1e88e5,color:#1565c0
    style S      fill:#fff8e1,stroke:#fb8c00,color:#e65100
</pre>

<script type="module">
  import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
  mermaid.initialize({ startOnLoad: true, theme: 'default' });
</script>

Now if the same workload for example were to be deployed in the GPU workloads, notice that there are two competing places where it can land, either the karpenter pool or the cluster autoscaler backed machinepool. If the workload provides affinity for karpenter node pool by specifying that in the nodeAffinity block for example and also tolerates the taints for this karpenter node pool, along with the specific karpenter node pool label, workload gets scheduled there.

<pre class="mermaid">
flowchart TD
    W["**Workload** *(GPU)*\nToleration: team=gpu:NoSchedule\nnodeAffinity: pool=gpu\nnodeAffinity: pool-provider=karpenter"]
    W --> S["kube-scheduler"]

    S -. "✕ blocked\n(taint not tolerated)" .-> KP_DEF
    S -->|"✓ scheduled\n(toleration matches, affinity matches)"| KP_GPU
    S -. "✕ blocked\n(affinity: pool-provider=karpenter\nnot matched)" .-> CAS_A
    S -. "✕ blocked\n(taint not tolerated)" .-> CAS_B

    KP_DEF["**Karpenter NodePool** *(default)*\nTaints: none\nLabel: pool=default\nLabel: pool-provider=karpenter"]
    KP_GPU["**Karpenter NodePool** *(gpu)*\nTaint: team=gpu:NoSchedule\nLabel: pool=gpu\nLabel: pool-provider=karpenter\nInstances: GPU families"]
    CAS_A["**Cluster Autoscaler MachinePool A** *(gpu)*\nTaint: team=gpu:NoSchedule\nLabel: pool=gpu\nLabel: pool-provider=cluster-autoscaler\nInstances: GPU families"]
    CAS_B["**Cluster Autoscaler MachinePool B**\nTaint: team=batch:NoSchedule\nLabel: pool=batch"]

    KP_GPU --> N["Nodes created by Karpenter\n(GPU instance families)"]

    style KP_GPU fill:#fce4ec,stroke:#e91e63,color:#880e4f
    style CAS_A  fill:#f3e5f5,stroke:#8e24aa,color:#4a148c
    style KP_DEF fill:#e8f5e9,stroke:#43a047,color:#1b5e20
    style CAS_B  fill:#f3e5f5,stroke:#8e24aa,color:#4a148c
    style N      fill:#c8e6c9,stroke:#43a047,color:#1b5e20
    style W      fill:#e3f2fd,stroke:#1e88e5,color:#1565c0
    style S      fill:#fff8e1,stroke:#fb8c00,color:#e65100
</pre>

If you were to now schedule the workload in the cluster autoscaler backed machinepool, you can do the same by giving the nodeAffinity for the cluster autoscaler backed pool. Notice here that the machinepool only can be created with homogeneous types of machines, which could be seen as more inflexible as compared to a Karpenter node pool. Where the node pool has flexibility for a lot of instance types, machine types and also architectures.

<pre class="mermaid">
flowchart LR
    subgraph KP["Karpenter NodePool (gpu)"]
        direction TB
        KP_META["Taint: team=gpu:NoSchedule\nLabel: pool=gpu\nLabel: pool-provider=karpenter"]
        KP_META --> KP_I["Instance types allowed"]
        KP_I --> M5["m5.xlarge"]
        KP_I --> M5_4["m5.4xlarge"]
        KP_I --> M6I["m6i.xlarge"]
        KP_I --> M6I_4["m6i.4xlarge"]
        KP_I --> MORE["... and more instance families / types"]
    end

    subgraph CAS["Cluster Autoscaler MachinePool A (gpu)"]
        direction TB
        CAS_META["Taint: team=gpu:NoSchedule\nLabel: pool=gpu\nLabel: pool-provider=cluster-autoscaler"]
        CAS_META --> CAS_I["Instance family (fixed)"]
        CAS_I --> M5_ONLY["m5.xlarge only"]
    end

    style KP     fill:#fce4ec,stroke:#e91e63,color:#880e4f
    style CAS    fill:#f3e5f5,stroke:#8e24aa,color:#4a148c
    style KP_META fill:#fce4ec,stroke:#e91e63,color:#880e4f
    style CAS_META fill:#f3e5f5,stroke:#8e24aa,color:#4a148c
    style KP_I   fill:#fff,stroke:#e91e63,color:#333
    style CAS_I  fill:#fff,stroke:#8e24aa,color:#333
    style M5     fill:#fff,stroke:#ccc,color:#333
    style M5_4   fill:#fff,stroke:#ccc,color:#333
    style M6I    fill:#fff,stroke:#ccc,color:#333
    style M6I_4  fill:#fff,stroke:#ccc,color:#333
    style MORE   fill:#fffde7,stroke:#ccc,color:#555
    style M5_ONLY fill:#fff,stroke:#ccc,color:#333
</pre>

## Tuning node pool configuration

There is a fair amount of tuning possible inside the node pool configuration. If you only want to bring up nodes in a specific availability zone, that is also configurable directly in the node pool spec.

<pre class="mermaid">
flowchart LR
    NP["**NodePool:** default"]

    NP --> Z["**Zone constraint**\ntopology.kubernetes.io/zone\noperator: In\nvalues: us-west-2a"]
    NP --> A["**Architecture**\nkubernetes.io/arch\noperator: In\nvalues: arm64, amd64"]

    Z --> ZN["Nodes brought up\nonly in us-west-2a"]
    A --> AA["arm64 *(e.g. Graviton)*"]
    A --> AB["amd64 *(e.g. m5, m6i)*"]

    style NP  fill:#e3f2fd,stroke:#1e88e5,color:#1565c0
    style Z   fill:#e8f5e9,stroke:#43a047,color:#1b5e20
    style A   fill:#e8f5e9,stroke:#43a047,color:#1b5e20
    style ZN  fill:#fff,stroke:#43a047,color:#333
    style AA  fill:#fff,stroke:#43a047,color:#333
    style AB  fill:#fff,stroke:#43a047,color:#333
</pre>

## Node disruption budgets

From v1.0 onwards, Karpenter introduced the concept of node disruption budgets. The default is 10%, and this is now configurable at the node pool level based on how much disruption you want to tolerate. You could tshirt size your karpenter pools for example, based on the disruption budget for each pool that you require.

<pre class="mermaid">
flowchart TD
    PARENT["**Karpenter Node Pools**"]

    PARENT --> LOW
    PARENT --> MED
    PARENT --> HIGH

    subgraph LOW["karpenter-pool-less-disruption"]
        direction TB
        L_CP["consolidationPolicy:\nWhenEmptyOrUnderutilized"]
        L_CA["consolidateAfter: 1m"]
        L_B1["budget: 5%\n(always)"]
        L_B2["budget: 0%\nmon–fri 09:00 for 8h"]
        L_CP --> L_CA --> L_B1 --> L_B2
    end

    subgraph MED["karpenter-pool-medium-disruption"]
        direction TB
        M_CP["consolidationPolicy:\nWhenEmptyOrUnderutilized"]
        M_CA["consolidateAfter: 1m"]
        M_B1["budget: 15%\n(always)"]
        M_B2["budget: 0%\nmon–fri 09:00 for 8h"]
        M_CP --> M_CA --> M_B1 --> M_B2
    end

    subgraph HIGH["karpenter-pool-high-disruption"]
        direction TB
        H_CP["consolidationPolicy:\nWhenEmptyOrUnderutilized"]
        H_CA["consolidateAfter: 1m"]
        H_B1["budget: 30%\n(always)"]
        H_B2["budget: 0%\nmon–fri 09:00 for 8h"]
        H_CP --> H_CA --> H_B1 --> H_B2
    end

    style PARENT fill:#e3f2fd,stroke:#1e88e5,color:#1565c0
    style LOW  fill:#e8f5e9,stroke:#43a047,color:#1b5e20
    style MED  fill:#fff8e1,stroke:#fb8c00,color:#e65100
    style HIGH fill:#fce4ec,stroke:#e53935,color:#b71c1c
    style L_CP fill:#fff,stroke:#43a047,color:#333
    style L_CA fill:#fff,stroke:#43a047,color:#333
    style L_B1 fill:#fff,stroke:#43a047,color:#333
    style L_B2 fill:#fff,stroke:#43a047,color:#333
    style M_CP fill:#fff,stroke:#fb8c00,color:#333
    style M_CA fill:#fff,stroke:#fb8c00,color:#333
    style M_B1 fill:#fff,stroke:#fb8c00,color:#333
    style M_B2 fill:#fff,stroke:#fb8c00,color:#333
    style H_CP fill:#fff,stroke:#e53935,color:#333
    style H_CA fill:#fff,stroke:#e53935,color:#333
    style H_B1 fill:#fff,stroke:#e53935,color:#333
    style H_B2 fill:#fff,stroke:#e53935,color:#333
</pre>

## The `karpenter.sh/do-not-disrupt` annotation

Another useful automation is injecting the `karpenter.sh/do-not-disrupt` annotation on workloads like batch jobs, which are sensitive to restarts. You would not want Karpenter to disrupt the node where such a workload is running.

Here is an example of a Kubernetes Job running on a Karpenter node pool node with the annotation set:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: long-running-batch-job
  namespace: default
spec:
  template:
    metadata:
      annotations:
        karpenter.sh/do-not-disrupt: "true"
    spec:
      tolerations:
        - key: "team"
          operator: "Equal"
          value: "batch"
          effect: "NoSchedule"
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: pool
                    operator: In
                    values:
                      - batch
                  - key: pool-provider
                    operator: In
                    values:
                      - karpenter
      restartPolicy: Never
      containers:
        - name: batch-worker
          image: my-batch-worker:latest
```

With this annotation on the pod, Karpenter will not drain the node this job lands on for the duration of its run, ensuring the job completes without being interrupted.

Using this annotation thoughtfully is essential for Karpenter to function normally in a cluster. Imagine a cluster where a large number of workloads have this annotation set — Karpenter would simply not be able to disrupt any of those nodes, leading to no drain operations and ultimately defeating the purpose of having Karpenter as the autoscaler in the first place.

## Single node pool vs. multiple node pools

Another common question is whether to have multiple Karpenter node pools or a single large one. There are pros and cons to both approaches.

Multiple node pools give you the flexibility to have different disruption budget knobs — different t-shirt sizes, if you will. For example, one node pool with a 5% disruption budget, the default at 10%, and others at 20% or even 30%. There are obviously other configuration knobs inside the node pool beyond disruption budgets, but you get the general idea.

That said, with segregation of workloads into different node pools, bin packing efficiency is not fully utilised. If instead you had one single large node pool where all Karpenter-targeting workloads would go, you would get the maximum bin packing efficiency out of Karpenter across all of them.

## Pod Disruption Budgets

Karpenter also respects pod disruption budgets specified for applications. If disrupting a node would violate a workload's PDB, Karpenter will not disrupt it.

## Cloud provider support

**EKS:** Karpenter has very mature support on EKS. Even if you are managing nodes yourself inside an EKS cluster, you can run Karpenter and that configuration is supported.

**Azure:** At this point in time, the Karpenter provider for Azure is only available when using AKS, the managed Kubernetes service. If you are self-managing Kubernetes clusters on Azure — for example with a kubeadm-based setup — Karpenter would not work for you there. I had opened [an issue](https://github.com/Azure/karpenter-provider-azure/issues/323) to check on the support for this.

**Cluster API:** There is also an upcoming project https://github.com/kubernetes-sigs/karpenter-provider-cluster-api — the Cluster API provider for Karpenter — which is still early but would be an option for running Karpenter on self-managed Kubernetes clusters and using karpenter as the autoscaler for CAPI. 

## All in all

Karpenter has matured quite a bit from the time of the v1alpha APIs through v1beta1 and now to the v1 API. It provides great cost optimisation benefits, given its ability to select from a large set of instance families and types to satisfy the compute requirements of your applications. The advantages are clear, and it is vastly superior to having to manage multiple auto-scaling groups for different applications.

If teams can work with a large node pool of varied instance families and types, and their applications specify exactly what they need, the result is maximum flexibility with endless configuration options — as opposed to every team coming to you to create or request an auto-scaling group. The difference in flexibility is pretty clear.

Another post on the v1beta1 to v1 API migration is coming soon.

## References

- [Karpenter](https://karpenter.sh)
- [Karpenter NodePools](https://karpenter.sh/docs/concepts/nodepools/)
- [Cluster API MachinePools](https://cluster-api.sigs.k8s.io/tasks/experimental-features/machine-pools)
- [Pod Disruption Budgets](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/#pod-disruption-budgets)
- [Karpenter Disruption](https://karpenter.sh/docs/concepts/disruption/)
