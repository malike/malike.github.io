---
layout: post
comments: true
title: Configuration Management for Microservices
layout: post
author: malike
categories: [distributed,sre]
tags: [distributed,configuration,zookeeper,elasticsearch,microservice]
fossname: go-kafka-alert
fossurl: https://malike.github.io/go-kafka-alert/   
fossimage: http://via.placeholder.com/220x220?text=Logo
fossdescription: A Go application that feeds of data from Apache Kafka to send alerts via SMS,EMAIL or WebHook delivery channels.
projectname: Configuration Management for Cloud Native/Distributed Systems
projectdescription: Project to show how to use Centralized Configuration Management in a cloud native or distributed systems
projecturl: https://github.com/malike/centralized-configuration-mangement
image:
    path: /posts/dilbert-cartoon.jpg
    width: 800
    height: 500
alt: .
redirect_from: "/Configuration-Management-For-Microservices-And-Distributed-Systems/"
---


Centralized configuration management is one of the major requirements in a microservice system. Although one of the major
proponents of a microservice system is decentralized services, certain parts still need to be centralized to ease
deployment pains. 
Centralized configuration is simply a main _location_ where all microservices can _pick_ their specific configurations to help them run perfectly.

Some of the well known options are : 

##### 1. [etcd](https://github.com/coreos/etcd)

_"etcd is a distributed reliable key-value store for the most critical data of a distributed system, with a focus on being:_

_Simple: well-defined, user-facing API (gRPC)_
_Secure: automatic TLS with optional client cert authentication_
_Fast: benchmarked 10,000 writes/sec_
_Reliable: properly distributed using Raft"_

When researching on using etcd as config server, I came across these resources [this, implemented in java](https://github.com/Redpill-Linpro/spring-config-etcd) and [also this in python ](https://www.compose.com/articles/building-a-dynamic-configuration-service-with-etcd-and-python/). 

##### 2. [Zookeeper](https://zookeeper.apache.org/)

_"ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services. All of these kinds of services are used in some form or another by distributed applications"_

This is supported in [Spring](http://cloud.spring.io/spring-cloud-zookeeper/). If you use Apache Kafka, you'll know how it relies on 
Zookeeper's high availability and fault tolerant for storing and retrieval of K,V pair of topic and consumer configurations.

I found [this](https://mmcgrana.github.io/2014/05/getting-started-with-zookeeper-and-go.html) when researching on using Zookeeper with Go. It's quite old but with very good explanation.

Netflix's [Apache Curator](https://curator.apache.org/) is java client for Zookeeper which extremely helpful with especially when not using the Spring stack. 

##### 3. [Consul](https://consul.io/)

Consul is not just a _K,V_ store, one underlying principle of Config Servers. It has `Service Discovery`, `Service Health Monitoring` as well as other functionalities which makes it the preferred choice. With it's well documented [apis and sdks](https://www.consul.io/api/libraries-and-sdks.html) with examples in different languages and frameworks it would be the best option. But the simplicity of the next option is what makes this post not about Consul. 

{::nomarkdown}
</br></br>

{:/}


...and finally **4. [Spring Cloud Config](https://cloud.spring.io/spring-cloud-config/)**

Since this post is about Spring Cloud Config, let's dive in :

It's best to explain [codes](https://github.com/malike/centralized-configuration-mangement.git), if you follow the link you'll see a project where by a `golang` and a `java` application worked with a Spring Cloud Config to retrieve configurations. The golang project is an already existing project [go-kafka-alert](https://github.com/malike/go-kafka-alert).
 
**1. Manual Configuration Refresh**

Aside from just serving as a K,V store to store and retrieve configurations, a Config Server should have in place systems to help update or refresh configurations in services after updates. In a Spring Boot/Cloud application connecting to a config server,this can be achieved by doing the following:

a. Location of config Server, by adding this to property file. This informs the Spring Boot/Cloud application to override default properties by checking the Config Server for properties. 

`spring.cloud.config.uri=http://localhost:8888`

It uses this convention `http://localhost:8888/{spring.application.name}-{profile}.yml`. Where `spring.application.name` is the name of the service configured in your spring application. 

b. Secondly using `@RefreshScope` annotation to distinguish properties to be updated after refresh and those that should not. In our sample application

c. Lastly
Hit the `/refresh` endpoint of the service for our service to reload the new configurations.

_NB: You'll need `org.springframework.boot:spring-boot-starter-actuator` as part of your project to expose the refresh endpoint_

How would you replicate a similar thing in a Golang application, 

**2. Automatic Configuration Refresh**

We've seen how the manual configuration update works. But it could be better to not involve manual processes. 
One advantage of this is,it reduces the amount of work to done with after configuration updated and also helps us use Spring Cloud Config in microservices or distributed systems not written using the Spring stack.

This would require a messaging queue,`Apache Kafka` or `RabbitMQ`. `Redis` is also a supported by Spring Cloud Config. It also requires a git webhook in `Github`,`Bitbucket` or `Gitlab` to tell the config server if any change is pushed, that is for configuration files hosted on `Git`. The messaging queue enables services subscribed to _see_ the configurations files updated. 

To get this working we'll need to add dependencies for `spring-cloud-config-monitor` and `spring-cloud-starter-bus-kafka`( you can use either redis or rabbit bus dependency)

`spring-cloud-config-monitor` enables the end point `/monitor` in our config server which we can then add as webhook to our configuration repository

![_config.yml](/posts/config-webhook.png) 
{::nomarkdown}
<span style="font-size:13px;">NB: Although this image shows how-to in github,it can also be done in gitlab and bitbucket</span>
{:/}


Anytime there's a change in this repo it's pushed to the Config Server. For configuration file stored
on the file system Spring Cloud Config automatically picks the changes so you won't require any webhook.

A sample Cloud Bus configuration in Spring Config Server, detailing where it should pick the configuration as well Kafka configuration to push events to.

<pre><code class="yml">
    spring:
        application:
            name: config-server
        cloud:
            config:
            server:
                monitor:
                github:
                    enabled: true         
                health:
                enabled: true     
                git:
                uri: https://github.com/malike/centralized-configuration.git
                force-pull: true
                username: username
                password: password
            bus:     
            enabled: true
            refresh:
                enabled: true
            stream:
            kafka:
                binder:
                zkNodes: localhost:2181
                brokers: localhost:9092    
</code></pre>

If our service is a Spring Boot/Cloud application we would need just [one other configuration]()
for it to work out of the box but for a `Go` (or a non Spring Boot/Cloud ) application also using the Config Server 
we'll need to be subscribed to the topic `springCloudBus` and update the config anytime there's an 
update the configuration file in used.

![_config.yml](/posts/springCloudBus-ConfigUpdate.png) 
{::nomarkdown}
<span style="font-size:13px;">sample message from in kafka after updating the go-kafka-alert-production we can use to trigger configuration reload</span>
{:/}

**3. Configuration Format Support**

Formats supported by Spring Cloud Config are `JSON`, `YML` and Java `properties`. The addded advantage of Spring  Cloud Config 
is any configuration file served as either format can be accessed in all available formats. For example a config file
that's served as `application-message-summary-uat.properties` can be accessed by using `application-message-summary-uat.properties`
or `application-message-summary-uat.json` or `application-message-summary-uat.yml`.

In the [sample configurations](https://github.com/malike/centralized-configuration.git) there's a `yml` configuration 

Configuration files can be loaded from the file system or git with a simple configuration or relational databases with extra configuration and db setup.

this for file system

    spring.cloud.config.server.git.uri=${HOME}/Desktop/config

or this for git
    
    spring.cloud.config.server.git.uri=https://github.com/malike/centralized-configuration.git

**4. HTTP  Support For Retrieving Configurations**

Although Spring Cloud Config is a java based application it can work with a variety of languages due to it's HTTP API. 

{::nomarkdown}
http://localhost:8888/{application}/{profile}[/{label} </br>
http://localhost:8888/{application}-{profile}.{yml|json|properties}</br>
http://localhost:8888/{label}/{application}-{profile}.{yml|json|properties}</br>
http://localhost:8888/{application}-{profile}.{yml|json|properties}</br>
http://localhost:8888/{label}/{application}-{profile}.{yml|json|properties}</br></br>
{:/}

*NB: Base url is http://localhost:8888 because thats the [configuration](https://github.com/malike/centralized-configuration-mangement/blob/master/config-server/src/main/resources/application.yml) in our config server*

For our sample app, with configuration names `message-summary` and `go-kafka-alert` we'll create config files with profiles for `uat`
and `production` as `message-summary-production.properties` or `message-summary-uat.properties`. As I mentioned earlier the format doesn't really matter once it's supported we can access with with any extension meaning we can access our `yml` configuration files as `json` or `properties`. 

**5. Security**

Spring Boot/Cloud apps can be secured simple by adding `spring-boot-starter-security` and with the [right configuration](https://spring.io/guides/gs/securing-web/) we can enable basic authentication for the Config Server.

We can also take it a step further to restrict passwords and sensitive data in the configuration. This would prevent anyone who has access to the configuration files from seeing our database credentials in plain text. Spring Cloud Config supports encrypting and decrypting properties. You can read more [here](https://cloud.spring.io/spring-cloud-config/single/spring-cloud-config.html#_encryption_and_decryption)
This would require you to install Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy File which is not part of the JVM by default. By pre-pending passwords and sensitive properties with `{cipher}` and in quotes. Sensitive properties would be encrypted in git but would be decrypted on requests by services. Spring Cloud Config also comes with two end points to help in encrypting and decrypting properties.

To enable [creating your key](https://cloud.spring.io/spring-cloud-config/single/spring-cloud-config.html#_creating_a_key_store_for_testing) and [adding it to your config server](https://cloud.spring.io/spring-cloud-config/single/spring-cloud-config.html#_key_management).


There many pros and cons of using Spring Cloud Config, I think I've highlighted the pros a lot in this post but some of the major cons associated with this the Config Server becoming an app on it's own that needs to be managed and monitored like the microservices or distributed systems are not already enough and the fact that it going down can force services to either not start or or rely on fallback configurations but then again these same points maybe seen as pros in other infrastructures.
I do hope though that you've understood how to use Spring Cloud Config to serve configurations in a microservice or distributed system. 



> Check out codes on [github](https://github.com/malike/centralized-configuration-mangement.git)
> and configuration files also [here](https://github.com/malike/centralized-configuration.git)



<br>
<br>
**REFERENCES**

[https://cloud.spring.io/spring-cloud-config/single/spring-cloud-config.html](https://cloud.spring.io/spring-cloud-config/single/spring-cloud-config.html)

