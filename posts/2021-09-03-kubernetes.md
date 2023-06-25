---
aliases:
- /kubernetes/2021/09/03/kubernetes
categories:
- kubernetes
date: '2021-09-03'
layout: post
title: Kubernetes
toc: false

---

[dangtong76 blog](https://dangtong76.github.io/kubernetes/)
[cloudacode medium](https://medium.com/@cloudacode/쿠버네티스-이해하기-production-grade-container-orchestration-system-3df8cd25fdb3)

---


# 5. Container Cluster

## 5.1. Kubernetes 소개

docker - 단일 시스템에서만 다수의 컨테이너 관리  
-> 다수의 시스템과 애프리케이션 설정을 쉽게 설정하고 유지보수할 수 있는 방식인 오케스트레이션(Ochestration)이 필요  

kubernetes - docker, rtk 같은 Container의 Ochestration 도구  

**_다중 컨테이너 관리를 위한 docker-compose를 설치해야 함._**

쿠버네티스는 클러스터 구성해서 오케스트레이션을 통해 컨테이너를 자동으로 관리, 2개 이상 시스템에서 관리 가능  
관리 대상을 object라고 함 - pods(컨테이너 논리적 구성 단위)와 controller(pods를 한번에 관리)로 구성(application workload)

### Kubernetes 사용 이유  

• 기민한 애플리케이션 생성과 배포: VM 이미지를 사용하는 것에 비해 컨테이너 이미지 생성이 보다 쉽고 효율적  
• 지속적인 개발, 통합 및 배포: 안정적이고 주기적으로 컨테이너 이미지를 빌드해서 배포할 수 있고 (이미지의 불변성 덕에) 빠르고 쉽게 롤백할 수 있다  
• 개발과 운영의 관심사 분리: 배포 시점이 아닌 빌드/릴리스 시점에 애플리케이션 컨테이너 이미지를 만들기 때문에, 애플리케이션이 인프라스트럭처에서 디커플된다  
• 가시성은 OS 수준의 정보와 메트릭에 머무르지 않고, 애플리케이션의 헬스와 그 밖의 시그널을 볼 수 있다  
• 개발, 테스팅 및 운영 환경에 걸친 일관성: 랩탑에서도 클라우드에서와 동일하게 구동된다  
• 클라우드 및 OS 배포판 간 이식성: Ubuntu, RHEL, CoreOS, on-prem, Google Kubernetes Engine 및 다른 어디에서든 구동된다  
• 애플리케이션 중심 관리: 가상 하드웨어의 OS에서 애플리케이션을 구동하는 수준에서 OS의 논리적인 자원을 사용하여 애플리케이션을 구동하는 수준으로 추상화 수준이 높아진다  
• 느슨하게 커플되고, 분산되고, 유연하며, 자유로운 마이크로서비스: 애플리케이션은 단일 목적의 머신에서 모놀리식 스택으로 구동되지 않고 보다 작고 독립적인 단위로 쪼개져서 동적으로 배포되고 관리될 수 있다  
• 자원 격리: 애플리케이션 성능을 예측할 수 있다  
• 자원 사용량: high efficiency high density  

클라우드 네이티브 구축 후에는 관리포인트를 줄이기 위해 managed service를 쓰는게 좋음  
application 기능에 집중할 수 있도록 infra 영역을 추상화  

deploy 시점이 아닌, build 시점에 jar 파일이나 source 코드를 생성하는게 아니고
container 이미지를 아얘 새로 만드는 추세  

### Kubernetes 기능  

컨테이너 플랫폼, 마이크로서비스 플랫폼, 이식성있는 클라우드 플랫폼 제공  

- Automatic Binpacking  
- Storage Orchestration  
- Secret & Configuration Management  
- Horizontal Scaling  
- Service Discovery & Load Balancing  
- Self Healing  
- Batch Execution  
- Automatic Rollbacks & Rollouts  


CI/CD 파이프라인, 애플리케이션 레벨의 서비스, 로깅, 모니터링, 경고 솔루션 등을 제공하지 않음  

---

### CNCF (Cloud Native Computing Foundation)  
CNCF에 인큐베이팅 되어 첫번째로 graduated한 kubernetes 프로젝트  

밴더들이 클라우드 네이티브 후원하는 이유?  
혼돈스러운 컨테이너와 관련된 다양한 기술적인 문제들을 오픈소스로 해결하는 하는 것을 목표  

ckad 사용자 측, cka 관리자 측(자체적으로 운영 관리 가능)  
#### Graduated & incubating projects  
#### Sandbox projects  
#### Archived projects  

      
---

## 5.2. Kubernetes 기본 구성단위  
[platform9 blog](https://platform9.com/blog/kubernetes-enterprise-chapter-2-kubernetes-architecture-concepts/)  

![image](https://user-images.githubusercontent.com/83441376/145669058-fa4c2505-574e-4827-936a-7045130486ca.png)  

![image](https://user-images.githubusercontent.com/83441376/145669061-cf579bad-4982-4712-9c1f-690fac7be012.png)  

### 1) Control Plane  
1-1) API 서버 : 쿠버네티스 오브젝트에 대한 데이터를 설정하고 검증. REST 서비스 제공  
1-2) 스케줄러 : 클러스터의 노드에 파드를 배포 하는 역할. feasibility 계산 하여 배포  
1-3) 컨트롤러 메니저 : 제어 루프를 이용해 설정된 상태를 지속적으로 유지하도록 함  
1-4) [ETCD](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/#stacked-etcd-topology) : 클러스터 오브젝트의 데이터를 저장  

![image](https://user-images.githubusercontent.com/83441376/145669066-453d02b6-f1b2-4e0c-b114-e409224ab725.png)  

### 2) Nodes  
2-1) 컨테이너 런타임 : 컨테이너를 실행하는 엔진 (Docker, rtk 등)  
2-2) Kubelet : API 서버와 통신하며 워커 노드의 컨테이너를 관리  
2-3) kube-proxy : 애플리케이션의 구성 요소간에 네트워크 트래픽을 분산  

![image](https://user-images.githubusercontent.com/83441376/145669231-ba8555e0-1ac2-4df2-934a-1454f0b71c00.png)  



---

## 5.3. k8s 사전 설치

### 1) choco 패키지 관리도구 설치  
```
] Set-ExecutionPolicy Bypass -Scope Process -Force; \[System.Net.ServicePointManager\]::SecurityProtocol = \[System.Net.ServicePointManager\]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('[https://community.chocolatey.org/install.ps1'](https://community.chocolatey.org/install.ps1')))
```
### 2)  vagrant 설치혹은 vagrant.exe 파일 다운로드 후 설치 가능  
```
] choco install vagrant
```
### 3)  하이퍼바이저 설치  
    virtualbox 설치 (확장팩)
### 4)  vagrant 명령어로 구성요소 설치  
    vagrant plugin install vagrant-hostmanager  
    vagrant plugin install vagrant-disksize  
    확인: vagrant plugin list윈도우의 경우 직접 작성하면 확장자이름이 자동설정될 수 있으니 주의
```
] vagrant box add ubuntu\/bionic64 
] vagrant up => Vagrantfile 이 있는 곳에서 실행
```
### 5) 가상머신에 쿠버네티스 배포  
ssh 설정
```
] ssh-keygen 
] ssh-copy-id node1 (노드 모두 입력) localhost 도 설정
```

---

## 5.4. k8s 설치

### 1) 관련 패키지(python) 및 git 설치  

```
] sudo apt update 
] sudo apt upgrade -y 
] sudo apt install -y python3 python3-pip git 
] git clone --single-branch --branch release-2.14 https://github.com/kubernetes-sigs/kubespray.git 
] cd kubespray/ 
] sudo pip3 install -r requirements.txt
```

### 2) 인벤토리 수정  

```shell
] cp -rfp inventory/sample/ inventory/mycluster 
] vim inventory/mycluster/inventory.ini 
[all] 
node1 ansible_host=192.168.56.21 ip=192.168.56.21 
node2 ansible_host=192.168.56.22 ip=192.168.56.22 
node3 ansible_host=192.168.56.23 ip=192.168.56.23 
controll-plane ansible_host=192.168.56.11 ip=192.168.56.11 
[all:vars] ansible_python_interpreter=/usr/bin/python3 
[kube-master] controll-plane 
[etcd] controll-plane 
[kube-node] node2 node3 node1 
[calico-rr] 
[k8s-cluster:children] 
kube-master kube-node calico-rr

] sudo vim /etc/hosts
192.168.56.11   controll-plane.example.com controll-plane
192.168.56.21   node1.example.com node1
192.168.56.22   node2.example.com node2
192.168.56.23   node3.example.com node3

] vim inventory/mycluster/group_vars/k8s-cluster/addons.yml
metrics_server_enabled: true
ingress_nginx_enabled: true
metallb_enabled: true
metallb_ip_range:
  - "192.168.56.50-192.168.56.99"
metallb_protocol: "layer2"

] vim inventory/mycluster/group_vars/k8s-cluster/k8s-cluster.yml
kube_proxy_strict_arp: true
```

### 3) 플레이북 실행  
```
] ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b
```

### 4) kubectl 설치  
```
] curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" 
] curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256" 
] echo "$(<kubectl.sha256) kubectl" | sha256sum --check 
] sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl 
] kubectl version --client
```

<br><br>
---

## 5.5. k8s Architecture

![]({{site.baseurl}}/images/post/docker_5_1.jpg)  

kube-api-server를 통해서 대부분의 작업을 함  
kube-controller-manager : on-premise 환경에서 각 리소스 관리  
cloud-controller-manager : cloud 환경에서 밴더사 자체적으로 관리

### 마스터(master)

cluster를 구성하기 위한 핵심 요소들의 모음  
cluster 조정을 위한 control plane 제공  
노드에 작업 분배하는 scheduling 작업  
cluster evnet 감지하고 대응  
운영환경에서는 multi master 환경을 구성 (FT)

1) API 서버(kube-apiserver)  
kubernetes API를 노출하는 component
kubernetes object 관리, 제어를 위한 front-end

2) etcd  
cluster의 meta 정보 정장  
key-value 형태로 저장  
cluster configuration 정보 보관하고 있으므로 백업 필수

3) scheduler  
cluster 내 생성되는(혹은 배정되지 않은) pod를 감지하고 그것을 구동할 노드를 선택  
resourse 상태, HW/SW/Policy 제약, Affinity 등 다양한 기준에 따라 배치 결정

4) cube-controller-manager

-   node controller: 노드 관리, 노드 다운시 대응
-   replication controller: 복제 컨트롤러를 사용하는 모든 object관리, 적정 pod 수 유지
-   endpoint controller: service와 pod 연결 (물리적 시스템과 컨테이너 내부 어플리케이션 연결)
-   service account & token controller: 쿠버네티스 namespace, account, token 등 인증 관련된 것 담당

5) cloud-controller-manager

AWS, GCP 등 각 밴더 별 클라우드 서비스 관리  

-   node controller: 노드 관리, 노드 다운시 대응
-   route controller: 클라우드 환경의 네트워크 경로
-   service controller: 클라우드 로드밸런서 관리
-   volume controller: 클라우드 볼륨 관리

### 워커 노드(worker node)
container를 실행하고 동작중인 Pods 유지시키고 kubernetes runtime 환경 구성 (minion이라고도 함)

1) kubelet  
각 노드에서 실행되는 agent로 마스터로부터 제공받은 구성 정보, 노드가 수행해야할 작업 동작

2) kube-proxy  
네트워크 규칙 유지하고 연결에 대한 포워딩 수행하므로써 서비스 추상화 가능하게 함
컨테이너에 연결될 네트워크 구성 관리

3) container runtime  
실제 컨테이너 동작을 책임지는 구성요소

-   Docker
-   containerd
-   CRI-O
-   rktlet
-   Kubernetes CRI(Container Runtime Interface)

### 추가 요소(add-on)

| 구성요소 | 설명 |
| :-: | :-- |
| 클러스터 DNS | 쿠버네티스 클러스터 내의 여러 오브젝트에 대하여 주소기반으로 오브젝트를 접근할 수 있는 DNS 제공 |
| DashBoard | 모니터링 용 웹기반 인터페이스 |
| 컨테이너 resource 모니터링 | 컨테이너 리소스 사용량의 시계열 매트릭스 기록 |
| 클러스터 logging | 컨테이너 로그를 중앙로그저장소에 저장하고 관리, 로그를 검색/열람하는 인터페이스 제공 |

### 쿠버네티스 API

1) API 버전 규칙  
alpha 버전 API : 개발 초기 버전  
beta 버전 API : 일정 수준 이상 테스트가 된 버전  
stable 버전 : vX 형태의 안정화된 버전

2) API 그룹  
core group: apiVersion:\[version\]  
core 이외 group: apiVersion: \[group\]/\[version\]


<br><br>
---


### Managing Object

1) Imperative commands(명령형 커맨드)  
재사용성이 떨어져 개발 환경에서 테스트 시, 일회성 작업일 경우 사용

```
] kubectl run nginx --image nginx
```

2) Imperative object configuration(명령형 오브젝트 구성)  
yaml, json 포맷으로 파일을 작성하여 kubectl은 작성된 파일을 참고하여 실행

```
] kubectl create -f nginx.yaml
```

3) Declarative object configuration(선언형 오브젝트 구성)  
특정 디렉토리에 오브젝트 파일을 배치하고 작성된 파일을 참고하여 오브젝트 관리

```
] kubectl diff -f configs/
] kubectl apply -f configs/
```

### Running App with Imperative commands 예시

```
] kubectl run mytest-app --image=<ACCOUNT>/myweb --port=8080 --generator=run/v1 # 레플리케이션 컨트롤러(pod) 생성
] kubectl get pods
] kubectl get replicationcontrollers
] kubectl expose replicationcontroller mytest-app --type=LoadBalancer --name myweb-svc # 서비스 생성
] kubectl get services
] curl http://192.168.56.11:31289
] kubectl scale replicationcontroller mytest-app --replicas=3
] kubectl get pods
] kubectl get replicationcontrollers
] curl http://192.168.56.11:31289 
] kubectl get all
] kubectl delete replicationcontrollers mytest-app
] kubectl delete service myweb-svc
```

<br><br>
---


## 5.6. Dockerfile Manifest (명세)

| 필드 | 설명 |
| :-: | :-- |
| apiVersion | object를 생성하기 위한 API 버전을 지정 |
| kind | object 종류 |
| metadata | name, label, namespace 등 기본정보 |
| spec | object 세부 상태 |


<br><br>
---

# 6. Application Workload

## 6.1. Pods
POD(파드)는 쿠버네티스 애플리케이션의 기본 실행 단위 (만들고 배포할 수 있는 가장 작은 단위)  
1 container = 1 application(1 pod)  

### Pods 특징  
• 도커는 쿠버네티스 파드에서 사용되는 가장 대표적인 컨테이너 런타임이지만, 파드는 다른 컨테이너 런타임 역시 지원 (rtk, containerd)  
• 파드당 컨테이너 비율은 대체로 1:1 이지만, 파드당 여러개의 컨테이너가 포함되는 경우도 있음  
• 파드내의 컨테이너는 오로지 하나의 노드 내에만 존재 (노드를 걸쳐서 파드가 존재 하지 않음)  
• 동일 파드 내의 컨테이너는 네트워킹 및 볼륨을 공유  
• K8S에 있는 POD들은 단순하고, 공유 가능한 네트워크 Address 주소 값을 가진다 (flat Natwork)  
• 각각의 POD은 **각각의 IP주소 값**을 가지고 있으며, 이 IP를 이용해서 통신 허용  
plane에 정의된 kube-proxy로 IP를 가질 수 있음
• NAT(Network Address Translation) gateway 같은 장비 없이- 마치 Local Area Network(LAN)처럼 통신이 가능  
• POD 내의 여러개 Container 는 서로 다른 포트를 통해 서비스 해야함  



### Pod 정의 (YAML 파일 생성)  

cat testapp-pod.yml  
```
apiVersion: v1
kind: Pod
metadata:
  name: testapp-pod
spec:
  containers:
  - image: <ACCOUNT>/myweb
    name: testapp-container
    ports:
    - containerPort: 8080
      protocol: TCP
```
_하이픈(\-) 기호는 리스트(List)를 의미_  


### Pod 생성 및 확인  

```
] kubectl create -f testapp-pod.yml 
] kubectl get pods 
] kubectl get pods testapp-pod -o yaml(-o json) 
] kubectl discribe pods testapp-pod 
] kubectl logs testapp-pod # 로그 확인 
] kubectl port-forward testapp-pod 8080:8080 # 포트포워딩 
] curl http://localhost:8080
```

---
## 6.2. Label(라벨) 

#### Label: 쿠버네티스 클러스터의 모든 오브젝트에 키/값 쌍으로 이루어진 값을 설정하여 리소스 식별, 속성 지정 역할  
• 라벨은 Pod뿐 아니라 쿠버네티스의 모든 리소스를 구성하는 간단하고 강력한 기능  
• 리소스에 부여 하는 임의의 Key/Valye 쌍  
• 라벨 셀렉터를 사용해 리소스를 선택할 때 사용  
• 일반적으로 리소스를 만들 때 부여 하지만, 만들어진 후에도 라벨을 추가 하거나 변경 가능  
• 한 네임스페이스 내 중복 불가  

#### Label Selector: Label을 식별하고 검색함  


#### Pod에 Label 생성

cat testapp-pod-label.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: testapp-pod-label
  labels:
    env: dev
    tier: frontend
spec:
  containers:
  - image: <ACCOUNT>/myweb
    name: testapp-container
    ports:
    - containerPort: 8080
      protocol: TCP
```

#### pod 라벨 추가, 삭제 및 필터링  
```
] kubectl label pod goapp-pod app="app1" tier="frontend"
pod/goapp-pod labeled

] kubectl get po --show-labels
NAME         READY   STATUS    RESTARTS   AGE     LABELS
goapp-pod    1/1     Running   0          5m19s   app=app1,domain=pay,env=prod,tier=frontend
goapp-pod2   1/1     Running   0          2m54s   domain=pay,env=dev

] kubectl get po -l 'env in (prod,stage), tier in (frontend)'
NAME        READY   STATUS    RESTARTS   AGE
goapp-pod   1/1     Running   0          7m56s

] kubectl label pod goapp-pod app- tier-                     
pod/goapp-pod labeled
```

#### node 라벨 추가 및 필터링 
```
] kubectl label node gke-myk8s-cluster-default-pool-68423958-ss8l gpu=true
node/gke-myk8s-cluster-default-pool-68423958-ss8l labeled

] kubectl get no -l gpu=true
NAME                                           STATUS   ROLES    AGE     VERSION
gke-myk8s-cluster-default-pool-68423958-ss8l   Ready    <none>   6d23h   v1.21.5-gke.1302
```

![]({{site.baseurl}}/images/post/docker_6_1_2-label.jpg)

### label 활용  

#### 기능과 배포 구분  
• app(application)은 Pod가 속한 application 도메인, 기능 혹은 마이크로 서비스 지정(ui, account service, product catalog, shopping cart, order service...)  
• rel(release)은 Pod에서 실행 중인 애플리케이션의 배포 버전(canary, beta, stable...)    
• app과 rel을 추가 하면서 Pod 들을 2가지 차원으로 구성  

```
apiVersion: v1
kind: Pod
metadata:
    name: shopapp-pod
    labels:
        app: shopping cart
        env: prod
        creation_method: manual
spec:
    containers:
    - name: shopapp-container
        image: busybox
        command: ['sh', '-c', 'echo shopping cart app container && sleep 3600']
```

#### 스케줄링  
• k8s에서는 어떤 노드로 Pod가 스케줄 될지 알 수 없음  
• 라벨을 이용해서 특정 노드로 스케줄링 가능  
• GPU 유무, 메모리(High-Memmory), SSD 유무 를 노드의 라벨에 적용  
• ex) SSD 가 있는 노드에 Pod 스케줄링 등  

```
] kubectl label node cart.shoping.com ssd=true # 한 노드에 라벨 추가
```
```
apiVersion: v1
kind: Pod
metadata:
    name: shopapp-pod
    labels:
        app: shopping cart
        env: prod
        creation_method: manual
spec:
    nodeSelector: # 노드셀렉터에서 ssd 있는 것으로 명시
        ssd: "true"
    containers:
    - name: shopapp-container
        image: busybox
    
```

---
## 6.3. Anotation(어노테이션)

• Key/Value 쌍으로 라벨과 유사 하지만 셀렉터로 선택 불가능  
• 라벨보다 훨씬더 많은 정보를 담을 수 있음  
• 주로 주석을 사용해서 쿠버네티스 객체에 상세한 설명을 추가하여 다른 사람이 알아보기 쉽게 하는 용도  
• ex) 객체를 만든 사람이름을 기록하면 공동 작업 시에 훨씬 쉽게 작업  

```
] kubectl annotate pod goapp-pod  make-by="nueees" team="se"
pod/goapp-pod annotated

] kubectl describe po goapp-pod
Name:         goapp-pod
Namespace:    default
Priority:     0
Node:         gke-myk8s-cluster-default-pool-68423958-w4rq/10.178.0.2
Start Time:   Sat, 18 Dec 2021 10:56:00 +0900
Labels:       domain=pay
              env=prod
Annotations:  make-by: nueees
              team: se

...
```

---
## 6.4. Name Space(네임스페이스)

#### Name Space: 쿠버네티스 클러스터 내 오브젝트와 리소스를 용도와 목적에 따라 논리적으로 완전히 분리된 환경  
• k8s의 객체 이름의 범위를 제공하는 것  
• 동일한 리소스 이름을 여러 네임스페이스에서 여러 번 사용 가능(개발 / 검증/ 운영 에서 동일한 리소스 명 사용)  
• 멀티-테넌트 환경으로 분리하는 효과  

### Name Space 종류
default: 기본 네임스페이스  
kube-node-lease: 쿠버네티스 노드(마스터/노드)의 가용성 체크를 위한 네임스페이스  
kube-public: 모든 사용자 접근가능  
kube-system: 클러스터의 리소스가 배치되는 네임스페이스  


#### Name Space 생성 및 조회

```
] kubectl get ns
] kubectl get pod –namespace(혹은 -n) kube-system
```

first_ns.yaml  
```
apiVersion: v1
kind: Namespace
metadata:
  name: first-namespace
```

```
] kubectl create -f .\namespace\pod-node.yaml -n first-namespace
pod/nodeapp-pod1 created

] kubectl get po -n first-namespace
NAME           READY   STATUS    RESTARTS   AGE
nodeapp-pod1   1/1     Running   0          12s
PS C:\u01\kubetpl> apiVersion: v1

```


---
## 6.5. Liveness Probe(라이브니스 프로브)  
파드 상태가 정상적인지 주기적으로 모니터링 서비스(컨테이너의 Health Check)  
Health Check가 실패할 경우 컨테이너를 다시 시작  

### Probe 종류
- HTTP GET Prove: 지정된 IP/PORT에 HTTP GET 요청, HTTP 응답코드가 2XX/3XX인지 확인  
- TCP Socket Prove: 지정된 IP/PORT에 TCP port 연결 시도  
- Exec Prove: 컨테이너 내부의 바이너리(명령)를 실행하고 종료 코드 확인  

### Probe 옵션
- initialDelaySeconds: pod 가 시작되고 최초의 liveness probe가 수행되기 까지 대기시간  
- periodSeconds: probe가 수행 되는 간격  
- timeoutSeconds: 지정된 시간 안에 응답을 반환 해야 성공으로 간주  
- successThreshold: probe가 실패한 이후에 지정된 횟수 만큼 성공 할 경우 성공으로 간주  
- failureThreshold: probe가 지정한 개수 만큼 연속으로 실패하는 재가동 함  
- 
### httpGet 옵션
- host: 접속할 호스트명, httpHeaders에 설정 해도 무방 (기본 pod ip)  
- scheme: 접속 방법 선택 (HTTP 또는 HTTPS)  
- path: HTTP 서버 접속 경로  
- httpHeaders: request 의 커스텀 헤더  
- port: 접속할 컨테이너의 포트 넘버 또는 이름 (1~ 65536 사이값)  

#### livenessProbe 생성 및 조회  

cat testapp-pod-liveness.yml  

```
apiVersion: v1
kind: Pod
metadata:
  name: testapp-pod-liveness
spec:
  containers:
  - image: <ACCOUNT>/myweb
    name: testapp-container
    ports:
    - containerPort: 8080
      protocol: TCP
    livenessProbe: # 설정 값 명시
        httpGet:
            path: /
            port: 8080
```
    

<br><br>
---


## 6.6. Replication Controller  
노드 장애 발생 시 사용자가 의도한 상태로 유지 해 주는 기능  

- Replication Controller는 POD가 사라지게 되면 대체 할 POD 생성  
- Pod 수정 시점이 아니라 새로운 Pod 가 생성 되는 시점에 적용 됨  
- Pod Template 은 언제든지 수정 가능  
- Label selector 변경되면 POD가 Replication Controller 범위 벗어남  

### Replication Controller 구성  
- replicas: 실행해야 하는 POD의 원하는 수를 지정하는 복제본수
- selector: Replication Controller 범위에 있는 Pod를 결정하는 라벨 셀렉터를 사용함
- template: 새로운 POD 복제본을 만들때 사용해야 하는 POD Template


```
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-rc
spec:
  replicas: 3  # 복제본수 명시
  selector:  # RC 범위 지정할 대상 명시
    app: nginx
  template: # 복제본 정보 및 spec 명시
    metadata:
      name: nginx-pod
      labels:
        tier: forntend
        app: nginx
        env: prod
        priority: high
    spec:
      containers:
      - name: nginx-container
        image: nueees/nginx
        ports:
        - containerPort: 8080
```

```
] kubectl get rc
NAME       DESIRED   CURRENT   READY   AGE
nginx-rc   3         3         3       2m27s

] kubectl get rc -o wide
NAME       DESIRED   CURRENT   READY   AGE     CONTAINERS        IMAGES         SELECTOR
nginx-rc   3         3         3       2m48s   nginx-container   nueees/nginx   app=nginx

] kubectl get po
NAME             READY   STATUS    RESTARTS   AGE
nginx-rc-klrmt   1/1     Running   0          32s
nginx-rc-km8vn   1/1     Running   0          32s
nginx-rc-rqg4q   1/1     Running   0          32s
```

rqg4q 삭제 후 gsk7x 재생성 확인  

```
] kubectl delete pod nginx-rc-rqg4q
pod "nginx-rc-rqg4q" deleted

] kubectl get pod
NAME             READY   STATUS    RESTARTS   AGE
nginx-rc-gsk7x   1/1     Running   0          14s
nginx-rc-klrmt   1/1     Running   0          3m58s
nginx-rc-km8vn   1/1     Running   0          3m58s
```

app label 변경  
nginx -> nginx-new
새로운 dmnv2가 생기면서 app=nginx은 3개로 유지됨  
```
] kubectl get pod --show-labels
NAME             READY   STATUS    RESTARTS   AGE     LABELS
nginx-rc-gsk7x   1/1     Running   0          4m49s   app=nginx,env=prod,priority=high,tier=forntend
nginx-rc-klrmt   1/1     Running   0          8m33s   app=nginx,env=prod,priority=high,tier=forntend
nginx-rc-km8vn   1/1     Running   0          8m33s   app=nginx,env=prod,priority=high,tier=forntend

] kubectl label pod nginx-rc-gsk7x app=nginx-new
pod/nginx-rc-gsk7x labeled

NAME             READY   STATUS              RESTARTS   AGE     LABELS
nginx-rc-dmnv2   0/1     ContainerCreating   0          3s      app=nginx,env=prod,priority=high,tier=forntend
nginx-rc-gsk7x   1/1     Running             0          6m48s   app=nginx-new,env=prod,priority=high,tier=forntend
nginx-rc-klrmt   1/1     Running             0          10m     app=nginx,env=prod,priority=high,tier=forntend
nginx-rc-km8vn   1/1     Running             0          10m     app=nginx,env=prod,priority=high,tier=forntend
```

scale up (3->5)

```
] kubectl scale rc nginx-rc --replicas=5
replicationcontroller/nginx-rc scaled

NAME             READY   STATUS    RESTARTS   AGE
nginx-rc-7889w   1/1     Running   0          14s
nginx-rc-d4rlh   1/1     Running   0          14s
nginx-rc-dmnv2   1/1     Running   0          4m52s
nginx-rc-klrmt   1/1     Running   0          15m
nginx-rc-km8vn   1/1     Running   0          15m
```


---
## 6.7. ReplicaSet (레플리카셋)
사용자가 요구하는 복제본 개수만큼 Pod를 복제하고 관리하는 기능  
기존 Replication Controller 서로 다른 라벨을 가진 pod 관리 안되므로 RC 대체를 위해 사용  

- 일반적으로 ReplicaSet을 직접 생성 하기 보다 **Deployment** 를 이용해서 생성함  
- Replication Controller 와 동일하게 동작 하지만 다양한 표현식(match~)을 통해 POD를 선택 가능  
- 관리해야 하는 pod을 식별하기 위한 selector, 유지해야 하는 pod의 개수, pod template 포함  
1) matchExpressions: Pod에 설정된 레이블의 키 조재 여부 조건 선택 가능   
2) matchLabels: Pod의 다중 레이블 조건 지원  
   

cat testapp-rs-exp.yml

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
    matchExpressions:
      - {key: env, operator: In, values: [prod]}
      - {key: priority, operator: NotIn, values: [low]}
  template:
    metadata:
      labels:
        tier: frontend
        env: prod
        priority: high
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
``` 


```
] kubectl get po,rs -o wide
NAME                 READY   STATUS              RESTARTS   AGE   IP       NODE                                           NOMINATED NODE   READINESS GATES
pod/frontend-6ghjd   0/1     ContainerCreating   0          3s    <none>   gke-myk8s-cluster-default-pool-68423958-x85v   <none>           <none>
pod/frontend-cl5b2   0/1     ContainerCreating   0          3s    <none>   gke-myk8s-cluster-default-pool-68423958-w4rq   <none>           <none>
pod/frontend-vx6jg   0/1     ContainerCreating   0          3s    <none>   gke-myk8s-cluster-default-pool-68423958-ss8l   <none>           <none>

NAME                       DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                 SELECTOR
replicaset.apps/frontend   3         3         0       3s    php-redis    gcr.io/google_samples/gb-frontend:v3   env in (prod),priority notin (low),tier=frontend
```

```
# POD 까지 삭제
] kubectl delete rs frontend
# POD 는 남기고 삭제
] kubectl delete rs frontend --cascade=orphan
replicaset.apps "frontend" deleted

NAME                 READY   STATUS    RESTARTS   AGE     IP           NODE                                           NOMINATED NODE   READINESS GATES
pod/frontend-6ghjd   1/1     Running   0          7m35s   10.48.2.14   gke-myk8s-cluster-default-pool-68423958-x85v   <none>           <none>
pod/frontend-cl5b2   1/1     Running   0          7m35s   10.48.0.24   gke-myk8s-cluster-default-pool-68423958-w4rq   <none>           <none>
pod/frontend-vx6jg   1/1     Running   0          7m35s   10.48.1.13   gke-myk8s-cluster-default-pool-68423958-ss8l   <none>           <none>
```



---
## 6.8. DaemonSet (데몬셋)
쿠버네티스 클러스터는 부하 분산, 이중화를 통한 장애 대응 목적 등을 위하여 최소 하나 이상의 노드로 구성되므로 다수의 노드를 사용할 경우 필요에 따라 **각 노드별로 특정 목적을 수행하는 파드를 한 개씩** 배치하여야 하는 경우 사용  


- RC,RS 는 특정 노드에 특정 개수의 Pod 를 실행 하지만, 데몬셋은 라벨로 지정된 각각의 노드에 하나씩 정확하게 실행  
- 시스템 수준의 작업을 수행하는 인프라 관련 Pod나 모든 노드에서 로그를 수집 및 리소스 모니터링을 담당하는 Pod의 용도로 수행  
- 당연히 RC,RS 와는 다르게 노드가 다운 되더라도 다른 노드에 복제본 Pod를 생성 하지 않음  


cat testapp-ds.yml

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nodeapp-on-ssd
spec:
  selector:
    matchLabels:
      app: nodeapp-pod
  template:
    metadata:
      labels:
        app: nodeapp-pod
    spec:
      nodeSelector:
        disk: ssd
      containers:
      - name: nodeapp-container
        image: nueees/nodeapp

```

nodeSelector에 Pod가 배포될 노드 선택  

```
] kubectl create -f testapp-ds.yml

] kubectl get po # 안 보임
No resources found in default namespace.

] kubectl get ds
NAME             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE 
nodeapp-on-ssd   0         0         0       0            0           disk=ssd        6s 

] kubectl get no # 노드 확인
NAME                                           STATUS   ROLES    AGE    VERSION
gke-myk8s-cluster-default-pool-68423958-ss8l   Ready    <none>   7d4h   v1.21.5-gke.1302  
gke-myk8s-cluster-default-pool-68423958-w4rq   Ready    <none>   7d4h   v1.21.5-gke.1302  
gke-myk8s-cluster-default-pool-68423958-x85v   Ready    <none>   7d4h   v1.21.5-gke.1302

```

68423958-ss8l 노드에 ds에 명시된 disk ssd인 라벨 생성  

```
] kubectl label node gke-myk8s-cluster-default-pool-68423958-ss8l  disk=ssd
node/gke-myk8s-cluster-default-pool-68423958-ss8l labeled

] kubectl get po
NAME                   READY   STATUS              RESTARTS   AGE
nodeapp-on-ssd-54pnq   0/1     ContainerCreating   0          13s

```

<br><br>
---
## 6.9. Deployment  
Pod 와 ReplicaSet 에 대한 선언적 업데이트 제공하며, 배포에 대한 세분화된 기능 제공  
Deployment 에 Desired state를 기술하면, Deployment Controller는 현재 상태에서 Desired state로 비율을 조정  
 
### Deployment 기능  
ReplicaSet에 대한 Update 담당  
1) Liveness Probe: 응답 체크  
2) Readlness Probe: 서비스 가능 상태 체크  

- Deployment는 Pod 와 ReplicaSet 에 대한 선언적 업데이트 제공하며, 배포에 대한 세분화된 기능 제공
- Deployment 에 의도하는 상태를 기술하면, Deployment Controller는 현재 상태에서 의도하는 상태로 비율을 조정함
- Pod 에 대한 롤링 업데이트 및 롤백 기능 제공

### Deployment use case  
신규 ReplicaSet을 생성하여 Pod를 새로운 버전으로 점진적 교체 수행 (Rolling Update)  
spec.strategy.type==RollingUpdate (default)이면 파드를 롤링 업데이트 방식  
spec.strategy.type==Recreate 이면 새 파드가 생성되기 전에 kill  

Application configuration 분리 (Decoupling): Config Map  

셀렉터는 다른 컨트롤러(다른 디플로이먼트와 스테이트풀 셋 포함)와 겹치지 않아야 함  
만약 다중 컨트롤러가 겹치는 셀렉터를 가지는 경우 해당 컨트롤러의 충돌  
Deployment 로 배포 후에는 ReplicaSet 을 직접 관리 하지 않고 deployment 통해서 제어  

### Deployment 생성 및 조회  

cat testapp-dp.yml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
          - containerPort: 80

```

```
] kubectl get pod,rs,deploy
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-5d59d67564-5fgmh   1/1     Running   0          69s
pod/nginx-deployment-5d59d67564-b7cz7   1/1     Running   0          69s
pod/nginx-deployment-5d59d67564-lwszv   1/1     Running   0          69s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-5d59d67564   3         3         3       69s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           69s
```

scale out  

```
] kubectl edit deployment.v1.apps/nginx-deployment
] kubectl scale deploy nginx-deployment --replicas=5
deployment.apps/nginx-deployment scaled
```

image update & rollback   
```
] kubectl --record deployment.apps/nginx-deployment set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1 # 또는 ( kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1 --record )
deployment.apps/nginx-deployment image updated
] kubectl rollout undo deployment.v1.apps/nginx-deployment --to-revision=2
deployment.apps/nginx-deployment rolled back
```

revision 이력확인  
```
] kubectl rollout history deployment/nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
3         kubectl.exe set image deployment/nginx-deployment nginx=nginx:1.9.2 --record=true
4         kubectl.exe set image deployment.v1.apps/nginx-deployment nginx=nginx:1.7.9 --record=true
5         kubectl.exe deployment.apps/nginx-deployment set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1 --record=true
```



<br><br>
---
## 6.10. StatefulSet
Pod이 스케줄 될 때 **지속적으로 유지**되는 식별자를 가질 수 있도록 관리하는 object  

- 분산 기반의 애플리케이션의 경우 호스트명, IP 등이 바뀌면 안됨  
- 분산형 애플리케이션 가동시 동시 다운 / 동시 가동 하면 안되는 경우가 존재

### Statefullset 특징
- 생성된 Pod 마다 별도의 디스크 공간을 가져야 하는 경우 사용    
- ReplicaSet의 경우 생성 및 삭제가 동시에 진행되는 반면, StatefullSet 은 순차적 으로 진행 (시간조절 가능)  
- Pod 삭제 시 ID 값 내림차순으로 삭제
Pod A-0, Pod A-1, Pod A-2 존재 시  
Pod A-2, Pod A-1, Pod A-0 순으로 삭제 됨  

- 삭제된 Pod가 다른 노드에 스케줄 되어 생성 되더라도 동일한 ID로 생성되기 때문에 이전에 생성된 PVC가 다시 부착 됨  


### use case:  
고유한 네트워크 식별자, 지속성을 갖는 스토리지(persistent volumes), 순차적 배포와 스케일링, 순차적인 자동  



<br><br>
---

#### 현재 네임스페이스 모든 리소스 삭제
```
] kubectl delete all --all
```




<br><br>
---

# 7. Network - Service

## 7.1. Service 생성
K8s 특성상 노드 또는 Pod에 장애가 발생할 경우 Pod 가 다른 노드 및 다른 IP 로 실행 -> 정적 라우팅으로 해결 안됨  

### Service: 쿠버네티스 시스템에서 같은 애플리케이션을 실행하도록 구성된 컨트롤러에 의해 생성된 Pod 그룹에 단일 네트워크 진입점 제공

- 서비스에 부여된 IP는 해당 서비스가 종료될 때까지 유지하고 클라이언트는 이 서비스에 부여된 고정 IP 및 PORT를 통해 Pod에 접근 가능  
- 클러스터 내 Pod들에게 접근하기 위한 방법으로 사용  
- 여러 Pod를 묶어 Healthy한 Pod로 Traffic 라우팅하는 로드 밸런싱 기능 제공  
- 클러스터의 Service CIDR 중에서 지정된 IP로 생성 가능  
- 서비스 이름은 클러스터내 고유한 DNS로 동작  

cat testapp-svc.yml

```
apiVersion: v1
kind: Service
metadata:
    name: testapp-svc
spec:
    ports: 
        -port: 80 # 서비스 포트
        targetPort: 8080 # pod의 포트
    selector:
        app: testapp-rs # pod 레이블 셀렉터
```


```
] kubectl create -f testapp-svc.yml
] kubectl get services # 진입점인 서비스
] kubectl get endpoints testapp-svc # 엔드포인트 -> 레플리카셋 컨트롤러의 파드
```

엔드포인트: 최종 목적지인 파드의 주소 및 포트 정보

```
] kubectl run nettool -it --image=\<ACCOUNT\>/network-multitool # 서비스 접근 테스트
```

### Session Affinity: 클라이언트가 특정 파드(웹서비스) 요청 시 이전에 처리된 파드로 동일하게 전달하여 처리

cat testapp-svc-ses-aff.yml

```
apiVersion: v1
kind: Service
metadata:
    name: testapp-svc-ses-aff
spec:
    sessionAffinity: ClientIP
    ports: 
        -port: 80 # 서비스 포트
        targetPort: 8080 # pod의 포트
    selector:
        app: testapp-rs # pod 레이블 셀렉터
```

```
] kubectl create -f testapp-svc-ses-aff.yml
```

### Configuring Service Multi-Port

Pod는 하나 이상의 컨테이너로 구성되어 있어서, 각 컨테이너는 서로 다른 포트 사용 가능

cat testapp-svc-multiport.yml

```
apiVersion: v1
kind: Service
metadata:
    name: testapp-svc-mp
spec:
    ports: 
    - name: testapp-http
      port: 80 # 서비스 포트
      targetPort: 8080 # pod의 포트
    - name: testapp-https
      port: 443 # 서비스 포트
      targetPort: 8443 # pod의 포트
    selector:
        app: testapp-rs # pod 레이블 셀렉터
```
```
] kubectl create -f testapp-svc-mp.yml
```

### Configuring Service by named-port

레플리카셋 컨트롤러의 Pod 템플릿에서 생성될 컨테이너의 포트에 이름을 부여하여 포트 이름 구성 가능

cat testapp-rs-named-port.yml


```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: testapp-rs-nport
spec:
  replicas: 3
  selector:
    matchLabels:
      app: testapp-rs-nport
  template:
    metadata:
      labels:
        app: testapp-rs-nport
    spec:
      containers:
      - name: testapp
        image: <ACCOUNT>/myweb
        ports:
        - name: testapp-http
          containerPort: 8080
``` 

  
cat testapp-svc-named-port.yml

```
apiVersion: apps/v1
kind: Service
metadata:
  name: testapp-svc-nport
spec:
  ports:
  - name: testapp-http
    port: 80
    targetPort: testapp-http
  selector:
    app: testapp-rs-nport
``` 

```
] kubectl create -f testapp-rs-named-port.yml -f testapp-svc-named-port.yml
```

<br><br>
---
## 7.2. Service 탐색

kubectl get 명령어로 IP 주소를 수동적으로 확인할 수 있지만 Object끼리 통신을 위한 방식 필요

1.  환경변수 방식
2.  쿠버네티스 클러스터 내 DNS 사용 방식

### DNS를 이용한 Service 탐색

kube-system 네임스페이스에서 쿠버네티스에 등록된 구성요소 확인 가능  
그중 _k8s-app=kube-dns_ 레이블 옵션을 통해 DNS 관련 파드 확인

1) DNS 관련 리소스 확인

```
] kubectl get all -n kube-system -l k8s-app=kube-dns
```

2) 파드 내부 DNS 설정 확인  
이름 기반의 주소로 네트워크에 접근하기 위해서 DNS 설정 필요

```
] kubectl exec testapp-rs-m65m4 -- cat /etc/resolv.conf # 위의 coredns 서비스 IP와 다름
```

각 파드의 DNS로 등록되어 있는 위 IP(169.254.0.0/16 형식)는 IPv4 주소형식에서 'Link Local Address'이며, 유효한 IP 주소가 아님  
3) NoceLocal DNSCache  
해당 주소(169.254.25.10)는 coreDNS로 직접적으로 요청이 전달되지 않게 하는 중간 단계 캐시 DNS  
DaemonSet 형태로 쿠버네티스의 각 노드마다 DNS 캐시 기능을 하는 Pod 배치하여 필요한 경우에만 CoreDNS 호출하는 구조

### Pod <-> NodeLocal DNSCache <-> iptables <-> coreDNS

```
] kubectl get daemonsets.apps -l k8s-app=kube-dns -n kube-system # 각 노드별 데몬셋 컨트롤러 확인
] kubectl get pods -l k8s-app=kube-dns -n kube-system | grep -A 2 Args # node local dns 확인

] kubectl run nettool -it --image=\<ACCOUNT\>/network-multitool --generator=run-pod/v1 --rm=true bash # 서비스 접근 테스트
```

주소구성: <리소스(서비스) 이름>.<네임 스페이스>.<리소스 종류>.<클러스터 도메인>


<br><br>
---
## 7.3. Service 종류

위의 내용은 내부 접근이고, 외부 접근 제공하는 서비스 구성 필요

### ClusterIP: 클러스터 내부용 진입점 제공  

### NodePort: 쿠버네티스 각 노드(호스트)의 포트를 외부 접근용으로 할당  
노드의 특정 포트를 할당 및 개방 하여 서비스(ClusterIP) 와 연동함  
노드의 포트 할당 범위가 (30000 ~ 32767 : --service-node-port-range) 로 제한됨 (장비나 VM의 IP 혹은 포트 변경 발생시 대응 어려움)  
특정 노드로 접속하더라도 ClusterIP 로 연동 되기 때문에 로드밸런싱 됨  

### LoadBalancer: NodePort의 확장판으로, 외부 로드밸런서로 접근하면 서비스를 통해 파드로 Redirection  

### ExternalName: 외부에서 접근하기 위한 서비스 유형이 아닌, CNAME 매핑을 통해 특정 FQDN과 통신을 위한 기능  

### Ingress:  


1. 외부 접근용 레플리카셋 생성 및 확인
```
] kubectl create -f testapp-rs.yml 
] kubectl get replicasets.apps 
] kubectl get pods
```
2.  NodePort 서비스 생성  
cat testapp-svc-ext-nodeport.yml   
```
apiVersion: v1
kind: Service
metadata:
    name: testapp-svc
spec:
    type: NodePort
    ports: 
      - port: 80 # 서비스 포트
        targetPort: 8080 # pod의 포트
        nodePort: 31111 # 
    selector:
        app: testapp-rs # pod 레이블 셀렉터
```

```
] kubectl create -f testapp-svc-ext-nodeport.yml # 해당 노드에서 사용할 포트 31111로 지정 
] kubectl get endpoints testapp-svc-ext-np # 서비스의 엔드포인트 확인 (Pod의 8080 포트로 Redirection 됨) 
] kubectl get nodes -o wide # 각 노드의 IP 확인
```

3.  LoadBalancer 서비스 생성  
cat testapp-svc-ext-loadbalancer.yml   
```
apiVersion: v1
kind: Service
metadata:
    name: testapp-svc-ext-lb
spec:
    type: LoadBalancer
    ports: 
      - port: 80 # 서비스 포트
        targetPort: 8080 # pod의 포트
    selector:
        app: testapp-rs # pod 레이블 셀렉터
```

```
] kubectl create -f testapp-svc-ext-loadbalancer.yml # 해당 노드에서 사용할 포트는 정의하지 않음 
] kubectl get services # LoadBalancer 서비스 확인
```

4.  ExternalName 서비스 생성  
cat testapp-svc-ext-externalname.yml   

```
apiVersion: v1
kind: Service
metadata:
    name: testapp-svc-extname-gl
spec:
    type: ExternalName
    externalName: www.google.com
```

```
] kubectl run nettool -it --image=\<ACCOUNT\>/network-multitool --generator=run-pod/v1 --rm=true bash # 서비스 접근 테스트 
] nllookup testapp-svc-extname-gl
```
```
] kubectl create -f testapp-svc-ext-externalname.yml # FQDN은 google이며 이에 대한 CNAME은 testapp-svc-extname-gl 
] kubectl get services # ExternalName 서비스 확인
```



<br><br>
---

# 8. Storage - Volumes

- Pod에서 실제 데이터가 있는 디렉토리를 보존 하기 위해 사용  
- Pod의 일부로 정의 되며, Pod 와 라이프사이클을 같이함  
- 독립적인 쿠버네티스 오브젝트가 아니며 스스로 생성 하거나 삭제할 수 없다(kind 가 없다)  
- 마운트(mount)를 통해서만 사용 할 수 있다  
- 실제 호스트 에서는 /var/lib/kubelet/pods/PODUID/volumes/ 위치에서 볼륨 확인 가능  

## 8.1. 일반 Volume
- emptyDir: 데이터를 저장하는데 사용, 비어있는 단순한 디렉토리  

containers 내 volumeMounts과 volumes 내 emptyDir로 정의  
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fortune-deployment
  labels:
    app: fortune
spec:
  replicas: 3
  selector:
    matchLabels:
      app: fortune
  template:
    metadata:
      labels:
        app: fortune
    spec:
      containers:
      - image: nueees/fortune
        name: html-generator
        volumeMounts:
        - name: html
          mountPath: /var/htdocs
      - image: nginx:alpine
        name: web-server
        volumeMounts: # 설정된 볼륨 마운트
        - name: html
          mountPath: /usr/share/nginx/html
          readOnly: true
        ports:
          - containerPort: 80
            protocol: TCP
      volumes: # 볼륨 설정
      - name: html
        emptyDir: {}
```

- hostPath: 노드의 파일 시스템에서 Pod의 디렉토리로 마운트 하는 데 사용  
• 노드의 파일시스템에 있는 특정 파일 또는 디렉토리를 가지는 볼륨  
• Pod는 삭제 되도 Host볼륨은 삭제 되지 않음  
• Pod를 재 시작 해도 동일한 노드에 스케줄 되어야 이전 데이타 를 볼 수 있음  
• 일반적인 목적의 Pod의 경우에 사용 하는 것이 좋지 않음  

- gitRepo: Git 리포지토리를 Checkout 해서 초기화 하는 볼륨    
• Git 리포지토리를 복제해 만들어지는 emptyDir 볼륨  
• 웹애플리케이션의 정적 컨텐츠를 제공하는 볼륨으로 사용함  

containers 내 volumeMounts과 volumes 내 gitRepo로 정의  
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitvolume-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:alpine
        name: web-server
        volumeMounts: # 설정된 볼륨 마운트
        - name: html
          mountPath: /usr/share/nginx/html
          readOnly: true
        ports:
          - containerPort: 80
            protocol: TCP
      volumes: # 볼륨 설정
      - name: html
        gitRepo:
          repository: https://github.com/nueees/k8s-test.git
          revision: master
          directory: .
```

- nfs: Pod에 마운트된 네트워크 파일 시스템  


- gcePersistentDisk(구글), awsEElastic-BlockStore(아마존), azureDisk(애저)  

containers 내 volumeMounts과 volumes 내 gcePersistentDisk 등으로 정의  
```
apiVersion: v1
kind: Pod
metadata:
  name: mongodb
spec:
  volumes: # 볼륨 설정
  - name: mongodb-data
    gcePersistentDisk: # 구글 persistentDisk 
      pdName: mongodb
      fsType: ext4
  containers:
  - image: mongo
    name:  mongodb
    volumeMounts: # 설정된 볼륨 마운트
    -  name: mongodb-data
       mountPath: /data/db
    ports:
    - containerPort: 27017
      protocol: TCP
```

- 그 외 cinder,cephfs,iscsi,flocker,quobyte,glusterfs 등  



<br><br>
---

## 8.2. Volume 공유 관리
일반적인 애플리케이션 컨테이너가 로그를 기록하고, 또 다른 컨테이너는 로그를 읽어서 로그 서버로 전송

### PersistentVolume  

```
apiVersion: v1
kind: PersistentVolume
metadata:
   name: mongodb-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  - ReadOnlyMany
  persistentVolumeReclaimPolicy: Retain
  gcePersistentDisk:
   pdName: mongodb
   fsType: ext4
```
### PersistentVolumeClaim  


```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc
spec:
  resources:
    requests:
      storage: 1Gi
  accessModes:
  - ReadWriteOnce
  storageClassName: ""
```


### Pod으로 생성 시

```
apiVersion: v1
kind: Pod
metadata:
  name: mongodb
spec:
  containers:
  - image: mongo
    name: mongodb
    volumeMounts:
    - name: mongodb-data
      mountPath: /data/db
    ports:
    - containerPort: 27017
      protocol: TCP
  volumes:
  - name: mongodb-data
    persistentVolumeClaim:
      claimName: mongodb-pvc
```


<br><br>
---
## 8.3. 특수 Volume
- configMap: 설정파일 및 환경 변수 보관 하는 볼륨  
- secret: 기밀 유지가 필요한 설정 파일 이나 환경 변수 보관 하는 볼륨  

### ConfigMap and Secret
• ConfigMap 및 Secret 는 Key Value 쌍으로 데이터를 저장 하는 점에서는 동일  
• ConfigMap은 일반적인 USE CASES  
    • 환경 변수 및 파라매터 전달  
    • Container 내부에서 사용되는 명령어 및 매개 변수  
    • 어플리케이션이 사용하는 읽기 전용 파일  
    • ConfigMap을 읽어서 K8s API를 이용해 Pod 내부에서 수행되는 코드 작성 할때  
    • Secret은 패스워드 및 인증서 와 같은 중요정보 보관용으로 사용 (BASE 64 인코딩, 암호화 X)  
• Immutable ConfigMaps  
    • Application 오류로 인한 원하지 않는 업데이트로 부터 보호  
    • 대규모 클러스터에서 성능개선 : kube-apiserver의 부하를 획기적으로 줄임. ConfigMap 을 감시 하지 않기 때문에  



<br><br>
---

# 9. 추가로..
## 9.1. Resource Request 및 Limit
CPU 및 메모리 에 대해 리소스 설정 가능  
CPU는 하이퍼스레딩 된 가상코어 단위  


cat resource-rq-lmt.yml
```
apiVersion: v1
kind: Pod
metadata:
    name: frontend
spec:
    containers: 
    - name: db
      image: mysql
      env:
      - name: MYSQL_ROOT_PASSWORD
        value: "password"
      resource:
        request: # 리소스 요청
          memory: "64Mi"
          cpu: "250m"
        limit: # 제한값
          memory: "128Mi"
          cpu: "500m"
    - name: wp
      image: wordpress
      resource: # 리소스 요청
        request:
          memory: "64Mi"
          cpu: "250m"
        limit: # 제한값
          memory: "128Mi"
          cpu: "500m"
     
```
 
### Pod 스케줄링 시 리소스 계산  
스케줄러는 요청 사항을 합해서 미 할당된 리소스를 계산 함  
실제 사용량은 여유가 있어도 다른 Pod에서 요청된 할당량으로 인해, 여유 리소스가 부족할 경우 Pod 할당 되지 않을 수 있음  
노드 describe 해서 나오는 allocatable 부분만 확인함  

```
] kubectl describe nodesd
```
```
Capacity: 
  cpu:
  ...
  memory:
Allocatable: # 스케줄러는 allocatable 이부분만 확인함
  cpu:
  ...
  memory:
  pods:
```

<br><br>
---
## 9.2. HPA (Horizontal Pod Auto-Scaler)  

### HPA 작동 원리
- Horizontal 컨트롤러가 HorizontalPodAutoscaler(HPA) 리소스를 만들어 수평 스케일링을 수행 함  
- 스케일링 대상은 리플리케이션 컨트롤러 / 디플로이먼트 / 리플리카셋 / 스테이트풀셋  
- Kubelet 의 cAdvisor 를 통해 메트릭이 수집됨  
- HPA 컨트롤러는 horizontal-pod-autoscaler-use-rest-clients 옵션를 통해 메트립 수집  
```
–horizontal-pod-autoscaler-use-rest-clients=true / --api-server=$(api server aggregator)
```


### HPA 계산 방법  
- HPA 가 Metric 을 수집해서 계산한 결과는 복제본의 수  
- 단일 메트릭 항목만 고려 하도록 구성된 경우, 모든 Pod의 메트릭 값을 합산후 HPA 에 설정된 목표 값으로 나눈 후 큰 정수로 반올림  
- 여러 메트릭을 고려 하도록 구성된 경우 계산이 훨씬 복잡 해짐  


<br><br>
---
## 9.3. Account 관리  

### User Account  
주로 클러스터 관리에 사용  

### Service Account  
응용 프로그램의 API 접근을 위해 사용  
각 Pod 는 네임스페이스 있는 단일 서비스 어카운트와 연관돼 있음  

### Role and Role Binding  
- 특정 리소스에 접근 가능한 롤을 만들고 롤 바인딩을 통해 사용자나 그룹에 롤을 부여함
- 바인딩은 유저 계정 및 서비스 계정 모두에 바인딩 가능함
- 하나의 유저는 여러 개의 롤을 바인딩을 통해 부여 받을 수 있음


<br><br>
---
## 9.4. Access Control  

### Attribute-Based Access Control  
### Role-Based Access Control  
- 롤(Role)은 특정 api나 리소스에 대한 권한들을 명시해둔 규칙들의 집합  
- 롤에는 롤(Role)과 클러스터롤(ClusterRole) 2가지 종류가 있음  
- 클러스터롤은 특정 네임스페이스에 대한 권한이 아닌 클러스터 전체에 대한 권한을 관리(네임스페이스에 소속 되지 않음 리소스)  




<br><br>
---
## 9.5. Taint 및 Toleration
- Taint는 {key}={value}:{option} 형식으로 이루어짐
- taint: 노드마다 설정 가능 하며, 설정한 노드에는 Pod가 스케줄 되지 않음
- Toleration: taint를 무시 할 수 있음
- Taint에는 3가지 종류가 있음


| Taint | 설명 |
| :-: | :-- |
| NoSchedule | toleration이 없으면 pod이 스케줄되지 않음 기존 실행되던 pod에는 적용 안됨 |
| PreferNoSchedule | toleration이 없으면 pod을 스케줄링안하려고 하지만 필수는 아님, 클러스터내에 자원이 부족하거나 하면 taint가 걸려있는 노드에서도 pod이 스케줄링될 수 있음 |
| NoExecute | toleration이 없으면 pod이 스케줄되지 않으며 기존에 실행되던 pod도 toleration이 없으면 종료시킴|

<br><br>
---

