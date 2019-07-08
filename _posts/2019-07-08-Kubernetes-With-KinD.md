---
layout: post
comments: true
title:  'Running Kubernetes Locally via KinD'
layout: post
author: "malike_st"
section: sre,devops
tags: [gke,kubernetes,istio,sidecar]
image: works-in-my-image.jpg
---



Every application will at a point grow significantly having to serve huge traffic depending on the size of the customer base. The capability of a system, network, or process to handle a growing amount of work is becoming a primary concern for developer teams right from the start. What if this is simulated in some test area. Well, in this article I will talk about one cool tool I found for performing application load testing: K6.io and also share some configuration snippets.

What is K6?
k6 is a developer-centric open source load testing tool for testing the performance of your backend infrastructure. It’s built with Go and JavaScript to integrate well into your development workflow, so you can stay on top of performance without fuzz.

Installation
Installation comes in different forms. For the sake of this illustration, I will be using the Linux installation guide as below :

sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 379CE192D401AB61
echo "deb https://dl.bintray.com/loadimpact/deb stable main" | sudo tee -a /etc/apt/sources.list
sudo apt-get update
sudo apt-get install k6
You refer to the official installation guide on the website.

How it Works
Just as most frameworks, k6 needs a configuration script with which it will be running tests. Below is an example code snippet for load testing a single GET endpoint.

import { check, sleep } from "k6";
import http from "k6/http";
export default function() {
    let res = http.get("http://my.get.url/");
    check(res, {
        "is status 200": (r) => r.status === 200
    });
 
};
Now we can run this test with below command :

k6 run --vus 10 --duration 30s load_test_example.js
See metrics output :


load test result
vus: current number of active virtual users

vus_max: Max possible number of virtual users

See the detail explanation of metrics here

Configuring Thresholds
This is one of the cool features I like about K6. What this means is that, you can typically set thresholds for success, failures or more specific errors like error 500 so that depending on the results you pass a deployment of a CI/CD pipeline. See the example below

import http from "k6/http";
import { Rate } from "k6/metrics";
var failedRate = new Rate("failed requests");
export let options = {
 thresholds: {
   "failed requests": ["rate<0.1"],
  }
};
export default function() {
 let res = http.get("http://my.get.url/");
 failedRate.add(res.status != 200);
};

You can see from the image above that failed requests is 0.00%

This definitely cool :-)

Visualization of Metrics
Another nice feature of K6 is integration into time series database Influxdb. This means we can visualize metrics being spat from your load testing in real-time using Grafana or another alternative. Again for the sake of this session, I will be using Grafana.

This assumes you have Influxdb and Grafana both already setup.
k6 run --vus 15 --duration 60s --out influxdb=http://localhost:8086/k6_test load_test_example.js
Setup an Influxdb Datasource

After running above we need to create a data source for Grafana (Configuration > Datasource). See below



Setup Dashboard

Download the Grafana k6 Load Testing Results template and import in Grafana to preview Graphs: Create > Import > Upload .gson





<br>
<br>
**REFERENCES**

[https://docs.microsoft.com/en-us/azure/architecture/patterns/sidecar](https://docs.microsoft.com/en-us/azure/architecture/patterns/sidecar)


[https://dzone.com/articles/deploying-microservices-spring-cloud-vs-kubernetes](https://dzone.com/articles/deploying-microservices-spring-cloud-vs-kubernetes)