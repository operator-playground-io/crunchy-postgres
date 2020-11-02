```
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
cd /home/student/postgres-operator && export PGO_OPERATOR_NAMESPACE=pgo 
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
pgo create cluster my-db --username pguser --password password -n pgo
```

### Try an Example 

Get the Cluster IP
```execute
export ip_addr=$(ifconfig eth1 | grep inet | awk '{print $2}' | cut -f2 -d:)
```
Get the Example 
```execute
cd /home/student/projects && git clone https://github.com/Andi-Cirjaliu/edge-node-react-postgres-contacts-deploy.git
```
Setup the Backend API for deployment
```execute
cd /home/student/projects/edge-node-react-postgres-contacts-deploy/frontend && export backend_port=30456 && sed -i \"s|ip|$ip_addr|\" .env && sed -i \"s|port|$backend_port|\" .env
```
Create the Contacts DB PostgreSQL Service
```execute
cd /home/student/projects/edge-node-react-postgres-contacts-deploy/k8s && kubectl create -f contacts-service.yaml
```
```execute
until nc -z -v -w30 $ip_addr 30435; do echo \"Waiting for Contacts database connection...\"; sleep 5; done;
```

Create the Contacts DB PostgreSQL Cluster with username and password and initialize the DB.
```execute
cd /home/student/projects/edge-node-react-postgres-contacts-deploy && PGPASSWORD=password psql -U pguser -h $ip_addr -p 30435 contacts < initialize-db.sql 2>output.txt
```
Start the application (Backend and Frontend) with Skaffold
```execute
cd /home/student/projects/edge-node-react-postgres-contacts-deploy&& skaffold config set default-repo localhost:5000 && skaffold run
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



