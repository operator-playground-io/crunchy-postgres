---
title: Crunchy Data PostgreSQL Operator Sample Application Tutorial
description: This tutorial explains how to use a PostgreSQL DB cluster created by the operator in an application.
---

### PostgreSQL DB Example Application :Contacts Application (React/Node.js/PostgreSQL)

***Introduction***

Contacts application comprises of a crunchydata PostgreSQL database , backend and frontend which are deployed independently as a microservice.
The example also uses Skaffold which handles the workflow for building, pushing and deploying your application, allowing you to focus on what matters most: writing code.

***Code Structure***

![codestructure](_images/contacts-app-structure.PNG)

It follows a simple modular and MVC pattern. There are 3 folders that are of our interest:
- k8s :  This contains all the deployment and service yaml for the application. This defines the deployment and exposure of our application.
- frontend: This contains the frontend code and uses Pug as a templating engine for view.
- backend: This contains all the backend code that is building using express js.

### Try the example
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
Create a contacts PostgreSQL DB Cluster 

```execute
pgo create cluster contacts --username pguser --password password -n pgo
```
Check the Contacts DB Cluster state and wait until its in state **1/1**
```execute
kubectl get pods -n pgo | grep "contacts"
```
![check-contacts-db-state](_images/contacts-db-1-1-state.PNG)

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
cd /home/student/projects/edge-node-react-postgres-contacts-deploy/frontend && export backend_port=30456 && sed -i "s|ip|$ip_addr|" .env && sed -i "s|port|$backend_port|" .env
```
Create the Contacts DB PostgreSQL Service
```execute
cd /home/student/projects/edge-node-react-postgres-contacts-deploy/k8s && kubectl create -f contacts-service.yaml
```
Check if the Contacts DB is up and running
```execute
nc -z -v -w30 $ip_addr 30435
```
Create the Contacts DB PostgreSQL Cluster with username and password and initialize the DB.
```execute
cd /home/student/projects/edge-node-react-postgres-contacts-deploy && PGPASSWORD=password psql -U pguser -h $ip_addr -p 30435 contacts < initialize-db.sql 2>output.txt
```
Start the application (Backend and Frontend) with Skaffold
```execute
cd /home/student/projects/edge-node-react-postgres-contacts-deploy&& skaffold config set default-repo localhost:5000 && skaffold run
```

### Access the example application

Click on the Key icon on the Stack Builder Dashboard and copy the value under the `DNS` section and `IP` field

URL :  http://##DNS.ip##:30465

### To Deploy changes to Kubernetes in Dev Mode

Go to Developer Dashboard tab, it will provide you with the IDE along with the integrated terminal.  Click on the bottom status bar and select `TERMINAL`. 

k8s folder contains all the manifest files and defines the deployment strategy for the application.
One can execute them using :

```execute
kubectl apply -f k8s/
```

In this example , we use `Skaffold` which simplifies local development. You can deploy the application is DEV mode which keeps watching for the files changes and on any change, triggers the entire deployment process automatically without the user having to run and manage it manually.

Navigate to the example:

```execute
cd /home/student/projects/edge-node-react-postgres-contacts-deploy
```

```execute
skaffold dev
```

On exiting the command, Skaffold will automatically destroy all the resources it created with above command.


Also, you can use the `skaffold run` to deploy the changes onto Kubernetes as a normal mode. In this mode, the resources created remains unless the user deletes them.

### Clean up the Kubernetes resources (Example application and PostgreSQL DB)

You can delete all the application resources created by executing the following command:

```execute
cd /home/students/projects/edge-node-react-postgres-contacts-deploy && kubectl delete -f k8s/
```

To delete the PostgreSQL DB , execute the below commands:

```execute
cd /home/student/projects/postgres-operator
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
