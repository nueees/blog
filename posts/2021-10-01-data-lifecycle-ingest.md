---
aliases:
- /gcp/2021/10/01/data-lifecycle-ingest
categories:
- gcp
date: '2021-10-01'
description: section1
layout: post
title: Data Lifecycle - Ingest
toc: false

---

[Cloud Architecture Center](https://cloud.google.com/architecture/data-lifecycle-cloud-platform#ingest)

---

![]({{site.baseurl}}/images/post/data-lifecycle-1.svg)


## 0.1. Orchestration 

### Cloud Composer
Apache Airflow에 내장된 workflow orchestration service (위 그림 서비스들을 관리)

![]({{site.baseurl}}/images/post/data-lifecycle-2.svg)


## 1.1. Ingesting app data
Data from app events, such as log files or user events, is typically collected in a push model, where the app calls an API to send the data to storage.  

1) Writing data to a file  
Cloud Storage에서 CSV file로 빼내거나
BigQuery로 DW에서 데이터를 뽑거나

2) Writing data to a database  
Cloud SQL 혹은 NoSQL인 Datastore 혹은 Bigtable로 데이터를 씀


### Cloud SQL
Fully managed **relational database** service for MySQL, PostgreSQL, and SQL Server  


### Datastore
a highly scalable **NoSQL database** for your *web and mobile applications*  


### Cloud Bigtable
A fully managed, scalable **NoSQL database** service for *large analytical and operational workloads*  


### Cloud Firestore
Easily develop rich applications using a fully managed, scalable, and serverless **NoSQL document database**  


---
## 1.2. Ingesting streaming data
The data consists of a continuous stream of small, asynchronous messages.  

1) Streaming data as messages  
streaming 데이터를 Pub/Sub으로 보내서 받은 message를 processing 혹은 storing  



### Pub/Sub
Real-time **messaging** and ingestion for event-driven systems and streaming analytics 



---
## 1.3. Ingesting bulk data
Large amounts of data are stored in a set of files that are transferred to storage in bulk.  

1) Scientific workloads  
Genetics data가 있는 VCF text file을 Cloud Storage에 upload (추후 Cloud Life Sciences로 처리)


2) Migrating to the cloud  
on-premise Oracle DB에서 Google Cloud SQL DB로 migration 할 때

3) Backing up data  
Cloud Storage Transfer Service를 통해 replicating해서 백업 할 때

4) Importing legacy data  
기존 legacy big data를 Bigquery (DW)로 import 할 때 



### Cloud Storage
Object Storage  

### BigQuery  
Serverless, highly scalable, and cost-effective multicloud data warehouse designed for business agility  












