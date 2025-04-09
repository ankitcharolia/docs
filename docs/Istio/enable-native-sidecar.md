---
layout: default
title:  Enable Native-sidecar in Istio
parent: Istio
nav_order: 3
---

## Kubernetes Native Sidecar
The specific fields for native sidecar containers in Kubernetes (as defined in KEP-753) include:

* `restartPolicy` at the container level (not just at the pod level)
* `terminationGracePeriodSeconds` at the container level

When using the native sidecar feature, you would mark your sidecar containers with these fields. For example:
```yaml
containers:
- name: main-app
  image: main-app-image
  restartPolicy: Always
  terminationGracePeriodSeconds: 300

- name: proxy-sidecar
  image: proxy-image
  restartPolicy: Always
  terminationGracePeriodSeconds: 30

- name: log-collector-sidecar
  image: log-collector-image
  restartPolicy: Always
  terminationGracePeriodSeconds: 30
```

**NOTE:** [https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#differences-from-regular-containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#differences-from-regular-containers)
```yaml
init containers do not support lifecycle, livenessProbe, readinessProbe, or startupProbe whereas sidecar containers support all these probes to control their lifecycle.
```

**The main purpose of these fields is to:**

1. Control the restart behavior of individual containers rather than the entire pod
2. Allow for finer-grained termination control
3. Enable Kubernetes to distinguish between sidecars and the main application container

## Control the termination order of container in the pod.

To control the termination order of containers in a Kubernetes pod, you can use several approaches:

**1. Native sidecar container fields:**

* `terminationGracePeriodSeconds` at the container level allows you to set different grace periods for each container
* `restartPolicy` at the container level helps define container behavior during termination

**2. PreStop hooks**
```yaml
containers:
- name: app
  image: app-image
  lifecycle:
    preStop:
      exec:
        command: ["sh", "-c", "sleep 10"]
```
This delays the termination of specific containers, effectively controlling the order. 

### Example
```yaml
# Container that should terminate first
preStop:
  exec:
    command: ["/bin/sh", "-c", "sleep 10"]
    
# Container that should terminate later
preStop:
  exec:
    command: ["/bin/sh", "-c", "sleep 20"]
```

## SIGTERM with preStop Hook in Kubernetes
When a pod is terminated in Kubernetes, a sequence of events occurs that can be customized using preStop hooks to control the termination process:

**The Termination Sequence**
1. **Initial SIGTERM Signal:** Kubernetes sends a SIGTERM signal to all containers in the pod.
2. **preStop Hook Execution:** Before the SIGTERM signal is delivered to the container's process, any configured preStop hooks are executed.
3. **Grace Period:** After the preStop hook completes, Kubernetes waits for the container to terminate gracefully within the configured `terminationGracePeriodSeconds` (default: 30 seconds).
4. **SIGKILL Signal:** If the container doesn't terminate within the grace period, Kubernetes sends a SIGKILL signal to forcibly terminate it.


### References:
* [Istio Native-sidecar Feature](https://istio.io/latest/blog/2023/native-sidecars/)
* [Enable Native-sidecar in Istio](https://karlstoney.com/moving-to-native-sidecars/)
* [CloudSQL Proxy as Native-sidecar](https://github.com/GoogleCloudPlatform/cloud-sql-proxy/blob/b139a27a9a98e14efb859124ed4c44c8086e4faa/examples/k8s-health-check/proxy_with_http_health_check.yaml)