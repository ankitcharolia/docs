---
layout: default
title: AWS MariaDB to Google MySQL Migration Guide
parent: Database
nav_order: 1
---

## How to migrate AWS MariaDB databse to Google MySQL Cloud SQL

### MariaDB dump from AWS RDS MariaDB
```shell
mysqldump  --column-statistics=0 -h XXXXXXXX.eu-central-1.rds.amazonaws.com -u root -p  db_name > mariadb-dump.sql
```

### Import SQL dump to Google Cloud SQL Instance
```shell
mysql -h 127.0.0.1 -u root -p  db_name < mariadb-dump.sql
```