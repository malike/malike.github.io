---
layout: post
comments: true
title: "Continuous Integration, continuous Deployment with AWS Using EKS, CodeBuild, CodePipeline, ECR and CloudFormation"
layout: post
author: malike
categories: [devops,sre]
tags: [distributed,codepipeline,codedeploy,codecommit,codebuild,cloudformation,microservice,k8s,aws,eks]
image:
  path: /posts/n_1_sample_size.jpg
  width: 800
  height: 500
  alt: aws eks.
---

Building up on our previous [article](https://malike.github.io/K8s-101-And-Istio-Service-Mesh.html), where a simple cloud native/ distributed system application was built to run on kubernetes using Travis CI, DockerHub locally. What if we move this infrastructure to the cloud, AWS for this instance. Using AWS tools to accomplish the same thing and deploy also test **AWS EKS**(AWS's Kubernetes offering).

This will include setting up a CI/CD pipeline and setting up the infrastructure to host the application on staging and production.

This article will be treated in 2 parts:

*a. Part One : CI/CD Pipeline*

*b. Part Two : Set up of infrastructure for deployment*

But we before we delve in, let me talk about  **AWS CloudFormation**.

### [CloudFormation](https://aws.amazon.com/cloudformation/)

_"AWS CloudFormation provides a common language for you to describe and provision all the infrastructure resources in your cloud environment"_

This is termed as _Infrastructure as Code_, and can help us automate our CI/CD pipeline as well as provision of resources for each step of the process. A CloudFormation template can be created using the visual builder present in AWS using drag-and-drop, or coding in json or yml.

In short, what **AWS CloudFormation** offers us is a way to combine AWS resources to build our CI/CD pipeline and also AWS resources to manage the infrastructure to host the application.

***a). CI/CD Pipeline***

*1. [AWS CodeCommit](https://aws.amazon.com/codecommit/) :*  This a fully-managed source service for git-based source repositories.

*2. [AWS CodeBuild](https://aws.amazon.com/codebuild/) :*  This a fully-managed CI server. This includes compiling source code, running test and producing software packages.

*3. [AWS CodePipeline](https://aws.amazon.com/codepipeline/):* This a fully-managed continous delivery service for automation of build,test,and deployment.

***b). Set up of infrastructure for deployment***

*4. [AWS ECR](https://aws.amazon.com/ecr/) :* This is a fully-managed Docker container registry that makes it easy for developers to store, manage, and deploy Docker container images

*5. [VPC](https://aws.amazon.com/vpc/) :* This is to provide a virtual cloud for our cluster. 

*6. [EKS Cluster](https://aws.amazon.com/eks/) :*  Amazon Elastic Container Service for Kubernetes. I'll talk more about this in a later section.

*7. [EC2 Instances](https://aws.amazon.com/ec2/) :*  EC2 nodes to run the Kubernetes pods

### Part One : CI/CD Pipeline

The CI/CD pipeline will use AWS resources to pick the source code from Github using **AWS CodeCommit** then using **AWS CodeBuild** build a docker image and then using **AWS CodePipeline** push the docker image to **AWS ECR**.

To create the CI/CD pipeline we can log into AWS and create it from the console UI

![aws pipeline ui cluster](/posts/aws-pipeline-ui.png)
###### <center> create ci/cd pipeline on aws ui </center>

or just use a CloudFormation template to to join all the resources in our template.

The CloudFormation template configured to use AWS CodePipeline to  will look [like](https://github.com/malike/cloud-native-kubernetes/blob/master/cloudformation/deployment-pipeline.yml).
If you're new to CloudFormation you can read more [here](https://speakerdeck.com/ndemoor/aws-cloudformation-crash-course) or [here](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/GettingStarted.Walkthrough.html).

From the Pipeline template, you notice we have three sections, **Parameters**, **Resources** and **Outputs**.

 **Parameters** enable us to use custom values to your template each time you create or update a stack. For our template this would be details for AWS CodeCommit to pull the source code from Github.

From Github, Using this url [https://github.com/settings/tokens](https://github.com/settings/tokens) we can get the parameters AWS CodeCommit will use.

*i. Github token name* <br/>
*ii. Github username* <br/>
*iii. Github repository name* <br/>
*iv. Github source branch name* <br/>

**Resources** declares AWS resources we'll like to include in the stack. The resources that enable us to build th CI/CD pipeline. It is in 3 parts, the first one been _CodePipeline_, this describes the pipeline from Github and using the infrastructure CloudFormation template to setup staging and production environment, the second _CodeBuild_, this also describes the resources used to build the source and the last part of resources describes all the _IAM_ policies needed.

**Outputs** which is not mandatory, is used to fetch values that can be reused in other templates.

![aws pipeline ci cd cluster](/posts/aws-pipeline-ci-cd.png)
###### <center> pipeline aws cloudformation visual </center>

To configure AWS CodeBuild to build the source code we need to add a `buildspec.yml` to the root of the source. This is is just to help AWS CodeBuild on what it needs to do to build the source. There are 3 phases involved to build a source `pre_build`, `build` and `post_build`.

For our source the goal we want to achieve for each build phase can be summarized as :

*pre_build :* This is the commands we need to run before build. In our case it does
Log in to Amazon ECR and set the repository URI to your ECR image and add an image tag with the first seven characters of the Git commit ID of the source.

*build :* This is commands to build the source. Build the Docker image and tag the image both as latest and with the Git commit ID

*post_build :* Push the image to your ECR repository with both tags.

If you check the source repository I've added some scripts for to help our structure of the repository.

This being the last part of configuring AWS resources to build a CI/CD pipeline we can move on test. How to test this?, lets make slight changes to the source on Github and verify if there will be new images on ECR. After committing the changes as we can see :

![ecr new images ](/posts/ecr-new-images.png)
###### <center> new images on ECR</center>

This confirms our CI/CD pipeline works perfectly. But we're not done. We still need to pick the images from **AWS ECR** and deploy to **AWS EKS**.

### Part Two : Infrastructure Set Up and Deployment

We left of the previous part with docker images uploaded to ECR after source code was pulled from Github and built. All with our CI/CD pipeline. Next step, set up an EKS infrastructure to pick the docker images, set up the infrastructure on EKS to deploy the application. We'll need a couple of resources to get this done and once again AWS CloudFormation comes to the rescue.

To deploy this on EKS we'll need, a VPC, EKS and 3 EC2 instances. Since our source code is made up a `java`, `go` and `html` microservices.

Putting everything together we have the [infrastructure](https://github.com/malike/cloud-native-kubernetes/blob/master/cloudformation/deployment-infrastructure.yml) CloudFormation template.

Putting the infrastructure together with the pipeline we can set up a `staging` environment and a `production` environment. But due to cost (:money_with_wings:) I'm only going to set up and staging environment.

### Summary  : Continuous Integration/ Continuous Deployment

To summarize what we've done so far :

**1.** Use CloudFormation to set up infrastructure for CI/CD pipeline.

**2.** Set up a Github repository, using AWS CodeCommit we can pick changes from the `master` branch Github.

**3.** The Docker image is pushed to Amazon ECR after a successful build and/or test stage.

**4.** CloudFormation sets up EKS clusters for staging.

**5.** EKS updates the deployment pods using a rolling update strategy by picking the images from Amazon ECR.

**6.** EKS updates the deployment pods using a rolling update strategy by picking the images from Amazon ECR automatically. _(Technically this wasn't done, due to cost )_

Hopefully you understand how to use strictly AWS resources to build CI/CD pipeline as well manage the infrastructure for such a deployment.

> Source code available on [Github](https://github.com/malike/cloud-native-kubernetes).

### References
[https://aws.amazon.com/blogs/compute/continuous-deployment-to-amazon-ecs-using-aws-codepipeline-aws-codebuild-amazon-ecr-and-aws-cloudformation/](https://aws.amazon.com/blogs/compute/continuous-deployment-to-amazon-ecs-using-aws-codepipeline-aws-codebuild-amazon-ecr-and-aws-cloudformation/)