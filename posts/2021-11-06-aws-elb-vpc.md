---
aliases:
- /aws/elb/2021/11/06/aws-elb-vpc
categories:
- aws
- elb
date: '2021-11-06'
description: section6
layout: post
title: ELB(Elastic Load Balancing), VPC
toc: true

---

AWS Documentation

---

# 6. Networking & Content Delivery

## 6.1. ELB (Elastic Load Balancing)


### ELB type  

1) Application Load Balancer: HTTP 요청  

2) Network Load Balancer: 네트워크/전송 프로토콜(4계층 - TCP, UDP), high performance, low latency  

3) Classic Load Balancer: EC2 Classic instances에서만 권장  


<br><br>
---

# 7. VPC (Virtual Private Cloud)
보안을 위해 AWS 리소스간 허용을 최소화하고 그룹별로 손쉽게 네트워크를 구성하기 위해 사용하는 AWS 계정 전용 가상 네트워크  

### CIDR (Classless Inter-Domain Routing) block  
Classless Inter-Domain Routing. An internet protocol address allocation and route aggregation methodology.  
[IPv4 CIDR blocks](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing#IPv4_CIDR_blocks)  

## 7.1. 서브넷 (subnet)  
VPC의 IP 주소 범위  
A range of IP addresses in your VPC.

## 7.2. 라우팅 테이블 (routing table)  
네트워크 트래픽을 전달할 위치를 결정하는 데 사용되는 라우팅이라는 규칙 집합  
A set of rules, called routes, that are used to determine where network traffic is directed.  

여러 서브넷에서 동시에 사용할 수 있으며, 이렇게 연결하는 Association 작업 함  

## 7.3. 인터넷 게이트웨이 (internet gateway)  
VPC의 리소스와 인터넷 간의 통신을 활성화하기 위해 VPC에 연결하는 게이트웨이 (NAT의 역할도 수행)  
A gateway that you attach to your VPC to enable communication between resources in your VPC and the internet.  

인터넷으로 라우팅되는 트래픽을 위해 VPC 라우팅 테이블에 target 역할 + 공인 IP주소가 할당된 VPC 내 인스턴스에 대해 NAT를 수행  
인터넷 게이트웨이가 연결된 서브넷: 흔히 public 서브넷 함  

Internet Gateway (IGW) allows instances with public IPs to access the internet

## 7.4. NAT 게이트웨이 (NAT gateway)  
네트워크 주소 변환을 통해 프라이빗 서브넷에서 인터넷 또는 기타 AWS 서비스에 연결하는 게이트웨이(stateful)  
1) NAT gateway: 대역폭 요구 늘어나면 auto scaling  
2) NAT instance: bastion host로 사용해서 public IP 없이 인스턴스 연결 가능  

NAT Gateway (NGW) allows instances with no public IPs to access the internet

## 7.5. VPC 엔드포인트 (VPC endpoint)  
![image](https://user-images.githubusercontent.com/83441376/141411445-a86f1cf8-e2d7-4cc4-a077-b2ae18ae133a.png)  
인터넷 게이트웨이, NAT 디바이스, VPN 연결 또는 AWS Direct Connect 연결을 필요로 하지 않고 PrivateLink 구동 지원 AWS 서비스 및 VPC 엔드포인트 서비스에 VPC를 비공개로 연결, VPC의 인스턴스는 서비스의 리소스와 통신하는 데 퍼블릭 IP 주소를 필요 X, VPC와 기타 서비스 간의 트래픽은 Amazon 네트워크를 벗어나지 X  

Enables you to privately connect your VPC to supported AWS services and VPC endpoint services powered by PrivateLink without requiring an internet gateway, NAT device, VPN connection, or AWS Direct Connect connection.  
Instances in your VPC do not require public IP addresses to communicate with resources in the service.  
Traffic between your VPC and the other service does not leave the Amazon network.  


## 7.6. Security Group  
보안 그룹은 인스턴스에 대한 인바운드 및 아웃바운드 트래픽을 제어하는 가상 방화벽 역할을 하는 규칙 집합  


### Network ACL  
subnet간의 통신 규칙 관리  

### Elastic IP
a static (doesn't change) IPv4 address associated with your AWS account  


<br><br>
---




