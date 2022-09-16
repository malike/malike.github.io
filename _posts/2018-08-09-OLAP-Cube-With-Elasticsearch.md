---
layout: post
comments: true
title: 'Using Elasticsearch as an OLAP Cube'
author: malike
categories: [foss]
image:
    path: /posts/foss_1.jpg
    width: 800
    height: 500
alt: .
fossname: elasticsearch-report-engine
fossurl: https://malike.github.io/elasticsearch-report-engine/
fossdescription: An Elasticsearch plugin to return query results as either PDF,HTML or CSV
fossimage: http://via.placeholder.com/220x220?text=Logo
tags: [olap,elasticsearch,report,analytics]
---

### What's an OLAP 

**Online Analytical Processing**  is the description of any technology that can help us to answer complex queries based on data stored in data warehouse, normally large volumes and across multiple sources. The data is structured such that complex queries would return results faster. This is purposely for decision making.

It supports complex aggregation and filters to enable faster decision making about future data and insight about past data with advance analysis. Performance of an OLAP is measured in how fast it returns aggregated queries. 

Data written in OLAP systems are hardly updated, in most cases it's not. This is because it would be an expensive query. So building an OLAP cube requires most if not all the  variables carefully thought out.

For a system to be used as an OLAP there are certain characteristics required.

**_1. Multidimensional data analysis_**

Whatever OLAP technology is being used, it must provide a multidimensional conceptual view of the data. 
It is because of the multidimensional view of data that we often refer to the data as a cube. A dimension often has hierarchies that show parent child relationships between the members of a dimension. The multidimensional structure should
allow such hierarchies. This is one of the distinctive characteristics of OLAP. 

With multidimensional analysis, data can be processed and viewed as part of a multidimensional structure enabling decision makers to view business data as data related related to other business data over a period of time.


**_2. Complex aggregations_**

The previous point describes how the data is stored with this step we can create multiple data aggregation levels, filter data and drill down across different dimensions and aggregation levels for different purposes. We can view the data in the form of graphs and charts or drill downs in tabular forms over different periods like bi-weekly, monthly, quarterly or weekly.


**_3. Business intelligence tool to visualize_**

The abstraction of running and visualizing all complex aggregations into simple user friendly views is also of great importance.
End users shouldn't know about the complex queries needed to get drill-downs. Most of the end users of OLAP services are business folks who would want the data to make fast decisions.
 

I can't talk about OLAP without mentioning **[OLTP](https://en.wikipedia.org/wiki/Online_transaction_processing)**, to read about the main differences between OLAP and OLTP, check [this](http://datawarehouse4u.info/OLTP-vs-OLAP.html) out. 

### Using Elasticsearch as an OLAP Cube

_Why is [Elasticsearch](https://www.elastic.co/products/elasticsearch)  a strong candidate to be used as OLAP based on our understanding of OLAP?_

**_1. How Elasticsearch supports multidimensional data analysis_**

Elasticsearch supports document stores,JSON, which we can model in any way we want. With support for REST, we can design any complex data models and write them in any programming language. Elasticsearch also has a [bulk insert api](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html). 


**_2. How Elasticsearch supports complex aggregations_**

Elasticsearch is based on [Lucene](http://lucene.apache.org/) and with the  JSON store we can quickly aggregate and filter the data using possible combinations. Having used in production for a financial institution we could aggregate over five years ,~180 million data very fast to enable the decision makers gather needed data over the raw data.


**_3. How Elasticsearch supports business intelligence tool to visualize_**

Elasticsearch has a [business analytics solution](https://www.elastic.co/solutions/business-analytics) which is based on Elasticsearch and Kibana, all part of the Elastic Stack.

Kibana already support [reports](https://www.elastic.co/products/stack/reporting) but it's use case is limited to data thats only on the view,HTML page, and does not give drill-down of data returned in the charts. Due to this limitation of generating reports with the analytics and reports of Elasticsearch and Kibana, I wrote a plugin [Elasticsearch Report Engine](https://malike.github.io/elasticsearch-report-engine/) to help generate more details reports and drill downs we see in Kibana.

This plugin supports CSV,PDF and HTML formats.

### How it works

Once this plugin is installed into elasticsearch search,it exposes the url http://localhost:9200/_generate, you can run queries on your cluster with the right parameters it would return PDF,HTML or CSV file. 

This version works on 
**[Elasticsearch 5.6.10](https://www.elastic.co/downloads/past-releases/elasticsearch-5-6-10)**

<br>

**_Install_**


```shell
sudo bin/elasticsearch-plugin install https://github.com/malike/elasticsearch-report-engine/releases/download/5.6.10/st.malike.elasticsearch.report.engine-5.6.10.zip
```
 
**_Grant permission_**

![Grant Access](https://malike.github.io/elasticsearch-report-engine/2permissions.png)


**_Folder Structure_**
 
Create folders `templates` and `reports` in `ES_HOME`. Store your  `*.jasper` and `*.jrxml`
files in the `templates` folder and pass the templateName as the `template` (with the right 
extension) parameter for `HTML` and `PDF` reports. 

[READ MORE](https://malike.github.io/elasticsearch-report-engine/)

If you have any problems using the plugin, kindly let [me know](https://github.com/malike/elasticsearch-report-engine/issues) or
send a [PR](https://github.com/malike/elasticsearch-report-engine/compare)


<br>
<br>
**REFERENCES**

[http://datawarehouse4u.info/OLTP-vs-OLAP.html](http://datawarehouse4u.info/OLTP-vs-OLAP.html)

[http://www.myreadingroom.co.in/notes-and-studymaterial/65-dbms/563-olap-and-its-characteristics.html](http://www.myreadingroom.co.in/notes-and-studymaterial/65-dbms/563-olap-and-its-characteristics.html)