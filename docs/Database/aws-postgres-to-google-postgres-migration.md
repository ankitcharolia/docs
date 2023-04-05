---
layout: default
title: AWS Postgres to Google Cloud-SQL Postgres Migration Guide
parent: Database
nav_order: 2
---

## How to migrate AWS Postgres to Google Postgres Cloud-SQL Instance

### Postgres dump from AWS RDS Instance
```shell
pg_dump -c -O -h db-rds-xxxxxx.xxxxxxx.eu-central-1.rds.amazonaws.com -p 5432 -U CmsProd -d CmsProd > cms-prod.sql
```

### connect to Google cloud-sql using cloud-sql-proxy (In Terminal-1)
```shell
acharolia@ankitcharolia:~$ ./cloud-sql-proxy <project-name>:<project-region>:<database-instance-name>
2023/04/05 15:57:43 Authorizing with Application Default Credentials
2023/04/05 15:57:44 [<project-name>:<project-region>:<database-instance-name>] Listening on 127.0.0.1:5432
2023/04/05 15:57:44 The proxy has started successfully and is ready for new connections!
```

### Restore Postgres dump to Google Cloud SQL Instance (In Terminal-2)
```shell
psql -U cms-prod -d CmsPev -h 127.0.0.1 -v --no-owner < cms-prod.sql
```