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
Check the Sample DB Cluster state and wait until its in state **1/1**
```execute
kubectl get pods -n pgo | grep "my-sample-db"
```
![check-my-sample-db-state](_images/my-sample-db-1-1-state.PNG)

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
Check The Service 
```execute
kubectl get svc -n pgo | grep "my-sample-db"
```

Get the Cluster IP
```execute
export ip_addr=$(ifconfig eth1 | grep inet | awk '{print $2}' | cut -f2 -d:)
```
Check the Cluster IP Address
```execute
echo $ip_addr
```

### Connect to DB from the cluster
```execute
PGPASSWORD=password psql -U pguser -h $ip_addr -p 30445 my-sample-db
```
### Create TABLE
The PostgreSQL CREATE TABLE statement is used to create a new table in any of the given database.
```execute
CREATE TABLE COMPANY(
   ID INT PRIMARY KEY     NOT NULL,
   NAME           TEXT    NOT NULL,
   AGE            INT     NOT NULL,
   ADDRESS        CHAR(50),
   SALARY         REAL,
   JOIN_DATE	  DATE
);
```

You can verify if your table has been created successfully using \d command, which will be used to list down all the tables in an attached database.

```execute
\dt 
```
### Insert Values to Table

The PostgreSQL INSERT INTO statement allows one to insert new rows into a table. One can insert a single row at a time or several rows as a result of a query.

The following example inserts a row into the COMPANY table −
```execute
INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY,JOIN_DATE) VALUES (1, 'Paul', 32, 'California', 20000.00,'2001-07-13');
```
The following example is to insert a row; here salary column is omitted and therefore it will have the default value −
```execute
INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,JOIN_DATE) VALUES (2, 'Allen', 25, 'Texas', '2007-12-13');
```
The following example uses the DEFAULT clause for the JOIN_DATE column rather than specifying a value −
```execute
INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY,JOIN_DATE) VALUES (3, 'Teddy', 23, 'Norway', 20000.00, DEFAULT );
```
The following example inserts multiple rows using the multirow VALUES syntax −
```execute
INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY,JOIN_DATE) VALUES (4, 'Mark', 25, 'Rich-Mond ', 65000.00, '2007-12-13' ), (5, 'David', 27, 'Texas', 85000.00, '2007-12-13');
```
All the above statements would create the following records in COMPANY table. The next chapter will teach you how to display all these records from a table.

### Fetch the Data from the Table
PostgreSQL SELECT statement is used to fetch the data from a database table, which returns data in the form of result table. These result tables are called result-sets.

```execute
SELECT * FROM company;
```

### To Exit from the DB

```execute
\q
```
### Run a command remotely 
1. SELECT Command
```execute
PGPASSWORD=password psql -U pguser -h $ip_addr -p 30445 my-sample-db -c "select * from company;"
```
2. INSERT Command
```execute
PGPASSWORD=password psql -U pguser -h $ip_addr -p 30445 my-sample-db -c "INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,JOIN_DATE) VALUES (6, 'Tim', 28, 'Texas', '2009-12-13');"
```



