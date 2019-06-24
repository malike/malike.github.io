---
layout: post
comments: true
title: "Running Kubernetes Locally via Docker for Mac"
layout: post
author: "malike_st"
section: sre,devops
tags: [kubernetes]
image: dev-env-gilbert.gif
---

Kubernetes is fast becoming, if not already, the _OS_ for building cloud native applications. As someone put it, _"the platform for building platforms"_. As such people will want to replicate the production or a close substitute for delivery during dev and testing. This is to help gain confidence, reduce feedback loop and if possible at a lower cost. Imagine every member of the team having a cluster to play with.

There are many alternatives like [KinD](https://kind.sigs.k8s.io/), which I'll write about in another post,  but this post is focused on Docker Edge for Mac. As the name suggests it's purposely for Macs. In this post I'll take you through how to install and also take it for a spin. We'll get to deploy sample applications and get to use some simple commands to manage our clusters.


