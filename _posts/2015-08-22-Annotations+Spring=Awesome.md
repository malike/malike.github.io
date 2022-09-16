---
layout: post
comments: true
title: Java Annotations + Spring = Awesome
author: malike
categories: [tech]
tags: [documentation,sample]
redirect_from: "/Annotations+Spring=Awesome/"
image:
    path: /posts/academia_vs_business.png
    width: 800
    height: 500
alt: .
---

Java Annotations are really cool. They give this feature to send extra information that can be used during runtime
and/or compile time. 

I don't need to talk about [Spring](http://spring.io/) here. I personally think it the best thing to happen to JEE.

Combining Java Annotations and Spring IOC we can get this "clean" thing we would learn about in the next 3 min.

Ok. Here is the summary of what I wanted to do and how I achieved it. 

### What to do?

* Design a framework that can easily be integrated to read data from multiple sources and then index it in a single destination .
(This is a sample scenario)

### How I did it?

 **1. Created annotation Foo**

```java
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.TYPE)
    public @interface FooAnnotation {

    }
```

**2. Created interface** 


This interface is to serve as a "standard" for all systems who want to index their data.

```java
public interface Bar< T> {

    public List< T> getAllBar();   

}
```

**3. Implement All the BARs you can**

We use @Component so that the Spring IOC component scan can pick it up

```java
@Component
@FooAnnotation
public class BarI implements Bar< BarIData>{

	@Override
    public List< BarIData> getAllBar(){
    	// lets get BarIData for this implementation here
    }  

}
```

**4. Get All BARs Implementation with  FooAnnotation** 



Now this where Spring IOC comes in. Once you have your ApplicationContext autowired.

```java
@Autowired
private ApplicationContext context;
```

You can get all beans with FooAnnotation

```java
// get my beans 
Map<String, Object> myBeans = context.getBeansWithAnnotation(FooAnnotation.class); 

myBeans.keySet().stream().forEach((b) -> { 

Bar< Object> bar = (Bar) context.getBean(b); 

//use this implementation of bar however you want

bar.getAllBar();

 });
```


> Yep.That should work.





