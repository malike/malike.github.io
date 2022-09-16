---
layout: post
comments: true
title: "Apache Kafka on Kubernetes with Strimzi Operator"
layout: post
author: malike
categories: [distributed,devops,sre]
tags: [k8s,apache-kafka,kafka,kubernetes,fluxcd,gitops]
image:
  path: /posts/restacking.gif
  width: 800
  height: 500
alt: apache kafka on k8s .
---

[Strimzi](https://strimzi.io/) is a kubernetes operator that enables a way to run an Apache Kafka 
cluster on Kubernetes in various deployment configurations with simple configurations. Meaning, we easily manage the lifecycle of our kafka deployment.

The details on how this is done can be found [here](https://strimzi.io/docs/operators/latest/overview.html)
but to summarize:  There are 4 main operators:

**1. Cluster Operator:** Deploys and manages Apache Kafka clusters, Kafka Connect, Kafka MirrorMaker,
Kafka Bridge, Kafka Exporter, Cruise Control, and the Entity Operator

**2. Entity Operator:** Comprises the Topic Operator and User Operator

**3. Topic Operator:** Manages Kafka topics

**4. User Operator:** Manages Kafka users

There are different configuration parameters that can be used for Strimzi as detailed [here](https://strimzi.io/docs/operators/latest/configuring.html)
However, this example will focus on setting up a minimal cluster with minimal configurations and then confirm if we can connect a client to Kafka.


## Kafka configuration with Kubernetes operator pattern

The source can be found [here](https://github.com/malike/kafka-on-k8s-strimzi.git).

I am using FLuxCD to package everything, not just for GitOps, obviously this repo is not set correctly for an end to end GitOps with FluxCD.
There's a Makefile in the repo, which helps organize the how the PoC will be setup.  

First we need to install FluxCD for the crds using the make 

```yaml
---
---
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: strimzi-kafka-poc-minimal-ephemeral
spec:
  kafka:
    version: 3.1.0
    replicas: 1
    listeners:
      - name: internal
        port: 9092
        type: internal
        tls: false
        configuration:
          bootstrap:
            annotations:
            # external-dns.alpha.kubernetes.io/hostname: dummy domain for lb if external
            # external-dns.alpha.kubernetes.io/ttl: "60"
    config:
      offsets.topic.replication.factor: 1
      transaction.state.log.replication.factor: 1
      transaction.state.log.min.isr: 1
      default.replication.factor: 1
      min.insync.replicas: 1
      inter.broker.protocol.version: "3.1"
    storage:
      type: ephemeral
  zookeeper:
    replicas: 3
    storage:
      type: ephemeral
  entityOperator:
    topicOperator: {}
    userOperator: {}
```

3. Confirm everything has been deployed correctly by running `kubectl get kafka  -n strimzi-kafka-poc`.

```commandline
NAME                                  DESIRED KAFKA REPLICAS   DESIRED ZK REPLICAS   READY   WARNINGS
strimzi-kafka-poc-minimal-ephemeral   1                        3                             True
```
This is a confirmation that our Kafka and Zookeeper configuration has been successfully set up
with Strimzi.

Note that, there a various configuration options to use the Strimzi Operator to set up Kafka Clusters
That can be found [here](https://strimzi.io/docs/operators/latest/overview.html)

![Setup](/posts/kafka-on-k8s-strimzi/kafka-k8s-strimzi.png)

## Testing Kafka Stream with Console from Redpanda

Now that the Kafka cluster setup is completed, we would like to write one or more messages in a topic
and confirm if we can consume the messages.
Luckily there is a docker image from Strimzi to help us with this.

Let us create a topic `kafka-topic` by running this command:

```commandline
kubectl -n strimzi-kafka-poc run kafka-producer -ti --image=quay.io/strimzi/kafka:0.23.0-kafka-2.8.0 --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list strimzi-kafka-poc-minimal-ephemeral-kafka-bootstrap.strimzi-kafka-poc.svc.cluster.local:9092 --topic kafka-topic
```

Then in the second terminal we launch a consumer and confirm if we can read messages from the topic `kafka-topic`

```commandline
kubectl -n strimzi-kafka-poc run kafka-consumer -ti --image=quay.io/strimzi/kafka:0.23.0-kafka-2.8.0 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server strimzi-kafka-poc-minimal-ephemeral-kafka-bootstrap.strimzi-kafka-poc.svc.cluster.local:9092 --topic kafka-topic --from-beginning
```


You should see the messages produced consumed successfully. There is also a minimal setup for two clients that can connect to the Kafka.  
The first is [Kafdrop](https://github.com/obsidiandynamics/kafdrop) and the second is [Redpanda console](https://github.com/redpanda-data/console)

![Redpanda Console](/posts/kafka-on-k8s-strimzi/kafka-k8s-strimzi-redpanda-console.png)

![Kafdrop](/posts/kafka-on-k8s-strimzi/kafka-k8s-strimzi-kafdrop.png)


