---
layout: default
title:  Minimizing Cross-Zone Traffic Charges with Istio
parent: Istio
nav_order: 2
---

### Zone-Aware Routing Solution
There were two features available to achieve this:

* Locality Load Balancing in Istio for services with Istio.
* Topology Aware Routing for services using Kubernetes’ Kube-Proxy.

Istio is a service mesh for managing and securing microservices. The choice between Istio’s Locality Load Balancing and Kubernetes’ Topology Aware Routing is determined by whether the service uses Istio. If the Pod communicating has an Istio sidecar, then Istio’s Locality Load Balancing will be utilized. If the Pod does not have an Istio sidecar, then Kubernetes’ Topology Aware Routing will be used.

ISTIO Mesh-wide config: Locality based load balancing distribution or failover settings. If unspecified, locality based load balancing will be **enabled by default.** However, this requires **outlierDetection** to actually take effect for a particular service, [**see this**](https://istio.io/latest/docs/tasks/traffic-management/locality-load-balancing/failover/) 

* [CLICK HERE](https://istio.io/latest/docs/reference/config/istio.mesh.v1alpha1/#MeshConfig-locality_lb_setting)


```shell
apiVersion: networking.istio.io/v1
kind: DestinationRule
metadata:
  name: echo
spec:
  host: echo.sample.svc.cluster.local
  trafficPolicy:
    loadBalancer:
      # simple: ROUND_ROBIN/LEAST_REQUEST (Istio will select default appropriate algorithm, if not specified)
      # This option allows us to provide session affinity based on the HTTP headers
      # consistentHash:
        # httpHeaderName: x-user
      localityLbSetting:
        enabled: true
        failoverPriority:
         - "topology.kubernetes.io/region"
         - "topology.kubernetes.io/zone"
    outlierDetection:
      # configure based on usual 5xx error rate of service
      consecutive5xxErrors: 10
      # configure based on the time taken to run up a new Pod usually.
      interval: 5m
      # configure based on HPA target utilization (default 10%)
      maxEjectionPercent: 15
      # configure based on HPA target utilization
      baseEjectionTime: 10m   
```

### References:
* [Reduce Cross-zone traffic using Istio](https://tetrate.io/blog/minimizing-cross-zone-traffic-charges-with-istio/)
* [Reduce inter-zone egress cost](https://engineering.mercari.com/en/blog/entry/20231027-reducing-inter-zone-egress-costs-with-zone-aware-routing-in-mercaris-kubernetes-clusters/)
