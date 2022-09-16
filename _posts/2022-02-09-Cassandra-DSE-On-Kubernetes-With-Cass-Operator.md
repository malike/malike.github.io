---
layout: post
comments: true
title: "Taking DataStax Enterprise for a Spin on Kubernetes"
layout: post
author: malike
categories: [distributed,devops,sre]
tags: [k8s,dse,java,spring-boot,cassandra,kubernetes,stateful]
image:
  path: /posts/dt100125.gif
  width: 800
  height: 500
alt: using datastax enterprise cassandra dse on k8s with sample java app.
---

## 1. Introduction

DataStax Enterprise (DSE) version 6.8 a hybrid cloud, that is can run on-premises or across regions on the cloud to give
all the capabilities of Apache Cassandra with enterprise tooling and expert support. In this particular concept it will
be deployed on AWS. Specifically EKS.

## 2. Setting Up DSE Cluster on EKS

To set up a simple DSE cluster.

* DSE version :  `DSE 6.8.4`
* EKS : `1.21`

a. Download file [https://github.com/malike/dse-on-k8s-with-java.git](https://github.com/malike/dse-on-k8s-with-java.git).

b. `cd` into the `deployment` directory of the repository.

c. Run `make install-cass-operator` to deploy the Cass Operator and also create the namespace `dev-dse-poc`. The Cass
Operator includes resources for the following:

1. **ServiceAccount**, **Role**, and **RoleBinding** to manage permissions necessary to run the operator.
2. **CustomResourceDefinition (CRD)** for the CassandraDatacenter resources used to set up the clusters managed by Cass
   Operator.
3. **Deployment parameters** to ensure the operator runs well.
4. `kubectl -n dev-dse-poc get po` to confirm the operator is running.

```shell
  NAME                             READY   STATUS    RESTARTS   AGE
  cass-operator-848fb7cd47-r5r6m   1/1     Running   0          2h
```

e. Create EBS `StorageClass` resource with the command `make create-ebs-storage`. This will create a StorageClass with
the definition.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: server-storage
provisioner: kubernetes.io/aws-ebs
parameters:
  fsType: ext4
  type: gp2
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

If you're testing on minikube the `StorageClass` will not work for you. You should use something like this below

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: server-storage
provisioner: k8s.io/minikube-hostpath
reclaimPolicy: Delete
volumeBindingMode: Immediate
```


The values can be customized based on need.

Verify the StorageClass is running perfectly.

`kubectl -n dev-dse-poc get storageclass`

```shell
NAME             PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
gp2 (default)    kubernetes.io/aws-ebs   Delete          WaitForFirstConsumer   false                  5d
server-storage   kubernetes.io/aws-ebs   Delete          WaitForFirstConsumer   false                  2h
```

f. Create cluster and datacenters parameters with `make create-data-center`.

```yaml
apiVersion: cassandra.datastax.com/v1beta1
kind: CassandraDatacenter
metadata:
  name: dc1
spec:
  clusterName: cluster2
  serverType: dse
  serverVersion: "6.8.4"
  managementApiAuth:
    insecure: { }
  size: 3
  storageConfig:
    cassandraDataVolumeClaimSpec:
      storageClassName: server-storage
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 5Gi
  config:
    jvm-server-options:
      initial_heap_size: "1G"
      max_heap_size: "1G"
      max_direct_memory: "1G"
      additional-jvm-opts:
        - "-Ddse.system_distributed_replication_dc_names=dc1"
        - "-Ddse.system_distributed_replication_per_dc=3"
``` 

This is a single datacenter with 1 rack, and 3 nodes.

Verify the datacenter is running by running `kubectl -n dev-dse-poc get po`

```shell
NAME                             READY   STATUS    RESTARTS   AGE
cass-operator-848fb7cd47-r5r6m   1/1     Running   0          2h
cluster2-dc1-default-sts-0       2/2     Running   0          2h
cluster2-dc1-default-sts-1       2/2     Running   0          2h
cluster2-dc1-default-sts-2       2/2     Running   0          2h
```

An output similar to this with all 4 pods running confirms a successful installation of DSE. One pod is for the CassOperator, which helps
manage the lifecycle of the DSE cluster, the other 3 pods.

Note that for setting up the DataCenter, racks must have identifiers. The number of racks created can not easily be
changed. The number of racks should match the replication factor in the keyspaces you plan to create.

Define the storage parameters Define the storage with a combination of the previously provisioned storage class and size
parameters. These values inform the storage provisioner how much room to require from the backend.


## 3. Accessing The DSE Cluster with our Sample Java Application

To access the DSE data center we'll need to get the credentials. By default, Cass Operator creates a Cassandra
superuser. A Kubernetes secret is created, named <clusterName>-superuser, which contains username and password keys.
This case our cluster name is `cluster2`.

`kubectl get secret cluster2-superuser -n dev-dse-poc -o yaml`

```yaml
apiVersion: v1
data:
  password: base64-encoded-password
  username: base64-encoded-username
kind: Secret
metadata:
  annotations:
    cassandra.datastax.com/watched-by: '["dev-dse-poc/dc1"]'
  creationTimestamp: "2021-10-12T10:35:28Z"
  labels:
    cassandra.datastax.com/watched: "true"
  name: cluster2-superuser
  namespace: dev-dse-poc
  resourceVersion: "6901797"
  selfLink: /api/v1/namespaces/dev-dse-poc/secrets/cluster2-superuser
  uid: 0befbdd3-df81-4527-a2af-05af64fe0b06
type: Opaque
```

The source code is available here [https://github.com/malike/dse-on-k8s-with-java.git](https://github.com/malike/dse-on-k8s-with-java.git)

Then we deploy by executing the goal `make deploy-app`.

![_config.yml](/posts/dse-on-k8s-with-java/dse-on-k8s-with-java.png)


## 4. Other Tools to manage the Cluster.

There are additional opensource tools to take advantage of to manage our cluster aside from the CassOperator. 
For example: <br/>
i. [management-api-for-apache-cassandra](https://github.com/k8ssandra/management-api-for-apache-cassandra), which provides a sidecar service layer that provides a set of operational actions on Cassandra nodes that can be administered. 
This is in use by the CassOperator as well. <br/>
ii. [reaper-operator](https://github.com/k8ssandra/reaper-operator) which helps to schedule and orchestrate repairs of Apache Cassandra clusters<br/> 
iii. [medusa-operator](https://github.com/k8ssandra/medusa-operator) which helps manage backup/restore capabilities for Apache Cassandra<br/>



