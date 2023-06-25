---
aliases:
- /hadoop/filesystem/2021/02/02/apache-hadoop
categories:
- hadoop
- filesystem
date: '2021-02-02'
description: section2
layout: post
title: Apache Hadoop
toc: true

---

📎 Tacademy

---

## 2.1. Hadoop

- 대용량 데이터를 처리하기 위한 컴퓨터 클러스터에서 동작하는 분산 프로그램 (단일 서버 < 클러스터로 묶은 서버)  
- 기존 RDBMS는 확장을 하려면 서버를 추가 구매해야하나 (scale-up)
- Hadoop은 node(컴퓨터)를 추가함으로써 선형적으로 확장 가능 (scale-out)

 ### 저장 storage: HDFS(Hadoop Distributed File System)  
 ### 연산: MapReduce


- 파일을 block(64M,128M) 단위로 분할  
- block들은 서로 다른 machine에 분산 저장(Mapreduce 처리)  
- block들은 여러 machine(기본 3 replication)에 복제되어 data node에 저장  
- Master node인 name node는 meta data(저장된 위치, block의 file 정보 등) 관리  


---
## 2.2. HDFS Access

1) shell  
2) java api  
3) ecosystem  
    - Flume(network source로 데이터 수집)
    - Sqoop(HDFS와 RDBMS 사이 데이터 전송)
    - Hue(Web 기반으로 browse, upload, download, file view)
 
### file write & read

#### name node의 mete data를 통해 접근
따라서 name node 데몬 중단되면 cluster 접근 불가  
HA로 2개의 name node를 구성하기도 함 (Active/Standby)  
1개 name node 구성시 helper node(Secondary name node)가 추가됨  

|files|blocks| 
|:-:|:--|
|/logs/031512.log | B1,B2,B3 |
|/logs/041213.log | B4,B5 |


|nodes|| 
|:-:|:--|
|node A| B1, B3, B4 |
|node B| B1, B2, B3, B4 |
|node C| B3, B5 |
|node D| B1, B5, B2 |
|node E| B2, B5, B4 |

*node 중 idle이 높은 node부터 접근*


---
## 2.3. 구성요소

1) client
node node를 통해 정보를 받고 이후 data node와 직접 통신  

2) master node(job tracker, name node)
slave node에 대한 정보와 실행할 task 관리  

3) slave node(data node, task node)
client 요청 시 data 전달, task 수행  



![]({{site.baseurl}}/images/mapreduce.png)


### Data Analytics 관점

- Job tracker: task tracker가 수행할 task 스케줄링, 모니터링
- Task tracker: task를 수행하고 job tracker에게 상황 

### Data Storage 관점

- Name node: Meta data 유지, client로 부터 데이터 요청오면 위치 전달
- Data node: 데이터를 HDFS block 단위로 구성, HA를 위한 replication 3 유지, heartbeat를 통한 파일 위치 전달




---
## 2.4. MapReduce

|File| 
|:--|
|Deer, Bear, River|
|Car, Car, River|
|Deer, Car, Bear|

### Mapping: 파일은 한 줄씩 읽어서 데이터 변경
데이터를 key와 value 형태로 pairing하고 list화  

|   |   |
|:--|:--|
|map1|Deer, 1|
|    |Bear, 1|
|    |River, 1|
|map2|Car, 1|
|    |Car, 1|
|    |River, 1|
|map3|Deer, 1|
|    |Car, 1|
|    |Bear, 1|


### shuffling
grouping, sorting  

|   |
|:--|
|Bear, 1|
|Bear, 1|
|Car, 1|
|Car, 1|
|Car, 1|
|Deer, 1|
|Deer, 1|
|River, 1|
|River, 1|


### Reducing: map의 결과 데이터를 집계
aggregating,  후 extract  

|   |   |
|:--|:--|
|red1|Bear, 2|
|red2|Car, 3|
|red3|Deer, 2|
|red4|River, 2|











