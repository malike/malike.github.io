---
layout: post
comments: true
title: "To Contain Or Not To Contain"
layout: post
author: malike
categories: [devops,sre]
tags: [docker,kubernetes,distributed,go,java]
image:
  path: /posts/spicy-comic-7.jpg
  width: 800
  height: 500
---

So some of the known reasons why some people have talked to me about not adopting [containers](https://cloud.google.com/containers/). So this my attempt to convince them based on the concerns raised and few use cases where I won't suggest using containers.

Here goes : 

_**1. Why put a "container" in  a container?**_

First of all it seemed like a convenient way to shift the issues from `go builds` or `jars` to containers. _"It works on my machine"_ to _"It works in my container"_. 

Also an extra abstraction that wasn't needed, what's really the difference between eg between : `java -jar ` for jar files and `docker run` for docker containers. In both cases a `jar`, for example would have all dependencies packaged and can be run with a single command. Especially when we had all of the builds "wrapped" in shell scripts which we could start with `service service_name start` or `systemctl start service_name`.

_but,_

It's more than that, containers provide

_**i. Uniform abstraction**_, in a microservice architecture where you might have services developed in different languages, each tackling specific parts of the ecosystem one will use a container to provide a uniform abstraction independent of coding language. 
   
_**ii. [Processes, one of the 12 factors for Cloud native apps](https://12factor.net/processes)**_ indicates that an app is executed in the execution environment as one or more stateless processes. Containers help in packing and deployment of various microservices and only exports the services over a port, which is also [one](https://12factor.net/port-binding) of the factors.

What a container brings on board which the other options don't is, it's easier to start/stop or automate both based on requirements. Again this is because of the uniform abstraction. This is done irrespective of the OS and dependencies required by the service. Since all the all the dependencies can be packaged in the container. 

_**2. Security and Maintainability**_

Fewer the merrier : [Fewer dependencies](https://hackernoon.com/im-harvesting-credit-card-numbers-and-passwords-from-your-site-here-s-how-9a8cb347c5b), fewer security issues, fewer dependencies to deploy your service. 

For example  the [ELK docker containers](https://hub.docker.com/search/?isAutomated=0&isOfficial=0&page=1&pullCount=0&q=elk+elasticsearch+logstash+kibana&starCount=0) has lots of images how would you be sure of the integrity of any of the images you want to use. To be certain it wasn't compromised. 

_but,_

_**i. [Build, Release and Run](https://12factor.net/build-release-run)**_ indicates we strictly separate build and run stages to basically transform our codebase through a non-development process,usually automated, by fetching code dependencies to obtain a executable binaries. 
This is called the **Build** stage, closely followed by the **Release** which tags binary with the right configurations and settings for next stage **Run** which just means executing the application. 

This set of processes could be mundane if done manually, a container makes it worse. The end product of the this step, the container, could use unverified dependencies which could be security concerns. Just us devs need to verify and confirm dependencies used in source codes, the same process needs to be done for containers. 


Most of CI platforms support containers in their process, to the point of automation as well.

<br/>
[Some (docker to be specific)](https://thenewstack.io/5-docker-security-best-practices/) of the best practices for running containers in production have been highlighted there, I left out some of the "obvious" ones.

In my next post we'll use Github to build a CI pipeline for a [Build, Release and Run](https://12factor.net/build-release-run) CI pipeline with Docker to highlight.

### "Not To Contain"

_**1. Backing services?**_

IMO containers are cool but not everything needs to be in container. This is not because of [problems](https://thehftguy.com/2016/11/01/docker-in-production-an-history-of-failure/) with docker or if there are no means to do it. 

Here's why :

Containers are great for running databases in a dev environments but running [backing services](https://12factor.net/backing-services) on production eg: _dbs, messaging queues , etc..._ in a  container just a _no no_. You're better of using a managed services provided by cloud services.
 If you really have to self-host such services in a reliable fashion, you’re in for a lot of work and learning which is by far still a better alternative to containers like docker.

_**2. "Microservic-ing" a monolith by putting it in a container?**_

There was a time we had a long discussion in office on how _micro_ a microservice needs to be, is this the individual REST points each in a container or a bunch of _like-minded_ REST services in a single container. There are many school of thoughts on this but then again, a small software company in Ghana wiling to adopt microservices is not Netflix or Google to split services to minute end points in containers especially when you're now about to go on production. It doesn't scale. Imagine a team of 3 devs working on 100 microservices with no automation or tools to support it.

A microservice, to me, is an organisms which needs to evolve, change based on lots of measurements, monitoring and how users interact with it. It's easier if you have a monolith from the onset so you can tell for instance Signup Service would need to scale and not Login Service so let's decompose the Signup Service into a separate container to exist on its own for the start, which means it's easier to scale etc. It's not just breaking it up, it needs to abide by the [12 factors](https://12factor.net/) as well. It shouldn't be tightly coupled with the other services.

[Modernize Traditional Applications](https://www.docker.com/solutions/mta) is another toolkit by docker to help migrate traditional applications. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/NYv0kwlkwAY" frameborder="0"  encrypted-media></iframe>

<br>
<br>
**REFERENCES**

[https://12factor.net/](https://12factor.net/)

[https://sysdig.com/blog/7-docker-security-vulnerabilities/](https://sysdig.com/blog/7-docker-security-vulnerabilities/)
[https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#create-ephemeral-containers](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#create-ephemeral-containers)








