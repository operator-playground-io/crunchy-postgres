```
---
title: Crunchy Data PostgreSQL Operator Tutorial
description: This tutorial explains how to create a DB using Crunchy PostgreSQL Operator
---

### Introduction

The Crunchy PostgreSQL Operator automates and simplifies deploying and managing open source PostgreSQL clusters on Kubernetes and other Kubernetes-enabled Platforms by providing the essential features you need to keep your PostgreSQL clusters up and running.

### Create database

For user to create PostgreSQL database Cluster using Crunchy PostgreSQL DB Operator

```execute
cd /home/student/postgres-operator
```

Install Client Credentials and Download the PGO Binary and Client Certificates:

```execute
PGO_CMD=kubectl ./deploy/install-bootstrap-creds.sh && PGO_CMD=kubectl ./installers/kubectl/client-setup.sh
```

Export PGO data that will be used for Cluster creation

```execute
export PATH=/home/student/.pgo/pgo:$PATH && export PGOUSER=/home/student/.pgo/pgo/pgouser && export PGO_CA_CERT=/home/student/.pgo/pgo/client.crt && export PGO_CLIENT_CERT=/home/student/.pgo/pgo/client.crt && export PGO_CLIENT_KEY=/home/student/.pgo/pgo/client.key && export PGO_APISERVER_URL=https://127.0.0.1:32443
```
Create a PostgreSQL DB Cluster 

```execute
pgo create cluster contacts --username pguser --password password -n pgo
```


### Clean up the Kubernetes resources

To delete the PostgreSQL DB , execute the below commands:

```execute
cd /home/student/postgres-operator
export PGO_OPERATOR_NAMESPACE=pgo
```

```execute
PGO_CMD=kubectl ./deploy/install-bootstrap-creds.sh && PGO_CMD=kubectl ./installers/kubectl/client-setup.sh
```

```execute
export PATH=/home/student/.pgo/pgo:$PATH && export PGOUSER=/home/student/.pgo/pgo/pgouser && export PGO_CA_CERT=/home/student/.pgo/pgo/client.crt && export PGO_CLIENT_CERT=/home/student/.pgo/pgo/client.crt && export PGO_CLIENT_KEY=/home/student/.pgo/pgo/client.key
```

```execute
export PGO_APISERVER_URL=https://127.0.0.1:32443
pgo delete cluster contacts -n pgo
```
