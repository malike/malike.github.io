---
layout: post
comments: true
title: ... a little more with spring metrics
author: malike
categories: [tech]
tags: [documentation,sample]
image:
    path: /posts/tape_measure.png
    width: 800
    height: 500
alt: .
redirect_from: "/Little-More-With-Spring-Metrics/"
---


If you read [this](http://malike.github.io/Spring-Metrics), we talked about capturing and reading application metrics.

In this post I'll show you how to do a *little* more with that data using Spring Metrics and [Lambda Architecture](http://malike.github.io/Lambda-Architecture-RT) which we also talked about. 

I'll categorize this post under two sections ***capturing time series of metrics*** and ***persisting metrics***


**1. Capture Time Series**

There are two ways to get this done. One of them is *hack* for the purpose of this post, because it makes no use of Spring Metrics or the Lambda Architecture.


I'm assuming we know how the Lambda Architecture works. The only addition to make this work for our purpose is we feeding the Lambda Architecture stream with metric events and then leaving it for our Lambda Architecture. Everything else remains the same as with [this approach](http://malike.github.io/Spring-Metrics) of capturing the metric. The results would be an aggregated time series of the results persisted in our **batch layer** with the **speed layer** giving us real time or near real time events of the data.

Now the hack, this solely relies on the memory by keeping all aggregated events stored in memory.
See *how-to* below.

Declare your variables

```java
 private LinkedHashMap<String, LinkedHashMap<String, LinkedHashMap<String, Integer>>> timeMap = new LinkedHashMap<>();
 private static SimpleDateFormat dateFormat = new SimpleDateFormat("dd-MM-yyyy HH:mm");
```

Increase metric

```java
public void increment(String metric) {   
    String time = dateFormat.format(new Date());
    LinkedHashMap<String, LinkedHashMap<String, Integer>> hashMap = timeMap.get(metric);
    hashMap = (timeMap.isEmpty() || null == hashMap) ? new LinkedHashMap<>() : hashMap;
    LinkedHashMap<String, Integer> statusMap = hashMap.get(time);
    statusMap = (null == statusMap) ? new LinkedHashMap<>() : statusMap;
    Integer count = statusMap.get(metric);
    count = (null == count) ? 1 : count++;
    statusMap.put(metric, count);
    hashMap.put(time, statusMap);
    timeMap.put(metric, hashMap);
}    
```

As you can see it's just for increasing a metric, for decreasing a metric I'm sure you can figure that part out.
 

**2. Persisting to database**

If you decide *to Lambda* you can bridge the channel reading the stream and the channel writing to Apache Kafka.

If you decide to *not Lambda* then build a simple schedule job that writes the metric
data to the database. You can easily get this done with something like this.

```java
@Configuration
public class MetricDBConfig {

    
    @Scheduled(initialDelay = 60000, fixedDelay = 60000)
    void saveMetric() {
        //Save to DB for singular or all metrics
    }
    
    public LinkedHashMap getSingularMetric(String metric) {
        //use this to get  time series for a metric
         return timeMap.get(metric);       
    }

    //
    public LinkedHashMap getAllMetric() {
        return timeMap; //all of 'em
    }

}
```

For the Lambda Architecture picking the event metrics to be written to Apache Kafka would be slightly different.

```java
@Configuration
public class MetricDBConfig {

    @Autowired
    private MetricRepository repository;
   

    @Scheduled(initialDelay = 60000, fixedDelay = 60000)
    void saveMetric() {
        //Save to Apache Kafka
    }

}
```


> Ok.

