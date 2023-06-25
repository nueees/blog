---
aliases:
- /gcp/datastudio/2021/10/03/data-lifecycle-analyze-visualize
categories:
- gcp
- datastudio
date: '2021-10-03'
description: section3,4
layout: post
title: Data Lifecycle - Analyze & Visualize
toc: false

---

[Cloud Architecture Center](https://cloud.google.com/architecture/data-lifecycle-cloud-platform#process_and_analyze)

---

![]({{site.baseurl}}/images/post/data-lifecycle-4.svg)

# Process and analyze

## 3.1. Processing large-scale data
source systems (Google Cloud Storage, Bigtable, Google Cloud SQL) 읽어온 큰 데이터를 처리하고
정규화하고 집계함.
데이터 양이 커서 클러스터로 분산 처리하거나 소프트웨어 툴들의 도움을 받음

### Dataproc
cluster에서 실행 (auto scaling)  
기존 Hadoop, Hive, Spark 에플리캐이션과 연동  
use case: Log processing, Reporting, On-demand Spark clusters, Machine learning  

### Dataflow
Serverless(cluster X), parallel 처리 (No-Ops)  
Spark나 Hadoop과 연동이 아닌 새로운 데이터 pipeline 처리  
use case: MapReduce 안쓰는 parallel processing, User analytics, Data science, ETL, Log processing  

### Dataprep
UI-Driven Data Preparation (No-Ops, 필요시 scaling 가능)
use case: Machine learning, Analytics  

#### connect Dataprep to BigQuery  

1) Create Flow
2) Add dataset (import)
3) Select BigQuery (left pane) & Create dataset
4) Import & Add to Flow

#### inspect, process data

1) Edit Recipe
dataset의 sample을 Transformer view에서 확인 가능 (visualization)  


#### execute job to load BigQuery

1) click Run in Transformer page  
2) click Edit on Create-CSV  
3) select BigQuery then create table  
4) name output table  
5) click Update  
6) click Run  


job history에서 monitoring 가능



---
## 3.2. Analyzing and querying data

### BigQuery
앞서 store 할 때 뿐만 아니라 분석할 때도 사용  
use case: User analysis(adtech, clickstream, game telemetry), Device and operational metrics(IoT), BI


### Machine learning
처리된 결과를 확대시키거나 data-collection 최적화를 제공하기도 하고 결과 예측도 함  

- 음성 인식  
- 자연어 처리  
- 번역  
- 동영상 자동 분석  
- AI 플랫폼 (TensorFlow)  





---
# Explore and visualize

---

[Cloud Architecture Center](https://cloud.google.com/architecture/data-lifecycle-cloud-platform#explore_and_visualize)

---

## 4.1. Explore and visualize

### Datalab
interactive한 web 기반 툴  
pandas, numpy, scikit-learn 등의 다양한 toolkit 지원  


### Data science ecosystem  
Datalab 말고도, web 기반 툴인 Apache Zeppelin 지원
R 사용하면 Rstudio Server나 Microsoft ML Server 지원
Scala나 Java 사용하면 Jupyter 지원


## 4.2. Visualizing business intelligence results

### Looker
BI platform

### BI Engine
analysis service 관리  

### Sheets
Spreadsheet visualization  

### Data Catalog
Data discovery and metadata management  




### Data Studio
Dashboarding and visualization  
over 200 connectors ( Google Analytics, BigQuery, Sheets, and external data sources...)

---
#### Practice
1) Intro  
- connect bigquery and data studio  
- visualize my data in data studio  


2) getting set up  
- accesse public datasets ( [catalog](http://g.co/cloud/marketplace-datasets) )  
- set up my GCP ( [my console](http://console.cloud.google.com/) )  

3) connecting data studio and bigquery  

(1) Open [data studio](https://datastudio.google.com/ )  
(2) Start with a Template (Blank template)  
(3) Add a data to report Connect to BigQuery (Allow permission to view BigQuery data)  
(4) Add table "san_francisco_311" (in public datasets)   
(5) Click Manage added data sources under Resources  
(6) Edit table fields of "311_service_requests"   
(7) "latitude","longitude" field from text to Latitude, Longitude in Geo  

4) creating visualizations   
  
- Click "Add a chart" and select "Treemap"  
- Place the "Treemap" chart and change the parameter ("category" field)  

5) building a dashboard  

- Click "Add a chart", select "Google Maps" and change its parameters (Dimension/Tooltip : "neighborhood", Metric/Bubble size: "record count")  
- Click "Add a chart" and select "Scorecard"  
- Select "Add a filter" in the Data panel and create filter ("Street and Sidewalk cleaning" in "category" field)  

6) creating filters  
- Click "Filter Control" (set filter dimension to "neighborhood")  
- Click "Arrange" and select "Make page-level"  

7) test it and share it  
- [practice_data_studio](https://datastudio.google.com/s/h0mhT-tvKh4)  


<!-- 
#### Practice02  

1) Intro  
- How a Google Data Studio Community Connector works  
- How to use Google Apps Script to build a Community Connector  
- How to use Community Connectors in Data Studio  

2) Community Connectors  
![image](https://user-images.githubusercontent.com/83441376/140704891-ca0f7c14-9141-4169-8a93-53a05047b9c8.png)  
Fetch data through Web APIs, JDBC APIs, flat files (CSV, JSON, XML), and Apps Script Services.  
ex) after publishing a package on npm, then to tack the download count -> npm package download counts API  

![image](https://user-images.githubusercontent.com/83441376/140705597-1f6c8bad-2e89-4ce9-882e-9ba4c4eaac61.png)  
Work flow:   
- getAuthType()  
- getConfig()  
- getSchema()  
- getData()  
 -->

---














