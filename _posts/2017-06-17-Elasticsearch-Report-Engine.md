---
layout: post
comments: true
title: Elasticsearch Report Engine
categories: [oss]
tags: [oss, elasticsearch]
---


Plugin to generate Reports from Elasticsearch Queries.

  - [Basic Overview](#overview)
  - [Install](#install)
  - [Usage](#usage)
    - [PDF](#pdf)
    - [HTML](#html)
    - [CSV](#csv)
 - [Download](#download)   
 - [Scheduling](#scheduling)   
 - [Contribute](CONTRIBUTING.md)
 - [Code of Conduct](CODE_OF_CONDUCT.md)
 - [License](https://github.com/malike/elasticsearch-report-engine/blob/master/LICENSE)



## Overview
Once this plugin is installed into elasticsearch search,it exposes the url http://{ip}:9200/_generate, you can run queries on your cluster with the right parameters it would return PDF,HTML or CSV file. 


## Install

_1. Install plugin_

```shell
sudo bin/elasticsearch-plugin install https://github.com/malike/elasticsearch-report-engine/releases/download/5.4.0/st.malike.elasticsearch.report.engine-5.4.0.zip
```
 
_2. Grant permissions_

![Grant Access](/posts/oss-elassticsearch-report-engine/2permissions.png)


_3. Folder Structure_ 
 
Create folders `templates` and `reports` in `ES_HOME`. Store your  `*.jasper` and `*.jrxml`
files in the `templates` folder and pass the templateName as the `template` (with the right 
extension) parameter for `HTML` and `PDF` reports. 

## Usage

#### PDF

**1. PDF Report**

The plugin uses [Jasper Report](https://community.jaspersoft.com/) as core engine for generating PDF reports.
PDF templates can be designed using [iReport Designer](https://community.jaspersoft.com/wiki/ireport-designer-getting-started). This
generates a `jrmxl` file. You can also use the compiled file with the extension `jasper`.

The plugin generates [base64 encoded](https://en.wikipedia.org/wiki/Base64) stream of the PDF report generated once
you pass the name of the template file and the query to fetch data from Elasticsearch.


  _**PDF Sample Request**_ 

  ```shell
    curl -H "Content-Type:application/json" -XPOST "http://localhost:9201/_generate"  -d '{"format":"PDF","fileName":"TEST_REPORT","index":"reportindex","template":"filename.jrxml","from":0,"size":10,"query":"{term:{description:Transaction}}"}'
   ```

  _**Parameters**_ 

  <br/>
        i. *format* : Format of Report **[Required]** <br/>
       ii. *index* : Elasticsearch Index **[Required]** <br/>
      iii. *template* : Jasper Report Template **[Required]** <br/>
       iv. *from* : Offset for querying large data **[Optional]** <br/>
        v. *size* : Size for querying large data **[Optional]** <br/>
       iv. *query* : Query to search Elasticsearch index **[Optional : Defaults to '*' if nothing is passed]**<br/>
       vi. *fileName* : File name **[Optional]** <br/>

  _**Generate PDF  Response**_

 
i. Success
   
   ```json 
   {"status":true,
     "count":1,
     "data": "base 64 encoded string",
     "message":"SUCCESS"
     }
   ```

ii. Missing Required Param

   ```json 
     {"status":false,
      "count":0,
      "data": null,
      "message":"MISSING_PARAM"
      }
   ```

     
iii. Report Format Unknown

   ```json
      {"status":false,
       "count":0,
       "data": null,
       "message":"REPORT_FORMAT_UNKNOWN"
      }
   ``` 

iii. System Error Generating Report

   ```json
      {"status":false,
       "count":0,
       "data": null,
       "message":"ERROR_GENERATING_REPORT"
      }
   ``` 

[Sample PDF](https://github.com/malike/elasticsearch-report-engine/blob/master/SampleReports/TEST_REPORT.pdf)
<br/>

#### HTML

**2. HTML Report**

Just like the PDF report,the HTML also uses [Jasper Report](https://community.jaspersoft.com/) as core engine for generating reports.

HTML Reports provides an alternative for use cases where reports should not be sent as an attached file.

The generates [base64 encoded](https://en.wikipedia.org/wiki/Base64) stream of the HTML report generated.
There's also an option to return the HTML string instead of the base64 encoded string. This 
requires passing `returnAs:PLAIN` as part of the request JSON.

_**HTML Sample Request**_

```shell
  curl -H "Content-Type:application/json" -XPOST "http://localhost:9201/_generate"  -d '{"format":"HTML","fileName":"TEST_REPORT","index":"reportindex","template":"filename.jrxml","from":0,"size":10,"query":"{term:{description:Transaction}}"}'
```  

_**Parameters**_
 
 <br/><br/>
       i. *format* : Format of Report **[Required]** <br/>
      ii. *index* : Elasticsearch Index **[Required]** <br/>
     iii. *template* : Jasper Report Template **[Required]** <br/>
      iv. *from* : Offset for querying large data **[Optional]** <br/>
       v. *size* : Size for querying large data **[Optional]** <br/>
      iv. *query* : Query to search Elasticsearch index **[Optional : Defaults to '*' if nothing is passed]**<br/>
      vi. *fileName* : File name **[Optional]** <br/>
     vii. *returnAs* : How you want HTML file returned. Possible values _PLAIN_ and _BASE64_  **[Optional : Defaults to BASE64]** <br/>


_**Generate HTML Response**_

 
i. Success

   ```json
    {"status":true,
     "count":1,
     "data": "base 64 encoded string",
     "message":"SUCCESS"
     }
   ```

ii. Missing Required Param

   ```json
     {"status":false,
      "count":0,
      "data": null,
      "message":"MISSING_PARAM"
     }
   ```

iii. Report Format Unknown

   ```json
    {"status":false,
     "count":0,
     "data": null,
     "message":"REPORT_FORMAT_UNKNOWN"
    }
   ``` 

iii. System Error Generating Report

   ```json
      {"status":false,
       "count":0,
       "data": null,
       "message":"ERROR_GENERATING_REPORT"
      }
   ```

[Sample HTML](https://github.com/malike/elasticsearch-report-engine/blob/master/SampleReports/TEST_REPORT.html)
<br/>

_*Note: For HTML reports you want returned as HTML string instead of a base64 encoded string.
Send this parameter as part of your default parameters_ : _"returnAs":"PLAIN_

#### CSV

**3. CSV Report**

Unlike the PDF and HTML reports,the CSV option does not use [Jasper Report](https://community.jaspersoft.com/) as core engine for generating reports.
Generating a CSV report uses the query and returns a [base64 encoded](https://en.wikipedia.org/wiki/Base64) of the file.


**_CSV Sample Request_**

```shell
    curl -H "Content-Type:application/json" -XPOST "http://localhost:9201/_generate"  -d '{"format":"CSV","fileName":"TEST_REPORT","index":"reportindex","from":0,"size":10,"query":"{term:{description:Transaction}}"}'
```
  
Parameters
  
  <br/>
      i. *format* : Format of Report **[Required]** <br/>
     ii. *index* : Elasticsearch Index **[Required]** <br/>
    iii. *returnAs* : How you want CSV file returned. Possible values _PLAIN_ and _BASE64_  **[Optional : Defaults to BASE64]** <br/>
     iv. *from* : Offset for querying large data **[Optional]** <br/>
      v. *size* : Size for querying large data **[Optional]** <br/>
     iv. *query* : Query to search Elasticsearch index **[Optional : Defaults to '*' if nothing is passed]**<br/>
     vi. *fileName* : File name **[Optional]** <br/>

_**CSV Sample Response**_


i. Success

   ```json
    {"status":true,
     "count":1,
     "data": "base 64 encoded string",
     "message":"SUCCESS"
     }
   ```

ii. Missing Required Param

   ```json
      {"status":false,
         "count":0,
         "data": null,
         "message":"MISSING_PARAM"
         }
   ```

iii. Report Format Unknown

   ```json
      {"status":false,
         "count":0,
         "data": null,
         "message":"REPORT_FORMAT_UNKNOWN"
         }
   ```

iii. System Error Generating Report

   ```json
      {"status":false,
         "count":0,
         "data": null,
         "message":"ERROR_GENERATING_REPORT"
         }
   ```

[Sample CSV](https://github.com/malike/elasticsearch-report-engine/blob/master/SampleReports/TEST_REPORT.csv)

<br/>

_*Note: For CSV reports you want returned as comma separated values instead of a base64 encoded string.
Send this parameter as part of your default parameters_ : _"returnAs":"PLAIN_

<br/>

## Download

Elasticsearch versions supported by this plugin include :

| Elasticsearch Version| Report Plugin Version | Comments |
| --------------------- | -------- |-------- |
| 5.4.0|[zip](https://github.com/malike/elasticsearch-report-engine/releases/download/5.4.0/st.malike.elasticsearch.report.engine-5.4.0.zip)| Tested   |
| 5.6.10|[zip](https://github.com/malike/elasticsearch-report-engine/releases/download/5.6.10/st.malike.elasticsearch.report.engine-5.6.10.zip)| Tested   |

<br/>

## Scheduling

This plugin can work with an [alerting system](https://malike.github.io/go-kafka-alert/) and a custom [elasticsearch watcher](https://malike.github.io/elasticsearch-kafka-watch/) to send emailed reports to specific contacts. By creating your watcher events in the [custom elasticsearch watch](https://malike.github.io/elasticsearch-kafka-watch/), events would be pushed to Apache Kafka once there's a _hit_, [go-kafka-alert](https://malike.github.io/go-kafka-alert/) listening on Apache Kafka for events would react by emailing embedded **HTML** reports or attached **CSV** or **PDF** reports.

This requires no updates to this plugin but setup and configurations in [go-kafka-alert](https://malike.github.io/go-kafka-alert/) and [elasticsearch watcher](https://malike.github.io/elasticsearch-kafka-watch/)

## Contribute

Contributions are always welcome!
Please read the [contribution guidelines](https://github.com/malike/elasticsearch-report-engine/blob/master/CONTRIBUTING.md) first.

## Code of Conduct

Please read [this](https://github.com/malike/elasticsearch-report-engine/blob/master/CODE_OF_CONDUCT.md).

## License

[GNU General Public License v3.0](https://github.com/malike/elasticsearch-report-engine/blob/master/LICENSE)


