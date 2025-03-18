---
layout: default
title: Convert Docker Image Scheme Version 1 to Version 2
parent: Docker
nav_order: 2
---

## Why to Convert from Docker Image Schema Version 1 to Schema Version 2?
**1. Deprecation of Schema 1:**
* Schema 1 is legacy and deprecated in modern tools (e.g., Docker 20.10+, containerd v2.0+).
* Newer container runtimes (e.g., containerd, podman) may not support Schema 1.

**2. Multi-Platform Support:**
* Schema 2 supports multi-architecture images (e.g., linux/amd64, linux/arm64) via a manifest list.
* Schema 1 only supports single-platform images.

**3. Security:**
* Schema 2 supports image signing (Notary v2) and vulnerability scanning.
* Schema 1 lacks modern security features like content trust.

**4. Performance:**
* Schema 2 uses content-addressable layers for efficient caching and deduplication.
* Schema 1 layers are identified by hashes of tarballs, leading to redundant downloads.

**5. OCI Compliance:**
* Schema 2 aligns with the Open Container Initiative (OCI) standards, ensuring compatibility with Kubernetes, Helm, and other tools.


## How to Conver Docker Schema Version 1 to Schema Version 2
### Install Skopeo (Option-1) 
* **Skopeo a command line utility that performs various operations on container images and image repositories.**
```shell
# Download skopeo  (https://github.com/containers/skopeo/blob/main/install.md)
# Ubuntu 20.10 and newer
sudo apt-get -y update
sudo apt-get -y install skopeo
```

### Install Skopeo (Option-2)
```shell
# OR use Dockerfile
FROM golang:1.23 AS skopeo-build

WORKDIR /usr/src/skopeo
ARG SKOPEO_VERSION="1.18.0"
RUN curl -fsSL "https://github.com/containers/skopeo/archive/v${SKOPEO_VERSION}.tar.gz" \
  | tar -xzf - --strip-components=1
RUN CGO_ENABLED=0 DISABLE_DOCS=1 make BUILDTAGS=containers_image_openpgp GO_DYN_FLAGS=
RUN ./bin/skopeo --version


FROM scratch AS skopeo-rootfs

COPY --from=skopeo-build /usr/src/skopeo/bin/skopeo /usr/local/bin/
COPY --from=skopeo-build /usr/src/skopeo/default-policy.json /etc/containers/policy.json


FROM ubuntu:24.04

COPY --from=skopeo-rootfs / /
RUN skopeo --version
```

```shell
sudo docker build . --target skopeo-rootfs --output=./skopeo_output
```


### Install crane
* **Crane is a tool for managing container images**
```shell
go install github.com/google/go-containerregistry/cmd/crane@latest
```


### Convert Docker image schema version
```shell
# convert image to v2s2 format, which is supported by containerd v2.0
skopeo copy docker://eu.gcr.io/google_containers/volume-nfs:0.8 docker://europe-west3-docker.pkg.dev/gcp-test/images/volume-nfs:0.9 --format v2s2
```

```shell
# Verify the New Image
crane manifest europe-west3-docker.pkg.dev/gcp-test/images/volume-nfs:0.9 | grep "application/vnd.oci.image.manifest.v1+json"
Ensure "mediaType":"application/vnd.docker.distribution.manifest.v2+json
```

```shell
# conver docker image to OCI format, which is supported by containerd v2.0
skopeo copy docker://eu.gcr.io/google_containers/volume-nfs:0.8 docker://europe-west3-docker.pkg.dev/gcp-test/images/volume-nfs:1.0 --format oci
```

```shell
# Verify the New Image
crane manifest europe-west3-docker.pkg.dev/gcp-test/images/volume-nfs:1.0 | grep mediaType
Ensure "mediaType": "application/vnd.oci.image.manifest.v1+json".
```

**NOTE:** skopeo copy command automatically upload the image to the destination registry
