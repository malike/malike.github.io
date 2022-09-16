---
layout: post
comments: true
title: Lambda Architecture With Kafka, ElasticSearch, Apache Storm and MongoDB
author: malike
categories: [distributed]
tags: [documentation,sample]
image:
    path: /posts/im_an_idiot.png
    width: 800
    height: 500
alt: .
---

How I would use [Apache Storm](https://storm.apache.org/),[Apache Kafka](http://kafka.apache.org/),[Elasticsearch](http://www.elasticsearch.org/) and [MongoDB](https://www.mongodb.org) for a monitoring system based on the lambda architecture.


**What is** ***[Lambda Architecture](https://www.manning.com/books/big-data)?***

It's a design principle where all derived calculations in a data system can be expressed as a re-computation function over all of your data. This re-computation would be done over immutable data readily available.

***Lambda Architecture*** is made of **3** layers,part played in the Lambda Architecture is summarized below :

1. **Batch Layer** which computes functions over all data with high latency and rewrites immutable fact transformations into your data stores (*e.g. via Hadoop,MongoDB in our case*)
2. **Speed Layer** which computes functions over recent data with low latency and mutates your real-time data stores directly (*e.g. via Storm*)
3. **Serving Layer** which abstracts over those two to provide unified answers to real-time and historical questions (*e.g. Cloudera Impala,ElasticSearch in our case*)

<br>
<br>


* **Batch Layer and Why use** ***[MongoDB](https://www.mongodb.org)?***

The purpose of the batch layer is to serve as the immutable data store. Its purpose is to read and create out of the the four
CRUD(*Create,Read,Update and Delete*) operations. (Reason the data is immutable). The database to be used for the batch layer
should be able to support faster reads and write. With these as our priority we can safely cross out relational databases
because consistency is not our highest priority when it comes to *reading and creating*.

Because I don't have the time to test all the available NoSQL databases to see which of them supports faster reads and writes,
I searched online. I regretted. To be honest,this was the hardest part. I was so overwhelmed with contradicting performance reports I felt I should test them all myself. But I don't have the time or the resources to do that now.
Hands down,the clear winner is [Hadoop](https://hadoop.apache.org/). I was actually looking for [Hadoop](https://hadoop.apache.org/) alternative because I wanted anyone could easily
deploy and have it up and running in no time.

So I picked [MongoDB](https://www.mongodb.org).

*Disclaimer: I picked [MongoDB](https://www.mongodb.org) as* ***my*** *batch layer,it doesn't in anyway mean its the best option(at the time of this post). So If you or your organization has a better reason to pick a [MongoDB](https://www.mongodb.org) alternative. Please do.*
<br>
<br>

Other considerations were [Aerospike](http://www.aerospike.com/), [Cassandra](http://cassandra.apache.org/),[ElephantDB](https://github.com/nathanmarz/elephantdb) and definitely [Hadoop](https://hadoop.apache.org/).
But whatever your choice is,the goal is to have ***faster,concurrent reads and writes***.


* **Speed Layer and Why use** ***[Apache Storm](https://storm.apache.org/) and [Apache Kafka](http://kafka.apache.org/)?***

The speed layer or the real time layer takes the stream of data,real time,calculates indexes the data. Obviously this is not immutable data.

Why do I use *[Apache Storm](https://storm.apache.org/)?* It was ***"born"*** for this.

*"Apache Storm is a free and open source distributed realtime computation system. Storm makes it easy to reliably process unbounded streams of data, doing for realtime processing what Hadoop did for batch processing. Storm is simple, can be used with any programming language, and is a lot of fun to use!*

*Storm has many use cases: realtime analytics, online machine learning, continuous computation, distributed RPC, ETL, and more. Storm is fast: a benchmark clocked it at over a million tuples processed per second per node. It is scalable, fault-tolerant, guarantees your data will be processed, and is easy to set up and operate."*

And also Nathan Marz who first wrote about the Lambda Architecture created [Apache Storm](https://storm.apache.org/) as well.

Why *[Apache Kafka](http://kafka.apache.org/)?*

We need a resilient messaging queue that would feed the speed layer with the stream of data. Sort of like a pool for
all stream data.So we have one source for getting the data.

[This](http://www.infoq.com/articles/apache-kafka) and [this](http://java.dzone.com/articles/exploring-message-brokers) is why I chose Apache Kafka.



* **Serving Layer and Why use** ***[ElasticSearch](http://www.elasticsearch.org/)?***

*"Elasticsearch is a flexible and powerful open source, distributed, real-time search and analytics engine. Architecture is from the ground up for use in distributed environments where reliability and scalability are must haves, Elasticsearch gives you the ability to move easily beyond simple full-text search. Through its robust set of APIs and query DSLs, plus clients for the most popular programming languages, Elasticsearch delivers on the near limitless promises of search technology"*


The serving layer combines the output from the batch and speed layer. This layer helps us get the data,combined, from
both the batch and serving layer.

Output from the batch and speed layers are stored in the serving layer, which responds to ad-hoc queries by returning pre-computed views or building views from the processed data

The serving layer provides and answers to getting historical(batch) data and real time data(speed).

If you end up using [MongoDB](https://www.mongodb.org) as your batch layer you can use  [MongoDB River](https://github.com/richardwilly98/elasticsearch-river-mongodb). This would help keep (backup) records for the historical data with minimal configuration.
<br>
For the real-time data.Our Speed layer using a Storm Spout would help us aggregate and index real-time data in [ElasticSearch](http://www.elasticsearch.org/).



> Putting It All together - [The Lambda RT Project](https://github.com/malike/LambdaRT)



<br>
<br>
**REFERENCES**

[http://jameskinley.tumblr.com/post/37398560534/the-lambda-architecture-principles-for-architecting
](http://jameskinley.tumblr.com/post/37398560534/the-lambda-architecture-principles-for-architecting
)  -  James Kinley
