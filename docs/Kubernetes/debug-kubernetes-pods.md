---
layout: default
title: How To Debug Kubernetes Pods Using Ephemeral Container
parent: Kubernetes
nav_order: 2
---

## What is Ephemeral Container ?

Ephemeral containers are special type of container that runs temporarily in an existing pod to perform some actions such as troubleshooting. This container is never automatically restarted and its lifetime is only up to the life of a pod.

This feature has been included in Kubernetes in v1.22 at alpha stage and in Kubernetes v1.23 as beta feature.

Ephemeral container can be defined inside pod template under .spec.ephemeralContainers with most of fields same as regular containers, but few fields not allowed.

```shell
spec:
  ephemeralContainers:
  - image: busybox:1.28
    imagePullPolicy: IfNotPresent
    name: debugger-qvxjd
    targetContainerName: ephemeral-demo # Container name with which to share process namespace
```
Ephemeral container can be used to debug a pod in either of below cases:

* If the container running inside pod doesn’t have any debugging utilities such as shell.
* If the container has crashed.

## What is the need of Ephemeral Container ?

Copying debugging tools into running containers on-demand with kubectl cp is cumbersome and not always possible (it requires a tar executable in the target container). But even when the debugging tools are available in the container, kubectl exec can be of little help if this container is in a crash loop.


## Inject an ephemeral container to troubleshoot the pod container.
```shell
kubectl debug -it ephemeral-pod-xxxxx --image=alpine --target=ephemeral-target-container
```

```shell
# Explanation
-it: starts a terminal in interactive mode

--image: is the image name and tag to create ephemeral container.

--target: is the container name with which ephemeral container should share the process namespace. This is important because by default a container do not share process namespace with other container inside a pod.
```

Now, Once we are in interactive shell, we can browse all files and directory, run command and see all processes of container with which ephemeral container is sharing process namespace.

### Install some troubleshooting utilities in Ephemeral Container
```shell
apk add --update --no-cache bind-tools sudo
```