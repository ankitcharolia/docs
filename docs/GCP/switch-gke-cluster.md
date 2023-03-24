---
layout: default
title: GKE Cluster Credentials Script
parent: GCP
nav_order: 1
---

### How to Get GKE Cluster Credentials
```shell
#!/bin/bash

REGION="europe-west3"

if [[ $# -ne 3 ]]; then
    echo "Please enter only 3 arguments. Example: switchGKE.sh <gcloud-config-name> <gcloud-project-name> <cluster-name>"
    exit 1
fi

gcloud config configurations activate $1
gcloud config set project $2
gcloud container clusters get-credentials $3 --region $REGION --project $2
```
### How to get gcloud-config-name and gcloud-project-name
```shell
gcloud config configurations list
```

**NOTE:** Execute the script: `./switchGKE.sh test-config test-project test-cluster`
