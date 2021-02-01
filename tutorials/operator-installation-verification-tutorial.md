---
title: PostgreSQL DB Operator Installation Verification Tutorial
description: This tutorial explains how to verify that the PostgreSQL DB Operator installed properly in the namespace
---

### Check the PostgreSQL DB Operator 

Execute the following command to check if the PostgreSQL Operator is running:

```execute
kubectl get pods -n pgo
```

You should see a pod starting with 'postgres-operator' with Ready value '4/4' and Status 'Running' similar to the below output:

```output
NAME                                 READY   STATUS      RESTARTS   AGE
pgo-deploy-qd2q6                     0/1     Completed   0          9m49s
postgres-operator-58f448cd8c-pksrv   4/4     Running     1          8m19s
```

