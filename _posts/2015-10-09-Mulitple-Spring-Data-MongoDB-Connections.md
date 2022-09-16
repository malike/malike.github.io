---
layout: post
comments: true
title: Multiple Spring Data MongoDB Connections
author: malike
section: tech
tags: [documentation,sample]
redirect_from: "/Multiple-Spring-Data-MongoDB-Connections/"
image:
    path: /posts/dilbert-19960228.gif
    width: 800
    height: 500
alt: .
---


Using [Spring Data Starter MongoDB](http://projects.spring.io/spring-data-mongodb/) to connect to multiple MongoDB databases is quite easy. Its less complicated when 
you use **just** MongoTemplate for all your queries. But gets complicated when you use [both MongoTemplate and MongoRepository](http://stackoverflow.com/questions/17008947/whats-the-difference-between-spring-datas-mongotemplate-and-mongorepository). MongoTemplate connects to the intended secondary datasource but MongoRepository still uses the primary*(strange but true)*.

Fortunately there is a simple way to solve it. 

**1. Create your custom *MongoTemplate* bean for your secondary datasource.**

```java
@Bean
public Mongo mongo() throws Exception{
 return new MongoClient(host,port);
}

@Bean(autowire = Autowire.BY_NAME, name = "secondaryMongoTemplate")
public MongoTemplate secondaryMongoTemplate() throws Exception {
	return new MongoTemplate(mongo(),database);
}

```


**2. And Finally *@EnableMongoRepositories* with the custom MongoTemplate**

```java
@EnableMongoRepositories( 
    basePackages ={"this.is.your.repository.package"}, 
    mongoTemplateRef = "secondaryMongoTemplate"
  ) 
```


Now all MongoRepository implementations would use the same datasource configuration as your custom MongoTemplate.

> Ok. Later.


