---
layout: post
comments: true
title: Gitlab CI/CD Pipeline for a Maven Project
author: malike
categories: [devops,sre]
tags: [devops,ci,cd,gitlab]
image:
    path: /posts/dilbert_devops_monkey.jpg
    width: 800
    height: 500
alt: . 
---



_**What is [CI/CD](https://dzone.com/articles/what-is-cicd)?**_

_"Continuous integration focuses on blending the work products of individual developers together into a repository. Often, this is done several times each day, and the primary purpose is to enable early detection of integration bugs, which should eventually result in tighter cohesion and more development collaboration."_

_"The aim of continuous delivery is to minimize the friction points that are inherent in the deployment or release processes. Typically, the implementation involves automating each of the steps for build deployments such that a safe code release can be done—ideally—at any moment in time. Continuous deployment is a higher degree of automation, in which a build/deployment occurs automatically whenever a major change is made to the code."_



[Gitlab CI Agent](https://docs.gitlab.com/runner/) is GitLab Continuous Integration & Deployment
GitLab has integrated CI/CD pipelines to build, test, deploy, and monitor your code.


To successfully create a CI/CD pipeline using Gitlab you'll need a couple of things :


**1. Pipline on Gitlab**

Create your CI/CD pipeline on [Gitlab](https://gitlab.com) or a self [hosted gitlab]() as well.

The pipeline,which is basically a set of instructions should be created in a file `.gitlab-ci.yml` placed in your project root. 
[Check this](https://docs.gitlab.com/ee/ci/quick_start/) to write your own but a simple pipeline would look something like this

```yml
# These are the default stages.
# You don't need to explicitly define them.
# But you could define any stages you want.
stages:
  - build
  - test
  - deploy

# This is the name of the job.
# You can choose it freely.
maven_build:
  # A job is always executed within a stage.
  # If no stage is set, it defaults to 'test'.
  stage: test
  # Since we require Maven for this job,
  # we can restrict the job to runners with a certain tag.
  # Of course, it is our duty to actually configure a runner
  # with the tag 'maven' and a working maven installation
  tags:
    - maven
    - java
    - ubuntu
  # Here you can execute arbitrate terminal commands.
  # If any of the commands returns a non zero exit code the job fails.
  script:
    - echo "Maven build started"
    - mvn verify
```



**2. Setup Host server**

Install `mvn` and the necessary dependencies to run your application. Then install the gitlab [runner](https://docs.gitlab.com/runner/install/) which is also available for platforms like [Linux](https://docs.gitlab.com/runner/install/linux-manually.html), [MacOS](https://docs.gitlab.com/runner/install/osx.html), [K8s](https://docs.gitlab.com/runner/install/kubernetes.html) and [Windows](https://docs.gitlab.com/runner/install/windows.html). 

If you're _too careful_ and would want to test if it works you can use [Docker](https://docs.gitlab.com/runner/install/docker.html) no need to pay for a new server.

##### *i. Verify your installation*

```shell
$ gitlab-runner list
Listing configured runners  ConfigFile=/{HOME}/.gitlab-runner/config.toml
```

This is expected because we do not have nay configured pipelines

##### *ii. Register a CI/CD pipeline*

Register  pipeline with :

```shell
gitlab-runner register
```

You'll have to fill the prompt with the right parameters

```shell
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
https://gitlab.com/
Please enter the gitlab-ci token for this runner:
token1234
Please enter the gitlab-ci description for this runner:
[$]: maven-demo-app
Please enter the gitlab-ci tags for this runner (comma separated):
maven, java, ubuntu
Whether to run untagged builds [true/false]:
[false]:
Whether to lock Runner to current project [true/false]:
[false]: true
Registering runner... succeeded                     runner=random_Q3_i4
Please enter the executor: docker-ssh, parallels, ssh, docker+machine, docker-ssh+machine, docker, shell, virtualbox, kubernetes:
shell
```
*Note:* 

`gitlab-ci`:
![gitlab pipeline](/posts/pipeline-ci-cd-token.png) 

`gitlab-ci tags` is a discrimintor with which we can tell the runner to either deploy the build or skip it. With this example the tags are `maven, java, maven` and by setting the runner not to run `untagged builds` we've essentially turn the discriminator on. We assigned the tag to our build in the `.gitlab-ci.yml` file with this :

```yml
tags:
    - maven
```    
Gitlab runner has different executors for running the build, the executor for the `maven` is `shell`.

You can also configure Gitlab to pass custom environment variables. I prefer not to do this though. Configurations are always managed
by a [configuration server](https://malike.github.io/Configuration-Management-For-Microservices-And-Distributed-Systems.html).


![gitlab pipeline](/posts/pipeline-ci-cd-config.png) 


#### *iii. Running the Build*

Now the Runner is ready we can test by pushing a commit to your GitLab. Gitlab automatically triggers the pipeline once the tags match.

Each commit is represent differently on Gitlab with test results as well details on whether it was deployed on target host.
There's also information on when it was executed

![gitlab pipeline](/posts/pipeline-ci-cd.png) 



<br/>
<br/>
**REFERENCES**
[https://docs.gitlab.com/runner/](https://docs.gitlab.com/runner/)