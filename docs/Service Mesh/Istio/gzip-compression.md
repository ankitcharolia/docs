---
layout: default
title:  Activate Gzip Compression On The Ingress-Gateway
parent: Service Mesh
nav_order: 1
---

## How to activate gzip compression on the ingress-gateway

```shell
#  activate gzip compression
---
apiVersion: networking.istio.io/v1alpha3
kind: EnvoyFilter
metadata:
  name: ingressgateway-gzip
  namespace: istio-ingress
spec:
  workloadSelector:
    labels:
      istio: ingressgateway
  configPatches:
    - applyTo: HTTP_FILTER
      match:
        context: GATEWAY
        listener:
          filterChain:
            filter:
              name: "envoy.http_connection_manager"
              subFilter:
                name: 'envoy.router'
      patch:
        operation: INSERT_BEFORE
        value:
          name: envoy.gzip
          config:
            remove_accept_encoding_header: true
            compression_level: BEST
```

To verify, dump the config of one of the ingress gateway pods and grep for gzip.
```shell
kubectl -n istio-ingress exec -it $(kubectl -n istio-ingress get pod -l istio=ingressgateway -o jsonpath='{.items[0].metadata.name}') -- curl -s localhost:15000/config_dump | grep gzip
```
Next, request a service through the ingress gateway and set the accept-encoding header with gzip,deflate:

```shell
curl -H "accept-encoding: gzip,deflate" ...
```
