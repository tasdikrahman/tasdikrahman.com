---
title: "Infrastructure Management at Scale with Cluster API, MachineDeployments and MachinePools"
description: "From Terraform to Cluster API: managing Kubernetes infrastructure at scale across cloud providers"
tags: [cluster-api, kubernetes, terraform, aws, azure, gcp, infrastructure]
comments: true
draft: true
---

I have been managing infrastructure and organisations of different shapes and sizes since early 2017. The toolset has really evolved over time, and different solutions have been developed by organisations along the way. Starting out, I saw a number of places calling cloud provider APIs directly and building CLI tooling on top of them.

## Background: Infrastructure management with Terraform

HashiCorp really gained market share here with Terraform. AWS also had CloudFormation, but across the organisations I have worked with or observed over time, most standardised on Terraform.

Terraform works beautifully if you want a DSL to manage infrastructure across different cloud providers with a module-based approach. There are also tools like Terragrunt that make life easier. I am a big fan of Terraform — I have written and open-sourced some modules over the years, nothing super complicated, but things I have seen people end up using.

Here's ac couple of them I created long back as just a way to learn how to write terraform modules 
- [https://github.com/tasdikrahman/terraform-google-network-subnet](https://github.com/tasdikrahman/terraform-google-network-subnet) GCP : for creation of subnet inside a VPC network.
- [https://github.com/tasdikrahman/terraform-google-network](https://github.com/tasdikrahman/terraform-google-network) GCP : for creation of VPC network 
- [https://github.com/tasdikrahman/terraform-google-network-firewall](https://github.com/tasdikrahman/terraform-google-network-firewall) - GCP : for creation of firewall rules inside the VPC 

A couple of GCP examples for terraform [https://github.com/tasdikrahman/terraform-gcp-examples](https://github.com/tasdikrahman/terraform-gcp-examples), while I was playing around with terraform

There are a couple of limitations, however, that surface quite heavily once you have a large sprawl of infrastructure. If I talk specifically about Kubernetes clusters, once you have anywhere beyond 10 clusters — or more — you start noticing these drawbacks immediately. 

The first is that when your infrastructure state drifts — which happens quite often — it is very hard to know upfront. Someone has to manually run a `terraform plan` to see what has changed. Second, if you are trying to do Kubernetes cluster upgrades (assuming you are using managed Kubernetes releases and nothing custom out of the box), those `terraform plan` and `terraform apply` runs have to wait for the cluster to finish upgrading. The same applies to managed nodes. Your upgrades are effectively serialised and tied to a Terraform run with someone watching over it.

I would be happy to be corrected here if something has changed in the Terraform workflow, but this is what I have seen over the years. The point is: it is not a problem with a handful of clusters, but it becomes a significant source of toil when you are managing hundreds of clusters and thousands of nodes. You end up running the same operations repeatedly across multiple clusters and cloud providers, spending more and more engineering time on it — potentially assigning more engineers to it, or eventually building bespoke automation around it. I have written about this in more depth in these posts:

- [Scaling cluster upgrades for Kubernetes](https://www.tasdikrahman.com/2022/09/26/scaling-cluster-upgrades-for-kubernetes/)
- [A few notes on GKE Kubernetes upgrades](https://www.tasdikrahman.com/2020/07/22/a-few-notes-on-gke-kubernetes-upgrades/)

## Cluster API and its providers ecosystem

Cluster API as a system has been maturing steadily. I have been using it for more than a couple of years now and I firmly believe that if you need to scale your automation beyond a handful of Kubernetes clusters and the lifecycle management that comes with them, Cluster API is a very good fit. I will go into detail on why.

I will not repeat what is already in the upstream docs, but to be brief: Cluster API lets you declare your Kubernetes infrastructure as Kubernetes API objects. This is what opens up the possibility of automation driven entirely through the Kubernetes API.

For example, if you want to create an EKS cluster and manage control plane upgrades, you manage the provider-specific control plane object on your management cluster — through your GitOps tooling, for instance — to track desired state. Your upgrade automation modifies the Kubernetes version field on that control plane object. The Cluster API controller takes over from there with a well-defined reconciliation loop, the control plane gets upgraded, and the version is reflected back on the object status once done. The automation is clean and convergent.

Another way of looking at this convergence is tools like Crossplane. If you want to create resources in different cloud providers, you create a custom resource in your management cluster, and the Crossplane provider controller reconciles it continuously to match the declared state. Compared to the Terraform approach — where infrastructure might drift and you might not know until someone runs a plan — Crossplane and Cluster API will always be working to bring the actual state back to the desired state.

## Machine management at scale: MachineDeployments and MachinePools

For node management in Cluster API, you have two main options.

**MachineDeployments** are a stable, non-experimental API. They are conceptually similar to a Kubernetes `Deployment` but for machines — they manage rolling updates and scaling for a group of machines. MachineDeployments are not backed by a cloud provider's autoscaling primitives; they manage machines directly.

**MachinePools** are backed by the cloud provider's native autoscaling primitives. In AWS that means Auto Scaling Groups; in Azure, Virtual Machine Scale Sets. You get all the benefits those bring — for example, instance refresh is available as a feature with AWS MachinePools. I briefly covered how we use this at our organisation in my talk at kubecon EU London last year. I will slides below showing the setup across a couple of cloud providers.

<iframe class="speakerdeck-iframe" style="border: 0px; background: rgba(0, 0, 0, 0.1) padding-box; margin: 0px; padding: 0px; border-radius: 6px; box-shadow: rgba(0, 0, 0, 0.2) 0px 5px 40px; width: 100%; height: auto; aspect-ratio: 560 / 315;" frameborder="0" src="https://speakerdeck.com/player/353687d1b5e84e6494945dcfa74816e3" title="Resilient Multi-Cloud Strategies: Harnessing Kubernetes, Cluster API and Cell-Based Architecture" allowfullscreen="true" data-ratio="1.7777777777777777"></iframe>

Because both MachineDeployments and MachinePools are backed by Kubernetes APIs, you can write automation around them as a Kubernetes controller — in whichever language you are comfortable with, using the Kubernetes client library for that language. Your upgrade automation becomes a controller that modifies the Kubernetes version on these objects and watches the reconciliation happen. I have seen this model scale very cleanly to tens of thousands of nodes across hundreds of clusters and multiple cloud providers.

The other significant benefit: if you are running clusters across multiple cloud providers, your upgrade automation and Day 2 operational patterns can be very similar regardless of provider, because you are always talking to the same Kubernetes API surface. The biggest win overall is the time saved on those Day 2 operations — upgrades, node rotation, scaling — all driven through Kubernetes APIs rather than bespoke tooling.

It is also worth mentioning the [Gardener project](https://gardener.cloud/), which is not a Kubernetes SIG project but takes a very similar approach — managing your Kubernetes infrastructure through Kubernetes APIs. I have not used it personally, but the convergence of ideas is clear: in both Cluster API and Gardener, you are managing Kubernetes infrastructure as Kubernetes API objects at the end of the day.

## When to use Cluster API — and when not to

That brings me to the conclusion: when does Cluster API and its ecosystem make sense, and when does it not?

I feel it is probably overkill if you have only a handful of Kubernetes clusters, even across different cloud providers — and there is still a real learning curve regardless. If you are at a stage of finding product-market fit, it is definitely overkill.

It starts to make sense, in my view, once you are into double digits in terms of cluster count. You do not even need to be across multiple cloud providers for the benefits to show, but at that scale the automation advantages start paying for themselves clearly.

It also does not make sense if nobody on your team is willing or able to own this. The learning curve is real. If the team has not bought into the tooling, the organisation should go with whatever the engineers have agreed on and are willing to implement and maintain — that is a significant factor and not one to dismiss lightly. That said, it does not change the automation advantages on offer. You should also budget time for the team to ramp up.

Ultimately the decision is down to the team. The tooling that best fits the team's expertise and willingness is the right choice. I hope this piece has made the trade-offs clearer and helps you reason about whether this is the right path for your organisation.

## References

- [Cluster API documentation](https://cluster-api.sigs.k8s.io/)
- [Cluster API MachineDeployments](https://cluster-api.sigs.k8s.io/tasks/deployments.html)
- [Cluster API MachinePools](https://cluster-api.sigs.k8s.io/tasks/machine-pools.html)
- [Cluster API Health Checks](https://cluster-api.sigs.k8s.io/tasks/health-checks.html)
- [AzureMachinePool documentation](https://github.com/kubernetes-sigs/cluster-api-provider-azure/blob/main/docs/machinepools.md)
- [AWSMachinePool documentation](https://github.com/kubernetes-sigs/cluster-api-provider-aws/blob/main/docs/machinepools.md)
- [Crossplane](https://www.crossplane.io/)
