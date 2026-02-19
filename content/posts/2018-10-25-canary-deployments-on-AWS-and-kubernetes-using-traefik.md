---
title: "Moving Canary deployments on AWS using ELB to kubernetes using Traefik"
description: "Moving Canary deployments on AWS using ELB to kubernetes using Traefik"
tags: [devops, aws, kubernetes]
comments: true
share: true
cover_image: '/content/images/2018/10/canary-release.png'
---

[Canary deployment](https://martinfowler.com/bliki/CanaryRelease.html) pattern is very similar to [Blue green deployments](https://martinfowler.com/bliki/BlueGreenDeployment.html), where you are deploying a certain version of your application to a subset of your application servers. If everything is alright and you have tested out that everything is working fine, you route a certain percentage of your users to those application servers and gradually keep increasing the traffic till a full rollout is achieved. 

One of the many reasons to do this can be to test a certain feature out with a percentage of users who use your service. This can be further extended to enabling a service to users of a particular demographic.

## Canary deployments on AWS

Canary in our use case @ [Razorpay](https://www.razorpay.com), was used by one of our API's which we served and gave out for consumption and the earlier method for canary deployments there before we were on [kubernetes](https://kubernetes.io) was to have two seperate [Autoscaling Groups](https://docs.aws.amazon.com/autoscaling/ec2/userguide/AutoScalingGroup.html) for the primary ASG serving the particular API and another ASG(with a smaller desired count for the ASG), let's call it canary ASG for now. 

Now both
- primary ASG 
- canary ASG

were having their own individual [ELB's](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/what-is-load-balancing.html) attached to them, with them being in our public subnet. Both the ELB's would have a CNAME DNS record pointing to their public FQDN given out by AWS.

<center><img src="/content/images/2018/10/vpc-diagram.png"></center>

For simplicity of drawing the ASG groups, I have not shown the ASG groups for both the canary and the main service in 2 separate AZ's, but it is the recommended way to go forward. As in case of an AZ failure, you have the other set of ASG instances to be routed by the ELB(with cross zone load balancing enabled)

The canary ASG would be attached to both the
- main ELB for the service
- canary's separate ELB

The capacity(min: desired) for the main service is more than the capacity for the ASG for canary, and the canary ASG capacity's max is set to it's desired. The reasoning for this is that, any regression wouldn't propagate to a larger number of users if autoscaling kicks in. 

Since our ELB is an Internet-facing load balancer, it gets public IP addresses(one for each AZ). The DNS name of an Internet-facing load balancer is publicly resolvable to the public IP addresses of the nodes of the ELB. Therefore, Internet-facing load balancers can route requests from clients over the Internet.

The load balancer node that receives the request selects an attached instance using the [round robin routing](https://community.cisco.com/t5/routing/what-is-round-robin-routing/td-p/1400406) algorithm for TCP listeners and the least outstanding requests routing algorithm for HTTP and HTTPS listeners.

Hence, the canary instances would also get traffic in a round robin fashion.

## Replicating the same in kubernetes

[traefik](https://traefik.io/) runs as our L7 load balancer, or as our [ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress/#ingress-controllers) inside kubernetes to route traffic to our [kubernetes services](https://kubernetes.io/docs/concepts/services-networking/service/) for various microservices running inside our cluster. 

traefik would be running on `hostNetwork: true` as [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) 

These pods will use the host network directly and not the “[pod network](https://kubernetes.io/docs/concepts/cluster-administration/networking/)” (the term “pod network” is a little bit misleading as there is no such thing - it basically just comes down to routing network packets and namespaces). So we can bind the Traefik ports on the host interface on port `80`. That also means of course that no further pods of a DaemonSet can use this port and of course also no other services on the worker nodes. But that’s what we want here as Traefik is basically our “external” loadbalancer for our “internal” services - our tunnel to the rest of the internet so to say. 

Sample configuration which you can use to deploy traefik


```yaml
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: traefik-ingress-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
- kind: ServiceAccount
  name: traefik-ingress-controller
  namespace: kube-system
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: traefik-ingress-controller
rules:
  - apiGroups:
      - ""
    resources:
      - pods
      - services
      - endpoints
      - secrets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: traefik
  namespace: kube-system
data:
  traefik-config: |-
    defaultEntryPoints = ["http","https"]
    [entryPoints]
      [entryPoints.http]
      address = ":80"
        [entryPoints.http.redirect]
        regex = "^http://(.*)"
        replacement = "https://$1"
      [entryPoints.https]
      address = ":443"
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
---
kind: Service
apiVersion: v1
metadata:
  name: traefik-ingress-service
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
    - protocol: TCP
      name: http
      port: 80
    - protocol: TCP
      name: admin
      port: 8080
  type: NodePort
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: traefik-ingress-controller
  namespace: traefik
  labels:
    k8s-app: traefik-ingress-lb
spec:
  selector:
    matchLabels:
      k8s-app: traefik-ingress-lb
  updateStrategy:
    rollingUpdate:
      maxUnavailable: 1
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress-lb
        name: traefik-ingress-lb
    spec:
      nodeSelector:
        edge-node-label: ""
      serviceAccountName: traefik-ingress-controller
      terminationGracePeriodSeconds: 60
      hostNetwork: true
      containers:
      - image: traefik:v1.7.16-alpine
        name: traefik-ingress-lb
        ports:
        - name: http
          containerPort: 80
          hostPort: 80
        - name: admin
          containerPort: 8080
        securityContext:
          privileged: true
        args:
        - --loglevel=INFO
        - --web
        - --kubernetes
        - --web.metrics.prometheus
        - --web.metrics.prometheus.buckets=0.1,0.3,1.2,5
        - --configFile=/etc/traefik/traefik.toml
        resources:
          limits:
            cpu: 200m
            memory: 300Mi
          requests:
            cpu: 100m
            memory: 150Mi
        volumeMounts:
        - name: config-volume
          mountPath: /etc/traefik
      volumes:
      - name: config-volume
        configMap:
          name: traefik
          items:
          - key: traefik-config
            path: traefik.toml
```

<center><img src="/content/images/2018/10/canary-k8s-traefik-vpc.png"></center>

The diagram above shows two ASG's for edge nodes, which will host the traefik daemonset(s).

There would be a CNAME DNS record for `myapp.example.com` which points to the public FQDN for the common ELB to which both the edge ASG's are attached to. Traffic would be routed to the edge VM's attached based on a round robin fashion here. Before that, the security groups attached to the ASG's can also be configured to only allow TCP connections on port 80(others would be blocked automatically as it's default deny). 

Similarly a DNS record for canary would be there. 

Traefik would be listening on port 80 on the host's network for incoming requests, and there would be an ingress object in the namespace of the app which would define which service to route the traffic based on the hostname.

We can have an ingress object like the following in the namespace `myapp` for the services
- myapp
- myapp-canary

```
# this feature is available only from traefik version 1.7.0 and upwards
# https://github.com/containous/traefik/releases/tag/v1.7.0
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: traefik-external
    traefik.ingress.kubernetes.io/service-weights: |
      myapp: 90%
      myapp-canary: 10%
  name: myapp-ingress
  namespace: myapp
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - backend:
          serviceName: myapp
          servicePort: 80
        path: /
      - backend:
          serviceName: myapp-canary
          servicePort: 80
        path: /
status:
  loadBalancer: {}
```

This way, traefik would route the traffic coming to `myapp.example.com` to the services
- myapp : 90% of the traffic would be routed here.
- myapp-canary: 10% of the traffic would be routed here.

You can have multiple services to which you can specify the weights and I would only be repeating myself to what has been written here https://docs.traefik.io/user-guide/kubernetes/#traffic-splitting

Another thing to note here is that, the service to which you are trying to do a weighted routing for your canary, should be in the same namespace as the other service(which is `myapp` in this case). This was asked here in their issue tracker and they pointed out the same https://github.com/containous/traefik/issues/4043. 

So you have seen how we can do canary deployments on AWS using traditional ELB's and ASG's as well as if you are on kubernetes with an ingress controller. 

## References

- [https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/how-elastic-load-balancing-works.html](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/how-elastic-load-balancing-works.html)
- [https://martinfowler.com/bliki/CanaryRelease.html](https://martinfowler.com/bliki/CanaryRelease.html)
- [https://github.com/containous/traefik/pull/3112](https://github.com/containous/traefik/pull/3112) PR where weighted traffic was added to traefik
- [https://docs.traefik.io/configuration/backends/kubernetes/](https://docs.traefik.io/configuration/backends/kubernetes/)
- [https://docs.traefik.io/user-guide/kubernetes/#traffic-splitting](https://docs.traefik.io/user-guide/kubernetes/#traffic-splitting)

## Credits

The Network diagrams were made using [draw.io](https://draw.io)
