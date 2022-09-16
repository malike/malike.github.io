---
layout: post
comments: true
title:  Build, Release, Dockerize and Run with Github, Travis CI and DockerHub
layout: post
author: malike
categories: [devops,sre]
tags: [docker,kubernetes,distributed,go,java]
image:
  path: /posts/ci-broken.png
  width: 800
  height: 500
---

If you've read my previous post, [Configuration Management for Microservices](https://malike.github.io/Configuration-Management-For-Microservices-And-Distributed-Systems.html), I created two microservices one in `Golang` and the other in `Java`. Using the two, I'll want to show how we can use [Travis CI](https://travis-ci.org/) for the  [Build, Release and Run](https://12factor.net/build-release-run) phase.

In a later article we'll use our docker images with [Kubernetes](https://kubernetes.io/).

### 1. Building Docker Images From Github for Go and Java Repos

To build a docker image for a our source codes we'll create  `Dockerfile`.
The `Dockerfile` is a set of instructions to build and properly describe a docker image.
Here's what some of the instructions mean :

##### Sample `Dockerfile` for Java

```Dockerfile
FROM openjdk:8
VOLUME /tmp
ADD target/config-server-1.0-SNAPSHOT.jar config-server.jar
RUN sh -c 'touch /config-server.jar'
ENV JAVA_OPTS="-Xms128m -Xmx256m"
EXPOSE 8888
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -Dspring.profiles.active=docker -jar /config-server.jar" ]
```

The [`Go Dockerfile`](https://github.com/malike/go-kafka-alert/blob/master/Dockerfile) is a multi-stage approach to build light weight images explained into details 
[here](https://medium.com/@pierreprinetti/the-go-dockerfile-d5d43af9ee3c).

**FROM**: Image with its tag as build-base. This is mostly [alpine linux](https://hub.docker.com/_/alpine/) because it's light and allows debugging of containers. There are other options as well like [scratch](https://hub.docker.com/_/scratch/).

**RUN**: Is used to execute a shell command-line within the target system.

**COPY**: Used to copy files from the local file-system to target image.

**ADD**: Used to also copy files like **COPY** but with addition [features](https://docs.docker.com/engine/reference/builder/#add).

**ENTRYPOINT** : Entrypoint sets the command and parameters that will be executed first in target system when it starts.

**ENV** : Used to define environment variables used in the target host.

**CMD**: Pass arguments to the **ENTRYPOINT** in the target system.

*You can check out the [docker documentation](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) to read more on it and [this](https://medium.com/@pierreprinetti/the-go-dockerfile-d5d43af9ee3c) on how to build build small size docker containers for `Go`*

### 2. Setting up our CI server : [Travis CI](https://docs.travis-ci.com/user/getting-started/).

_"Travis CI a hosted, distributed continuous integration service used to build and test software projects hosted at GitHub"_

Like the `Dockerfile` is a set of instructions to build a docker image, `.travis.yml` also contains a set of instructions for Travis CI to build and test the source hosted on Github.

In our case we'll use Travis CI to test and compile our source and then package a docker image that can easily be run. Travis CI doesn't store the docker images so we need to push to a docker repository. This can be a private one or a public one. For this example we'll use the public [DockerHub](https://hub.docker.com/).

To build and test our project,which uses maven, we'll add this.

```yml
script: cd $SERVICE_DIR && mvn clean verify
```

The maven goal test is intentionally left our because Travis CI does that by [default](https://blog.travis-ci.com/2017-03-30-deploy-maven-travis-ci-packagecloud/) once we specify java, I'm using the `script` because thats way let Travis CI building sub directories projects.
This works because I've specified the sub directories as environment variables.

```yml
env:
  global:
  - SERVICE_DIR=config-server
  - SERVICE_DIR=message-summary
```

After successfully building the source we'll need to package it using the docker configuration, `Dockerfile`. But first we'll have to enable Docker for Travis CI 

```yml
services:
    - docker
```

I wrote a bash script to handle the building and the uploading of the images. It uses the known 
[`docker build`](https://docs.docker.com/engine/reference/commandline/build/), [`docker tag`](https://docs.docker.com/engine/reference/commandline/tag/#description) and [`docker push`](https://docs.docker.com/engine/reference/commandline/push/) commands to complete this task.

First of Travis CI, would need access to upload the packaged images. Travis CI has a simple system for not storing passwords in plain text which requires installing a travis gem `sudo gem install travis`. Encrypt your passwords and add them to your travis config file by `cd`-ing into the project folder then `travis encrypt VARIABLE="secret-to-be-encrypted"`, confirm the repository name, it should return an encrypted text which you can add to your `.travis.yml`

```yml
env:
  global:
  - secure: "encrypted-dockerhub-username-1234"
  - secure: "encrypted-dockerhub-password-1234"
```

*For more details on writing your [travis configuration](https://docs.travis-ci.com/user/customizing-the-build/) to read more on it*

OK, now that we have a Docker image that is built by Travis, how do we use it? Well, we want to push the image to a Docker Registry for storage. Users can then pull the image from the registry to the machine where they will run the image.

##### Complete [`.travis.yml`](https://github.com/malike/go-kafka-alert/blob/master/.travis.yml) would look like this for a Java application

```yml
sudo: required
language: java
jdk:
  - oraclejdk8
before_script:
  - mongo go_kafka_alert --eval 'db.createUser({user:"travis",pwd:"test",roles:["readWrite"]});'  
services:
- mongodb
- docker
 
env:
  global:
  - secure: "encrypted-dockerhub-username-1234"
  - secure: "encrypted-dockerhub-password-1234"
  - SERVICE_DIR=config-server
  - SERVICE_DIR=message-summary

before_install:
  - chmod +x ./.travis/pushimage.sh

script: cd $SERVICE_DIR && mvn clean verify

after_success: 
  - bash ../.travis/pushimage.sh config-server
  - bash ../.travis/pushimage.sh message-summary
```

##### Check [`.travis.yml`](https://github.com/malike/go-kafka-alert/blob/master/.travis.yml) for the travis config for our Go application

Summary so we successfully setup an automated process using Travis CI, `Dockerfile` configurations to build an image and then successfully uploaded to [DockerHub](https://hub.docker.com/u/stmalike/dashboard/). Although the title is _"Build, Release, Dockerize and Run with Github, Travis CI and DockerHub"_ we're yet to actually run our service. <br/>
This would be continued in the next article with other things we can introduce in the pipeline to produce efficient results.

<br>
<br>
**REFERENCES**

[https://12factor.net/](https://12factor.net/)

[https://medium.com/@pierreprinetti/the-go-dockerfile-d5d43af9ee3c](https://medium.com/@pierreprinetti/the-go-dockerfile-d5d43af9ee3c)