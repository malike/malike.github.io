---
layout: post
comments: true
title: "seeing is believing, if cheaper the better"
author: "malike_st"
categories: 'electronics'
tags: [data-flow,opencv,iot]
projectname: Project Theia
projectdescription: Sight for IoT devices uses mqtt, kafka and spring cloud
image: car_problems.png
---


So I decided to submit an entry for [Hackaday Robotics Module Challenge](https://hackaday.io/prize/details#two). I only decided to because I had the components and I thought I could squeeze in some time over the weekend to complete this project and actually have a write up on it. Unfortunately I couldn't finish before the deadline but instead of leaving it unfinished I decided to finish it,blog about it.

What's [Project Theia](). Its a combination of 3 main parts,[the hardware](), [the server]() and [plugins for data processing](),  everything is available on github and per the rules it has to be OSS so contributions are welcome.

It's coded in Java, C, JS and Python and heavily relies on `mqtt` and `Apache Kafka` for communication, with `mqtt` for Theia Hardware and Theia Server and  `Apache Kafka` to stream feeds from the camera between configured paths to achieve required goals.

It's built using a [micro-kernel]() architecture, meaning customization can be be done without necessarily modifying the core components.


#### The Componentes

**1. The Hardware**

Components and Data Sheets

**_i. OV7676 Camera_**

**_ii. Arduino Pro Mini_**

**_iii. Bluetooth Module_**

**_iv. sim300L _**

**_iv. SD Card Module_**



**2. The Server**

[Spring Cloud Data Flow]()

**3. The Plugins**

[Spring Cloud Data Flow Apps]()

**_i. Source_**

**_ii. Transformers_**

**_iii. Sink_**



#### Putting it all together : Sample Projects

_1. Using Thiea To Send Email when it Recognises a Face_

Per the rules, you'll also need to submit a sample project of your Robotics Module,so this is the sample project I buitl with it


If you read


_2. Using Thiea To Build A Self Navigating Robot That Streams Video To Instagram_









