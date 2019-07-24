---
layout: post
comments: true
title: "Developing on Kubernetes Locally with Docker for Mac"
layout: post
author: "malike_st"
section: sre,devops
tags: [kubernetes]
image: dev-env-gilbert.gif
---

Kubernetes is fast becoming, if not already, the _OS_ for building cloud native applications. As someone put it, _"the platform for building platforms"_. As such people will want to replicate the production or a close substitute for delivery during dev and testing. This is to help gain confidence, reduce feedback loop and if possible at a lower cost. Imagine every member of the team having a cluster to play with.

There are many alternatives like [KinD](https://kind.sigs.k8s.io/), which I'll write about in another post,  but this post is focused on [Docker Edge for Mac](). As the name suggests it's purposely for Macs. In this post we get to install and take it for a spin. We'll get to deploy sample applications and get to use some simple commands to manage our clusters.

The installation is pretty simple; if you follow the steps [here](https://docs.docker.com/docker-for-mac/install/). Now we that we have it installed we'll need `kubectl`.

`kubectl` is the kubernetes cli. It helps us run commands against clusters to manage, monitor and deploy application or resource. You can find some of teh cool stuff you can do with it [here](https://kubernetes.io/docs/reference/kubectl/overview/).  If you prefer a web based interface instead for managing your cluster there's an option as well. The Kubernetes dashboard is purposely for that. We can easily install it with `kubectl create -f`  command. This command expects you pass a `yaml` file that describes the resources you want to create and deploy. Fortunately there's a `yaml` file for that [here](https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta1/aio/deploy/recommended.yaml).

`kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta1/aio/deploy/recommended.yaml`

Now let's confirm if our dashboard is installed properly and running with this :

`kubectl get pods --all-namespaces | grep dashboard`

Now that we've confirmed our kubernetes dashboard is running. What does the command mean. What's a pod?. What does the `--all-namespaces` mean? and finally how do we access the dashboard.

 how do we access it. From the `yaml` you can see it's Service resource port is `8443`. We'll delve in _service resource_ later. However the dashboard can't be accessed _externally_ unless it's exposed. To expose the kubernetes dashboard we run this command.

`kubectl port-forward kubernetes-dashboard-5f7b999d65-mjths 8443:8443 --namespace=kube-system`.

To port forward http://localhost:8443 to the kubernetes dashboard also running on port `8443`.
But what does the `--namespace=kube-system` mean ?. 
