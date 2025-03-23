---
layout: default
title: Important Prometheus Query to debug in k8s environments
parent: Observability
nav_order: 1
---

## Important Prometheus Query to debug in k8s environments
**highest amount of traffic received by the pod**
```shell
topk(10, sum(rate(container_network_receive_bytes_total{namespace="default"}[5m])) by (pod, namespace) / 1024 / 1024)
```

**highest request received by pod**
```shell
topk(10, sum(rate(nginx_http_requests_total{namespace="default"}[5m])) by (helm_sh_chart))
```

**99th percentile latency for each pod in seconds. This is often used to identify how long the slowest requests take (typically for performance bottlenecks)**
```shell
topk(10, histogram_quantile(0.99, sum(rate(nginx_http_request_duration_seconds_bucket[5m])) by (le, pod, namespace)))
```

**request hit to varnish**
```shell
topk(15, sum(rate(varnish_main_client_req{namespace="default"}[5m])) by (pod))
```

**Aggregates the CPU usage per pod (in cores)**
```shell
# top 10 pod consuming highest CPU
topk(10, sum(rate(container_cpu_usage_seconds_total{container!="POD", namespace!=""}[5m])) by (pod))
# CPU usage per container
sum(rate(container_cpu_usage_seconds_total{namespace="default", pod=~"nginx-.*"}[5m])) by (container)
```

**Top 10 containers by Memory Usage (in MiB)**
```shell
# This shows the memory usage per container within each pod, which gives you a more granular breakdown. same graph will be shown twice (once for pod and once for pod and container)
topk(10, sum(container_memory_working_set_bytes{container!="POD", namespace!=""}) by (pod, container) / 1024 / 1024)
# Memory usage per container
sum(container_memory_working_set_bytes{namespace="monitoring", pod="prometheus-kube-prometheus-stack-prometheus-0"}) by (container) / 1024 / 1024
```

**Top 10 Containers by CPU Throttling**
```shell
# Some containers (especially sidecars like logging, monitoring, or proxies) may not have enough CPU allocated.
topk(10, sum(rate(container_cpu_cfs_throttled_periods_total{container!="POD", namespace!=""}[5m])) by (pod, container, namespace))
```

**Check Memory Limits Being Hit**
```shell
topk(10, sum(container_memory_rss{container!="POD", namespace!=""}) by (container, pod, namespace) / 1024 / 1024)
```

**Check if CPU/Memory Requests and Limits Are Being Respected**

* Pods Using More CPU Than Their Requests
```shell
topk(10, sum(rate(container_cpu_usage_seconds_total{container!="POD", namespace!=""}[5m])) by (pod, namespace) / sum(kube_pod_container_resource_requests_cpu_cores{namespace!=""}) by (pod, namespace))
```
📌 Value > 1 means the container is using more CPU than it requested (i.e., the container is consuming extra CPU beyond its request, potentially leading to throttling if the system has insufficient CPU resources).
📌 Value < 1 means the container is using less CPU than it requested (i.e., the container is underutilizing its allocated resources).


* Pods Using More Memory Than Their Requests
```shell
topk(10, sum(container_memory_working_set_bytes{container!="POD", namespace!=""}) by (pod, namespace) / sum(kube_pod_container_resource_requests_memory_bytes{namespace!=""}) by (pod, namespace))
```
📌 If the value is >1, the container is using more memory than requested, leading to potential OOMKills.