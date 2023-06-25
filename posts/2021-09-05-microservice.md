---
aliases:
- /kubernetes/2021/09/05/microservice
categories:
- kubernetes
date: '2021-09-05'
layout: post
title: Microservice
toc: false

---

출처\_Building Cloud private native 전문가 양성과정 교재 + 추가 MSA 
[kihoonkim blog](https://kihoonkim.github.io/2018/03/25/Microservices%20Architecture/first-msa-retro/)

# 10. 클라우드 서비스 소개

## 10.1. 클라우드 서비스 개요  

-   네트워크를 이용해서 사용자들이 원하는 방식으로 서비스를 제공  
-   확장성  
-   사용 모델  
-   분산  
-   개인 시스템의 성능에 구애받지 않음  

### 장점  
 
-   초기 구성 비용 절감  
-   초기 구성 시간 절약  
-   확장성  

### 단점  

-   오랜 기간 사용할 경우 비용 부담 증가  


<br><br>
---
## 10.2. 클라우드 서비스 분류

### SaaS (Software as a Service): 서비스로서 소프트웨어 애플리케이션 서비스를 제공

보통 IaaS, PaaS위에 올라가고, 중앙에서 호스팅되고 있는 소프트웨어를 웹 브라우저 같은 애플리케이션을 통해 사용  
ex) Google Docs  

### PaaS (Platform as a Service): 서비스로서 플랫폼 소프트웨어를 제공

SaaS의 개념을 개발 플랫폼으로 확장한 방식  
플랫폼(OS)를 웹에서 쉽게 빌려 사용  
확장성과 경제적 이유로 On-Premise환경을 Cloud로 확장  
ex) Google APP Engine, OpenShift  

### IaaS (Infrastructure as a service):서비스로서 인프라 자원 제공

Server, Storage, Network를 가상으로 만들어 사용자가 필요한 자원 사용  
관리와 책임이 클라우드 소비자에게 존재

ex) 대부분 퍼블릭 클라우드 서비스 (AWS EC2, S3)  


<br><br>
---
## 10.3. 클라우드 서비스 종류

### Private Cloud

자체적으로 데이터센터 안에 클라우드 환경 구축

-   자산 스스로 보유, 구축
-   기존 IT 인프라 자원 활용 가능
-   소규모로 구축할 때 비용이 높음
-   보안 서비스를 자체적으로 구축해야 함

### Public Cloud

비용을 지불하고 서비스 제공 업체가 구축한 Server, Storage, Network 등 IT Infra 사용

-   공용 클라우드는 가입 형태의 서비스
-   대규모 서비스로 구축 시 비용 절감
-   서비스 제공자가 구축한 보안 서비스 안에서 운용

### Hybrid Cloud

공용 + 사설 클라우드의 장점만 선택해서 사용

-   필요에 의해 데이터나 컴퓨팅 자원의 위치 조절
-   데이터의 중요도와 비즈니스 핵심 업무 여부에 따라 선택 가능


<br><br>
---
## 10.4. GCP Kubenetes

### GCP k8s 환경 설치

1-1) [google cloud sdk 설치](https://cloud.google.com/sdk/docs/install?hl=ko#windows)  
1-2) GCP에서 프로젝트 생성 후, Kubernetes Engine에서 클러스터 생성  
1-3) 윈도우 shell에서 gcloud 초기화 및 연동  
```
PS C:\u01\kubetpl> gcloud init # 초기화
Welcome! This command will take you through the configuration of gcloud.

Settings from your current configuration [default] are:
accessibility:
  screen_reader: 'False'
core:
  account: chwhse@gmail.com
  disable_usage_reporting: 'True'

Pick configuration to use:
 [1] Re-initialize this configuration [default] with new settings
 [2] Create a new configuration
Please enter your numeric choice:  1 # 초기화

Your current configuration has been set to: [default]

You can skip diagnostics next time by using the following flag:
  gcloud init --skip-diagnostics

Network diagnostic detects and fixes local network connection issues.
Checking network connection...done.
Reachability Check passed.
Network diagnostic passed (1/1 checks passed).

Choose the account you would like to use to perform operations for this configuration:
 [1] chwhse@gmail.com
 [2] Log in with a new account
Please enter your numeric choice:  1 # 내 계정 선택

You are logged in as: [chwhse@gmail.com].

Pick cloud project to use:
 [1] affable-beaker-173409
 [2] inner-deck-334800
 [3] my-project-1499687426424
 [4] perceptive-map-173311
 [5] pjt-android
 [6] pjt-shop
 [7] pjt-shop-1502416936516
 [8] Create a new project
Please enter numeric choice or text value (must exactly match list item):  2 # k8s 용 pjt 선택

Your current project has been set to: [inner-deck-334800].

Do you want to configure a default Compute Region and Zone? (Y/n)?  Y

Which Google Compute Engine zone would you like to use as project default?
If you do not specify a zone via a command line flag while working with Compute Engine
resources, the default is assumed.
 [34] asia-northeast1-a
Did not print [39] options.
Too many options [89]. Enter "list" at prompt to print choices fully.
Please enter numeric choice or text value (must exactly match list item):  34 # asia-northeast1-a 존 선택

Your project default Compute Engine zone has been set to [asia-northeast1-a].
You can change it by running [gcloud config set compute/zone NAME].

Your project default Compute Engine region has been set to [asia-northeast1].
You can change it by running [gcloud config set compute/region NAME].

Error creating a default .boto configuration file. Please run [gsutil config -n] if you would like to create this file.
Your Google Cloud SDK is configured and ready to use!

* Commands that require authentication will use chwhse@gmail.com by default
* Commands will reference project `inner-deck-334800` by default
* Compute Engine commands will use region `asia-northeast1` by default
* Compute Engine commands will use zone `asia-northeast1-a` by default

Run `gcloud help config` to learn how to change individual settings

This gcloud configuration is called [default]. You can create additional configurations if you work with multiple accounts and/or projects.
Run `gcloud topic configurations` to learn more.

Some things to try next:

* Run `gcloud --help` to see the Cloud Platform services you can interact with. And run `gcloud help COMMAND` to get help on any gcloud command.
* Run `gcloud topic --help` to learn about advanced features of the SDK like arg files and output formatting
```




<br><br>
---
# 11. Microservice

## 11.1 Microservice 이해 및 구축 방법론
서비스를 비즈니스 경계에 맞게 세분화 하고, 서비스 간 통신은 네트워크 호출을 통해 진행하여 확장 가능하고 회복적이며 유연한 어플리케이션을 구성하는 것  

### Microservice 특징  
기존의 Monolithic 방식은 변화 대응이 어렵고, 새로운 기능 추가 및 업데이트가 어려움  

- Loosely Coupled: 장애 격리가 쉽고 신뢰도가 높음  
하나의 서비스 변경될 때, 다른 서비스 변경 X  
시스템 변경없이 서비스 변경 및 배포 가능  
testability  

- High Cohesion: Bounded_Context[^1]라는 확실한 경계가 있고 구체적인 책임을 나눔  
single responsible하에 하나의 마이크로 서비스 안에 함께 변경되는 것들은 같이 모아둠  
설계 시, 업무 도메인 정의 후 해당 업무 도메인 내에서 변경을 고려해서 마이크로 서비스 분리  

- Resilience (회복성)  
하나의 서비스 장애가 다른 서비스로 전이(cascade failure)되지 않음  
전체 장애 차단하고 기능은 저하 시킴(Circuit breaker routing)  

- Scaling (확장성)  
app 단위 배포 및 확장을 의미하는 전통적인 scale out이 아니라,  
마이크로 서비스 단위 배포 및 확장하는 scale out의 개념  

- Tech Heterogeneity (기술 이기종성)  
서비스마다 필요한 기술 적용 (성능, 데이터 모델, 구현 용의성..)  
너무 많은 기술 스택은 오히려 부담

- Easy Deployment (배포 용이성)  
전체 build&deploy(다같이 야근)가 아닌 서비스 단위의 build&deploy  
테스트 코드 혹은 TDD 필요  
컨테이너 환경에 적합  

- deploy/rollback 시 발생하는 risk 감소 (Blue-Green deploy로 점진적 변경)

- Autonomous (자율성)  

- resource 낭비 줄임 (부분 scale out)  


## MSA 고려시 주의사항  

### 생산성 하락  
기존 시스템이 복잡하지 않을 경우, MSA 방식에서 요구되는 요소들이 오히려 생산성을 떨어뜨림  
작은 서비스 들이 흩어져 있는 구조로 모니터링 어려움  
산재된 로그 분석 어려움  
admin이 마이크로 서비스에 대한 이해가 부족하면 장애 대처 X  

### 대안  
컨테이너 기술 및 컨테이너 오케스트레이션 활용하고,  
**기존 모놀리스 환경에서:**  
소스코드를 여러 라이브러리로 나누어서 유틸 생성하고 재사용  
라이브러리를 통해 Java에 Module 컨셉을 부여해서 여러 버전의 모듈관리 (OSGI/Erlang)  



### Microservice 구성 전략  

- 비즈니스 개념에 따른 모델링  
도메인 경계 정의, 기술보단 비즈니스 경계로, 서비스는 두고 DB 스키마부터 단계적 분리  

- 공통 자동화 문화 적용  
테스트 자동화 (CI[^2]/CD[^3]) 및 Custom Image 환경 구축  

#### 서비스의 크기?
Bounded Context 단위  
하나의 Aggregate를 갖는 것이 좋음  
그렇게 되려면 서비스가 굉장히 잘게 나눠져야 하고, 한 팀이 여러 서비스를 관리  
-> 코드 repository가 많이 지고 각각의 빌드 파이프라인도 많이 진다. 
-> 여러 서비스를 개발하기위해 IDE도 자주 스위칭을 해야 할 것이다. 
우리는 크게 4,5 개 정도의 도메인을 식별했고, Rest URI 수준에서 분리
(근본적인 목적은 여러팀이 Agile하게 일할 수 있도록 만드는 것)


#### Blue-Green 배포
![blue-green-deployment](https://www.redhat.com/cms/managed-files/blue-green-deployment-model.gif)  
Blue: 이전 버전  
Green: 신규 버전  

#### Canaria 배포
![canary-deployment](https://user-images.githubusercontent.com/83441376/139980171-e44febeb-1bee-4047-804d-79f8ebe2fd9e.png)

[배포관련 참고](https://harness.io/blog/blue-green-canary-deployment-strategies)





---
<br><br>

### Micro Service Architecture
[MSA architecture 참고 링크](https://www.nginx.com/blog/introduction-to-microservices/)


## 11.2 MSA component


### Edge Server (API Gateway)
**Zuul**(w/Ribbon), Spring Cloud GW  

![image](https://user-images.githubusercontent.com/83441376/140001251-f2111415-a76d-42ac-a2cc-fced85968fc9.png)  
모든 클라이언트 요청에 대한 end point를 통합하는 서버(프록시 서버처럼 동작)  
한 서비스에 한개 이상의 서버가 존재하기 때문에 이 서비스를 사용하는 클라이언트 입장에서는 다수의 end point가 생기게 되며, 
end point를 변경이 일어났을때, 관리하기가 힘들다.  
이런 문제를 해결하기 위해 서비스들의 end point를 하나로 묶을 수 있는 API 게이트웨이가 필요  
각 서비스를 직접 호출하지않고 모든 요청이 API 게이트웨이를 통하게 만드는 것  
하나의 요청에 여러 서비스를 호출한 후 하나의 결과로 취합  


요청에 따라 필요한 서비스로 라우팅  
모든 서비스의 API 를 노출하는 대신 필요한 API 만을 노출해서 캡슐화  
클라이언트 별로 다른 API 를 제공  
하나의 요청에 필요한 서비스를 각각 호출해 결과를 모아서 응답  
내부에서 사용하는 프로토콜이 다를 경우 외부에는 웹 친화적인 프로토콜(HTTP, WebSocket 등)으로 변환  
클라이언트와의 통신을 줄일 수 있고, 클라이언트의 코드도 단순  
인증 및 권한, 모니터링, load balancing, caching, logging 등 추가적인 기능  


하나의 엔트리 포인트를 갖는 것은 장점이자 단점  

API 게이트웨이에서 하는 역할이 많고, 게이트웨이에 장애가 나면 서비스 전체가 사용이 불가능 -> circuit breaker  
각 서비스의 API 를 수정하면 API 게이트웨이를 함께 수정해야 합니다. 이는 개발 과정에서 병목(bottleneck)이 되어 개발 과정 지연  
API 게이트웨이 또한 개발하고 유지보수해야 할 대상  



### Load Balancer: 
**Ribbon**  

- ServerList: 로드 밸런싱 대상 서버 목록  
configuration을 통해 static 하게 설정 가능  
eureka 등을 기반으로 dynamic하게 설정 가능  
- Rule: 요청을 보낼 서버를 선택  
Round Robbin - 한 서버씩 돌아가며 전달  
Available Filtering - 에러가 많은 서버 제외  
weighted Response Time - 서버별 응답 시간에 따라 확률 조절  
- Ping: 서버list가 살아있는지 체크  
static, dynamic 모두 가능  

### Service Discovery (Service Registry):  
**Eureka**, Spring Cloud LB  

![image](https://user-images.githubusercontent.com/83441376/140001039-1e4c6908-fcfc-4b68-b2ba-effbf77107d8.png)  

클라우드에서 인스턴스는 동적으로 할당되기 때문에 IP주소나 포트 정보가 정해지지 않은 데다가 오토스케일링도 일어나고 중지되고 복구되면서 네트워크 위치가 계속해서 바뀌게 됩니다.

따라서 클라이언트나 API 게이트웨이가 호출할 서비스를 찾는 매커니즘이 필요하고 이를 서비스 디스커버리(Service Discovery)라고 합니다. 이러한 로직을 구현하는 쪽에 따라서 두 가지 방식으로 나뉩니다.

- 클라이언트 사이드 디스커버리 패턴(Client-Side Discovery Pattern)  
- 서버 사이드 디스커버리 패턴(Server-Side Discovery Pattern)  


API 게이트웨이는 각 서비스를 호출하기 위해 IP 주소와 포트를 알고 있어야 합니다. 기존 환경에서는 이러한 서버의 위치가 고정이라 문제가 없지만, 클라우드 기반에서는 각 서비스가 동적으로 할당된 서버에 배포되면서 해당 서비스의 위치를 파악하는 것이 어려워졌습니다. 이렇게 해당 서비스의 위치를 찾는 기술을 서비스 디스커버리(Service Discovery)라고 합니다. API 게이트웨이는 서버 사이드, 혹은 클라이언트 사이드 기준으로 서비스 디스커버리를 구현할 수 있습니다.


### Circuit Breaker:  
Hystrix, Resilence4j  

- 회로 차단기 (Circuit Breaker)  
이름 그대로 전기선에 유입된 과전류를 감지하는데 이와 마찬가지로 하위 컴포넌트가 과전류에 손상되지 않도록 보호  
원격 자원에 대한 모든 호출을 모니터링하고 임계점까지 실패한다면 회로 차단기가 활성화 되어 빠른 실패를 유도, 그리고 문제가 있는 원격 자원을 더 이상 호출하지 못하게 함  
- 폴백 (Fallback)  
원격 서비스에 대한 호출이 실패할 때, 대체 코드(fallback method)를 실행  
- 벌크헤드 (Bulkhead)  
서비스에 대한 동시 요청 수를 제한  
원격 자원에 대한 호출을 자원 별 스레드 풀로 분리  
특정 원격 자원 호출이 느려지더라도 다른 자원들에 대한 호출은 영향이 없음  


### Config Server:  
Spring Cloud Config Server    

Spring Cloud Config는 분산 시스템에서 환경설정을 외부로 분리하여 중앙에서 관리할 수 있는 기능을 제공  
Application data(환경별 구성 데이터)를 microservice와 분리  
따라서 마이크로서비스 인스턴스가 아무리 많더라도 항상 동일한 구성을 유지할 수 있음  
Spring Cloud Config에는 고유한 관리 저장소가 있지만 아래의 오픈 소스 프로젝트와도 통합할 수 있음  

- Spring Config Server : 공통 환경설정을 가지고 있는 Config Cloud Server
환경설정(name-value pair, YAML 파일)을 위한 HTTP, 리소스 기반 API
속성 값 암호화 및 암호 해독 (대칭 또는 비대칭)
\@EnableConfigServer 어노테이션을 사용하여 쉽게 Spring Boot 어플케이션에 적용

- Spring Config Client : 공통 환경설정을 받아 사용하는 Application Server
Config Server에 붙어 원격 속성 소스로 Spring 환경 초기화
속성 값 암호화 및 암호 해독 (대칭 또는 비대칭)



### Message Stream:  
RabbitMQ, Kafka

Spring cloud bus: 동적으로 config 변경을 적용하기 위한 MOM[^4]을 구성한 MQ(Message Queue) Handler

(1) MQ(Message Queue)에 Publisher(=config server)와 Subscriber(마이크로서비스)를 등록  
(2) config 변경 정보를 MQ에 전송(Publish Message)  
(3) 각 마이크로서비스에서 config 동적 반영(Reload Config)  


### Monitoring: Zipkin  








<br><br>
---
## 11.3 Spring Cloud를 활용한 Microservices 개발

### Deploying Microservices: Spring Cloud vs. Kubernetes
![image](https://user-images.githubusercontent.com/83441376/145658868-23bba2c5-97a6-4232-9a3f-4aebc21fa15e.png)
  

### Spring Cloud
Spring Cloud는 Microservce Architecture 환경에서의 개발과 배포, 운영을 쉽게 구성할 수있도록 지원하는 Spring Boot 기반의 Framework  
Cloud Native한 패턴과 메커니즘을 제공   
많은 서비스들이 Java 기반의 Spring Boot로 개발되기에, Java 기반 MSA를 구축하고자 할 시 최적의 선택  


#### Spring Boot  
기존 Spring 이 가지고 있던 문제들을 해결하여, 보다 빠르게 상용 Spring Application을 개발하고자 시작  
Conventioin over Configuration (설정보다 관례)  
- Library 관리 자동화 (Starter 이용)  
- 자동 environment configuration (Annotaion 이용)  
- 내장 Tomcat 및 JAR 모델 지원 ( Web Application이어도 JAR로 packaging runtime 지원)  



#### Annotaion  
마치 코드의 메타데이터 같은 역할을 하여, Compile, Runtime 과정에서 코드를 어떻게 처리할 것인지 나타냄  
Annotation을 통해 Spring Boot가 지향하는 Convention over Configuration을 가능하게 함  
```
@SpringBootApplication
public class TestApplication {
  public static void main (String[] args) {
    SpringApplicatoin.run(MyApplication.class, args);
  }
}
```


<br><br>
---




[^1]: 내부는 독립적으로 차단되고 외부와 연결하려면 interface를 통해서 소통 가능
[^2]: 자동화된 빌드 및 테스트가 수행된 후, 개발자가 코드 변경을 repository에 정기적으로 병합하는 개발 방식

[^3]: 코드 변경 사항을 repository에서 고객이 사용 가능한 production 환경까지 자동으로 release하는 개발 방식

[^4]: 분산 시스템 환경에서 독립된 서비스 간에 비동기 방식으로 메세지를 전송/수신할 수 있게 도와주는 인프라

