---
layout: post
comments: true
title: Spring Metrics For Simple Event Monitoring
author: malike
categories: [tech]
tags: [documentation,sample]
image:
    path: /posts/screwupcolor.png
    width: 800
    height: 500
alt: .
---


Sometimes (or all the time) you would want to know whats happening in your application's life cycle.
[Spring Boot Metrics](http://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html) is just the right tool
for application metrics in any [Spring Boot Application](https://spring.io/guides/gs/spring-boot/). It exposes 2 metrics services, **"Gauge"** ,  **"Counter"** out of the box as well as an interface **PublicMetrics** to capture custom
metrics.

We would use all 3 to capture application level metrics.

Before getting this to work add the Spring Boot Actuator Dependency (I assume you are using maven)

```xml
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
    <version>{version}</version>
  </dependency>
```

The first two are fairly simple. A simple 2 step process of ***initialize*** and  ***use***.

**1. Gauge**

A Gauge helps record a single value. Data type is *double*.
Autowiring  GaugeService before using it.

```java
@Autowired
private GaugeService gaugeService;
```

then

```java
gaugeService.submit("sample.metric", 20); //20 is sample metric value
```

**2. Counter**

A  Counter records an increment or decrement. Data type is *int*.

```java
@Autowired
private CounterService counterService;
```

this

```java
counterService.increment("sample.metric"); //adds 1
```

or that

```java
counterService.decrement("sample.metric"); //subtracts 1
```

**3. PublicMetrics**

This uses a different approach. Implement [PublicMetrics](https://github.com/spring-projects/spring-boot/blob/v1.2.5.RELEASE/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/endpoint/PublicMetrics.java).

then

```java
@Component
public class CustomMetric implements PublicMetrics {

    public CustomMetric() {
    }

    @Override
    public Collection< Metric< ?> > metrics() {
        Collection< Metric< ?> > result = new LinkedHashSet<>();
        result.add(new Metric<>("sample.metric", 20)); //20 is sample metric value
        // u can add multiple metrics to results
        return result;
    }
}
```


Now that we've captured the metrics. How do we *read it?*

Goto [http://localhost:{port}/metrics](http://localhost:{port}/metrics).

> That's about it.

