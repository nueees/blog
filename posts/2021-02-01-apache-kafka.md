---
aliases:
- /kafka/message_broker/queue/2021/02/01/apache-kafka
categories:
- kafka
- message_broker
- queue
date: '2021-02-01'
description: section1
layout: post
title: Apache Kafka
toc: true

---

📎 [kafka documentation](https://kafka.apache.org/documentation/)

---

## 1.1. Kafka
a distributed, partitioned, and replicated commit log service  

소스 애플리케이션(mysql, oracle, nosql,.../app/was/...)과 타겟 애플리케이션(hadoop, search engine, monitoring, email...)개수가 많아지고,프로토콜 포맷 파편화가 심해져 유지보수가 어려워짐

이러한 복잡함을 해결하기 위해 중간에서 소스 애플리케이션과 타겟 애플리케이션 연결을 느슨하게 함 (scale out)

프로듀서가 큐에 적재하고 컨슈머가 가져가는 구조
<!-- 클릭로그, 결제로그 -> 로그적재, 로그처리 -->

- Kafka Client: kafka와 데이터를 주고 받기 위해 사용하는 (producer, consumer, admin, stream 등) API


---
## 1.2. Brocker & Cluster

- brocker: Kafka application 서버 단위
- cluster: 3대 이상의 brocker로 구성

- zookeeper: metadata 저장하여 brockers 관리
- controller: n대 brocker 중 마스터 brocker (allocate partition, health check)


---
## 1.3. Topic

![]({{site.baseurl}}/images/post/kafka-basic-concepts.png)
메시지(데이터) 분류 단위
n개의 파티션 할당 가능
각 파티션 마다 고유 offset 가짐
메시지 처리순서는 파티션 별로 관리
- partition: 레코드를 담고 있고 replication 가능 
- ISR(In-Sync Replica): replication(leader+follower)의 sync로 된 묶음
- segment: 실제로 메시지가 저장되는 파일 시스템 단위
![]({{site.baseurl}}/images/post/topic.JPG)
![]({{site.baseurl}}/images/post/topic2.JPG)

---
## 1.4. Producer
메시지(레코드)를 brocker로 송신 API

### Role
Topic에 해당하는 메시지를 해당 파티션에 offset과 함께 기록
- Publishing: 특정 topic으로 데이터 송신
- 처리 실패 시 재시도
- key 사용시, 해당 partition으로 데이터 매핑


---
## 1.5. Consumer
메시지(레코드) brocker에서 수신 API
다수의 consumer를 group화 할 수 있음

### Role
- Polling: Topic의 partition으로 부터 메시지(레코드)를 가져감
- Commit: Partition offset 위치를 기록
- Parallel Processing: Consumer 여러개일 경우, Consumer group을 통해 병렬 처리

#### Offset number:
파티션 내 갖게되는 고유 키
토픽 별, 파티션 별로 고유하게 지정됨
컨슈머가 데이터를 어디까지 읽었는지 확인하는 용도
데이터를 읽은 곳까지 commit

컨슈머 이슈가 발생해도 \_\_consumer_offset 토픽에 저장된 지점 확인(High availablility)

![]({{site.baseurl}}/images/post/multi_consumer.JPG)
![]({{site.baseurl}}/images/post/different_consumer_group.JPG)


**병렬처리를 위해서는 consumer 개수는 partition 개수보다 적거나 같아야 함**


---
## 1.6. Kafka Streams
데이터 변환(transformation) API

- stateful, stateless와 같은 상태 기반 stream 처리 가능
- stream api와 DSL(Domain Specific Language) 동시 지원
- Exactly-once 처리, HA 특징
- Kafka security(acl, sasl 등) 지원
- stream 처리를 위한 별도 cluster(yarn 등) 불필요

### ESP (Event Stream Platform)
대표적으로 Kafka, Kinesis, Flink, Spark, Storm, Logstash...

### Kafka Streams API 
- Source processor
- Sink processor






---
## 1.7. Kafka Connect
Kafka client가 아닌 connect를 통해 데이터를 import/export 가능
code 없이 configuration으로 데이터 migration 시 사용






