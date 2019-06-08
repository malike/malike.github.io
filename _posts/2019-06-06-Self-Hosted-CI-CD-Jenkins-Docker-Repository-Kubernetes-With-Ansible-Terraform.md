---
layout: post
comments: true
title:  'Build a Secured Self Hosted Jenkins and Docker Repository for CI/CD for Kubernetes with Docker Notary, Terraform and Ansible'
layout: post
author: "malike_st"
section: sre,devops
tags: [jenkins]
image: dt151220.jpg
---

In previous post I've shown how we can use either [Travis CI]() or [AWS tools]() to build a CI/CD pipeline to package our codes as docker images. This post is slightly different from the other two although the goal is still the same; to build a CI/CD pipeline. How is this one different ?, it is self hosted and it has other steps to ensure the integrity of our images and also ensure all internal policies have been accepted before it get's deployed in Kubernetes.

This post will be broken down in the following steps because readers might interested in different parts of the pipeline and not the whole thing. For those who will try building the whole thing hope this set up helps.

### [1. Setting Up Jenkins]() : Set up a Jenkins Server responsible for building our code and pushing it to our repository

### 2. [Setting Up Docker and Docker Notary]() : Set up a docker repository to host images build by Jenkins Server

### 3. [Configuring CI Pipeline]() : Configure Jenkins to fetch source code from Github and  build successfully

### 4. [Configuring CD Pipeline]() : Configure docker images to be deployed in K8s clusters set up for QA

### 5. [Setting Up Policies to Ensure Integrity of Images Before Deployment]() : Configure policies to ensure all images passed by QA can be deployed on Production K8s cluster.



<br>
<br>
**REFERENCES**
