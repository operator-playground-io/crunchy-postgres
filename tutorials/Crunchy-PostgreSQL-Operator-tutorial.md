---
title: Crunchy Data PostgreSQL Operator Tutorial
description: This tutorial explains how to create a DB using Crunchy PostgreSQL Operator
---

### Introduction

The Crunchy PostgreSQL Operator automates and simplifies deploying and managing open source PostgreSQL clusters on Kubernetes and other Kubernetes-enabled Platforms by providing the essential features you need to keep your PostgreSQL clusters up and running.

### Check the PostgreSQL DB Operator 

```execute
kubectl get pods -n pgo
```

### Create database

For user to create PostgreSQL database Cluster using Crunchy PostgreSQL DB Operator

```execute
cd /home/student/projects/postgres-operator && export PGO_OPERATOR_NAMESPACE=pgo 
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
pgo create cluster my-sample-db --username pguser --password password -n pgo
```

### Setup connectivity to the DB
Create DB Service
```execute
echo "apiVersion: v1
kind: Service
metadata:
  labels:
    name: my-sample-db
    pg-cluster: my-sample-db
    vendor: crunchydata
  name: my-sample-db
  namespace: pgo
spec:
  ports:
  - name: sshd
    nodePort: 30088
    port: 2022
    protocol: TCP
    targetPort: 2022
  - name: postgres
    nodePort: 30445
    port: 5432
    protocol: TCP
    targetPort: 5432
  selector:
    pg-cluster: my-sample-db
    role: master
  sessionAffinity: None
  type: NodePort" | kubectl apply -n pgo -f -
```

Get the Cluster IP
```execute
export ip_addr=$(ifconfig eth1 | grep inet | awk '{print $2}' | cut -f2 -d:)
```

### Connect to DB from the cluster
```execute
PGPASSWORD=password psql -U pguser -h $ip_addr -p 30445 my-sample-db
```





