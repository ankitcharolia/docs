---
layout: default
title: Setup GCP Project
parent: GCP
nav_order: 2
---

### How to Setup GCP Project
```shell
#!/bin/bash
set -euxo pipefail

if [[ $# -ne 1 ]]; then
    echo "Enter the project name of Google Account. Example: configureGCPProject.sh <gcp-project-name> <your-email>"
    echo "Example: ./configureGCPProject.sh test-project ankitcharolia@gmail.com"
    exit 1
fi

gcloud config configurations create $1
gcloud config set account $2
gcloud config set project $1

# Enable Compute API
gcloud services enable compute.googleapis.com                             

gcloud config set compute/zone europe-west3-a
gcloud config set compute/region europe-west3
```
**NOTE:** Execute the script: `./configureGCPProject.sh test-project ankitcharolia@gmail.com`