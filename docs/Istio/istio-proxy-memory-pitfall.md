---
layout: default
title:  Envoy Proxy Memory Pitfall in Istio
parent: Istio
nav_order: 5
---
### The issue — excessive memory consumption by Istio proxy sidecars
Envoy proxies in Istio Service Mesh perform all of the traffic management functions for the services in the mesh, such as routing, mTLS, circuit breaking, authorization, retries, etc.
Istio proxy suddenly starts consuming 700MB to 1.2 GB each in the **larger cluster** without changing the applicatin code. Quotas of 1GB per pod were simply not feasible.

### Root cause
* In order to perform its traffic management function, each proxy sidecar needs to be aware of the services ecosystem in the cluster. This information is fed to the proxy sidecars by the “pilot” component of the Istio control plane
* by default, pilot assumes that each proxy could potentially need to route traffic to any service in the cluster, so it goes ahead and pushes metadata about every service in the cluster to each proxy. Proxies hold this metadata in memory. By default, the amount of metadata pushed by the Istio control plane is directly proportional to the number of services deployed in the cluster.

* Use the istioctl tool to find out how much config information the sidecar proxy in a particular pod is loaded with.
```shell
istioctl proxy-config clusters <POD> -n <NAMESPACE>
```
**NOTE:** **Ask AI tools to prepare Sidecar manifest** for specific POD by providing `istioctl pc clusters <POD>` output.

### Simple FIX
* Using the Sidecar custom resource, you can easily limit the **namespaces** for which the Istio control plane will push information to your proxy sidecars.

```shell
apiVersion: networking.istio.io/v1
kind: Sidecar
metadata:
  name: default
  namespace: istio-system
spec:
  egress:
  - hosts:
    - "./*"
    - "istio-system/*"
```

**NOTE:** The above Sidecar limits the scope of the traffic management of Istio proxy sidecars deployed in the cluster to only the services deployed in the same namespace as them, and to services deployed in the istio-system namespace, where the Istio control plane and ingress/egress gateway services are deployed.

### References:
* [Envoy Proxy Memory Pitfall in Istio](https://medium.com/geekculture/watch-out-for-this-istio-proxy-sidecar-memory-pitfall-8dbd99ea7e9d)