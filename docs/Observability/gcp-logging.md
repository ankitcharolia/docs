---
layout: default
title: Important GCP Log Queries
parent: Monitoring
nav_order: 2
---

### Find specific requests those are taking longer than 10s
```shell
resource.labels.container_name="controller"
resource.labels.namespace_name="default"
jsonPayload.duration>10.0
```

### Find app specific logs
```shell
resource.type="k8s_container"
resource.labels.project_id="gcp-test"
resource.labels.location="europe-west3"
resource.labels.cluster_name="gcp-test"
resource.labels.namespace_name="default"
jsonPayload.kubernetes.labels.app="watchdog"
severity>=ERROR
```