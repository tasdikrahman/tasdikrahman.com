---
title: "Various ways of enabling canary deployments in kubernetes"
description: "Various ways of enabling canary deployments in kubernetes"
tags: [kubernetes, docker, devops]
comments: true
share: true
cover_image: '/content/images/2018/10/canary-release.png'
---

**Update** I gave a quick lightening talk about the same talk @ DevopsDays India, 2019. The slides for which can be found below

<script async class="speakerdeck-embed" data-id="0b063af43dd54ea794077749a347b58e" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

### What canary can be

Shaping the traffic in a way, so that we could direct a % of traffic to the new pods and promoting the same deployment to a full scaleout and gradually phasing out the older release.

### Why canary?

Testing on staging doesn’t weed out all the possible reasons for something failing, final testing for a feature being done on some part of the traffic is not something unheard of.
Canary being a precursor to enable full blue green deployments. 

#### Why?

If you don’t use feature flags for your services, canary testing becomes paramount to test out features.

#### Approaches to enable canary

##### Using Bare bone deployments in k8s
###### How?

Creation of two sets of deployments and referring to figure 1, v1 and v2, both would be separate deployment objects with separate label selectors. Both v1 and v2 deployments would be exposed via the same svc object which would point to their pods.

###### Advantages
- Plain and simple.
- Can be done without any plugins/extra stuff in the vanilla k8s we get in GKE.

###### Disadvantages

Traffic will be a function of replicas, and cannot be customized. For example, if the traffic splitting is done between the two deployment with v1 having 3 replicas and v2 having 1 replica, the traffic split for canary will be 25%

##### Using Istio
###### How?
In an istio enabled cluster, we need to set the routing rules to configure the traffic distribution.

Similar to the approach above, we have two deployments and svc objects for the same service, called v1 and v2.

The rule will look something like 

```sh
$ kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: helloworld
spec:
  hosts:
    - helloworld
  http:
  - route:
    - destination:
        host: helloworld
        subset: v1
      weight: 90
    - destination:
        host: helloworld
        subset: v2
      weight: 10
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: helloworld
spec:
  host: helloworld
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
EOF
```

###### Advantages
- Has been out there in the wild for quite some time. i.e Battle tested
- [Flagger](https://medium.com/google-cloud/automated-canary-deployments-with-flagger-and-istio-ac747827f9d1) can be added to have automated canary promotion.
- GKE has an add on feature which can be used to install istio in our clusters
- Traffic routing and replica deployment are two completely orthogonal independent functions
- Focused canary testing, eg: instead of exposing the canary to an arbitrary number of users, if you wanted the users from some-company-name.com to the canary version, leaving the other users unaffected, you can do that too with a rule to match the headers for the match to check for the cookie for the above. 

```sh
...
spec:
  hosts:
    - helloworld
  http:
  - match:
    - headers:
        cookie:
          regex: "^(.*?;)?(email=[^;]*@some-company-name.com)(;.*)?$"
...
```
- Tracing gets as a side benefit

###### Disadvantages
- Another add on to manage inside the cluster if gone through the route of installing the istio version

##### Using Linkerd
###### How?

Linkerd has a canary CRD that enabled how a rollout should occur

It automatically creates two sets of deployments for a deployment name `podinfo`

```
NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
podinfo              ClusterIP   10.7.252.86   <none>        9898/TCP   96m
podinfo-canary       ClusterIP   10.7.245.17   <none>        9898/TCP   23m
podinfo-primary      ClusterIP   10.7.249.63   <none>        9898/TCP   23m
```


And then you have to do a rollout to start the traffic shaping to happen to the `podinfo-canary`.
A detailed post on how to do this is [here](https://linkerd.io/2/tasks/canary-release/)
###### Advantages

- Flagger can be integrated for automated canary promotion/demotion
- Battle tested and has been in use by the virtue of being the first service mesh
###### Disadvantages
- Another component inside the k8s cluster to be maintained

##### Using Traefik
###### How?
It requires the same setup of two deployment and svc objects for the service which needs to have canary enabled for it and makes use of the ingress object in k8s to define the traffic split between the services. 

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    traefik.ingress.kubernetes.io/service-weights: |
      my-app: 99%
      my-app-canary: 1%
  name: my-app
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: my-app
          servicePort: 80
        path: /
      - backend:
          serviceName: my-app-canary
          servicePort: 80
        path: /
```
###### Advantages

- Easy to setup with no frills, as it’s just an ingress controller in the k8s controller (like contour/ nginx-ingress-controller)
- Doesn’t need the pods to be scaled.
- Support for tracing included.

###### Disadvantages
- No inbuilt process to shift weights from v1 to v2 or revert back traffic in case of increased error rates. Ie. not a clear cut way to integrate with flagger for automated canary promotion/demotion 

#### References

- [https://martinfowler.com/bliki/CanaryRelease.html](https://martinfowler.com/bliki/CanaryRelease.html)
- [https://martinfowler.com/bliki/FeatureToggle.html](https://martinfowler.com/bliki/FeatureToggle.html)

Canary using istio

- [https://istio.io/blog/2017/0.1-canary/](https://istio.io/blog/2017/0.1-canary/)

Bare bones canary on k8s

- [https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#canary-deployments](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#canary-deployments)

Canary using traefik

- [https://blog.containo.us/canary-releases-with-traefik-on-gke-at-holidaycheck-d3c0928f1e02?gi=c58435e35526](https://blog.containo.us/canary-releases-with-traefik-on-gke-at-holidaycheck-d3c0928f1e02?gi=c58435e35526)
- [https://www.tasdikrahman.com/2018/10/25/canary-deployments-on-AWS-and-kubernetes-using-traefik/](https://www.tasdikrahman.com/2018/10/25/canary-deployments-on-AWS-and-kubernetes-using-traefik/)
- [https://docs.traefik.io/user-guide/kubernetes/#traffic-splitting](https://docs.traefik.io/user-guide/kubernetes/#traffic-splitting)

Canary using linkerd

- [https://linkerd.io/2/tasks/canary-release/](https://linkerd.io/2/tasks/canary-release/)
- [https://linkerd.io/2/features/traffic-split/](https://linkerd.io/2/features/traffic-split/) Done using https://flagger.app/
- [https://www.tarunpothulapati.com/posts/traffic-splitting-linkerd/](https://www.tarunpothulapati.com/posts/traffic-splitting-linkerd/)
Excerpt: “Flagger combines traffic shifting and L7 metrics to do canary deployments, etc. It will slowly increase the weight to the newer version, based on the metrics, and if there is any problem (e.g. failed requests), it would roll back. If not it will continue increasing the weight until all the requests are routed to the newer version. Tools like Flagger can be built on top of SMI and they work on all the meshes that implement it.”
