---
aliases:
- /prometheus/kubernetes/2022/12/20/prometheus
categories:
- prometheus
- kubernetes
date: '2022-12-20'
description: Exporter, PromQL, Operator
layout: post
title: Prometheus
toc: false

---

[실습으로-배우는-프로메테우스](https://github.com/SeongJuMoon/_Lecture_prom_learning.kit)

---

# 3. Prometheus Installation

## Helm [native prometheus](https://artifacthub.io/packages/helm/edu/prometheus)

```
$ helm repo add edu https://iac-source.github.io/helm-charts
$ helm repo update
$ helm install prometheus edu/prometheus \
  --set pushgateway.enabled=false \
  --set alertmanager.enabled=false \
  --set nodeExporter.tolerations[0].key="node-role.kubernetes.io/master" \
  --set nodeExporter.tolerations[0].effect="NoSchedule" \
  --set nodeExporter.tolerations[0].operator="Exists" \
  --set nodeExporter.tolerations[1].key="node-role.kubernetes.io/control-plane" \
  --set nodeExporter.tolerations[1].effect="NoSchedule" \
  --set nodeExporter.tolerations[1].operator="Exists" \
  --set server.service.type="LoadBalancer" \
  --set server.global.scrape_interval="1m" # default \
  --set server.global.evaluation_interval="1m" # default \
  --set server.extraFlags[0]="web.enable-lifecycle" \
  --set server.extraFlags[1]="storage.tsdb.no-lockfile" \
  --namespace=monitoring \
  --create-namespace
```

## 3.1. [Prometheus Configuration](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#configuration-file)

```
global:
  scrape_interval: 15s
  evaluation_interval: 30s
  # scrape_timeout is set to the global default (10s).

  external_labels:
    monitor: codelab
    foo: bar

rule_files:
  - "first.rules"
  - "reporting/*.rules"
  - "alerting/*.rules"

alerting:
  alertmanagers:
    - scheme: https
      static_configs:
        - targets:
            - "1.2.3.4:9093"
            - "1.2.3.5:9093"
            - "1.2.3.6:9093"

scrape_configs: # targets 집합으로 각 target에서 어떻게 수집할 건지 param 
  - job_name: prometheus

    honor_labels: true
    # scrape_interval is defined by the configured global (15s).
    # scrape_timeout is defined by the global default (10s).

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    file_sd_configs:
      - files:
          - foo/*.slow.json
          - foo/*.slow.yml
          - single/file.yml
        refresh_interval: 10m
      - files:
          - bar/*.yaml

    static_configs:
      - targets: ["localhost:9090", "localhost:9191"]
        labels:
          my: label
          your: label

    relabel_configs:
      - source_labels: [job, __meta_dns_name]
        regex: (.*)some-[regex]
        target_label: job
        replacement: foo-${1}
        # action defaults to 'replace'
      - source_labels: [abc]
        target_label: cde
      - replacement: static
        target_label: abc
      - regex:
        replacement: static
        target_label: abc
      - source_labels: [foo]
        target_label: abc
        action: keepequal
      - source_labels: [foo]
        target_label: abc
        action: dropequal

    authorization:
      credentials_file: valid_token_file

    tls_config:
      min_version: TLS10

  - job_name: service-x

    basic_auth:
      username: admin_name
      password: "multiline\nmysecret\ntest"

    scrape_interval: 50s
    scrape_timeout: 5s

    body_size_limit: 10MB
    sample_limit: 1000

    metrics_path: /my_path
    scheme: https

    dns_sd_configs:
      - refresh_interval: 15s
        names:
          - first.dns.address.domain.com
          - second.dns.address.domain.com
      - names:
          - first.dns.address.domain.com

    relabel_configs:
      - source_labels: [job]
        regex: (.*)some-[regex]
        action: drop
      - source_labels: [__address__]
        modulus: 8
        target_label: __tmp_hash
        action: hashmod
      - source_labels: [__tmp_hash]
        regex: 1
        action: keep
      - action: labelmap
        regex: 1
      - action: labeldrop
        regex: d
      - action: labelkeep
        regex: k

    metric_relabel_configs:
      - source_labels: [__name__]
        regex: expensive_metric.*
        action: drop
```


## 3.2. Prometheus Recording Rules
6.4. recording rules 추가에서 다룸  


# 4. Prometheus Exporter

## 4.1. Service discovery (수집 대상)

configuration tap의 job_name의 kubernetes_sd_configs(수집 대상)을 통해 target을 생성
```
global:
  ...
rule_files:
  ...
scrape_configs:
- job_name: prometheus
  ...
- job_name: kubernetes-apiservers
  ...
  kubernetes_sd_configs:
  - role: endpoints
    ...
- job_name: kubernetes-nodes
  ...
  kubernetes_sd_configs:
  - role: node
    ...
- job_name: kubernetes-nodes-cadvisor
  ...
  kubernetes_sd_configs:
  - role: node
    ...
- job_name: kubernetes-service-endpoints
  ...
  kubernetes_sd_configs:
  - role: endpoints
    ...
- job_name: kubernetes-service-endpoints-slow
  kubernetes_sd_configs:
  - role: endpoints
    ...
- job_name: prometheus-pushgateway
  ...
  kubernetes_sd_configs:
  - role: service
    ...
- job_name: kubernetes-services
  ...
  kubernetes_sd_configs:
  - role: service
    ...
- job_name: kubernetes-pods
  ...
  kubernetes_sd_configs:
  - role: pod
    ...
- job_name: kubernetes-pods-slow
  ...
  kubernetes_sd_configs:
  - role: pod
    ...
```


## 4.2. cAdvisor(container monitoring)
각 container 내의 Kubelet을 통해 메트릭 수집  
![image](https://user-images.githubusercontent.com/83441376/208759712-49d03556-90a2-45ef-8cd0-69806be689cb.png)


graph tap에서 container로 prefix주고 검색하는 모든 항목이 cAdvisor를 통해 수집된 데이터임. 
```
container_network_receive_bytes_total{pod="pod-name"}
```



## 4.3. node-exporter(node monitoring)
각 node(os 관련)의 메트릭 수집 
```
ls /sys/fs/cgroup # sys 정보
] blkio  cpuacct      cpuset   freezer  memory   net_cls,net_prio  perf_event  systemd
  cpu    cpu,cpuacct  devices  hugetlb  net_cls  net_prio          pids

cat /proc/version | cat /proc/cpuinfo | cat /proc/meminfo # proc 정보
] Linux version 3.10.0-1127.19.1.el7.x86_64
] processor       : 0
  model name      : 11th Gen Intel(R) Core(TM) i7-1165G7 @ 2.80GHz
  cpu cores       : 2
  ...
] MemTotal:        2046800 kB
  MemFree:           89764 kB
  MemAvailable:     903108 kB
  Buffers:            1036 kB
  Cached:           919464 kB
  ...
```

graph tap에서 node로 prefix주고 node name 넣으면 node os 관련 수집된 데이터임. 
```
PromQL] node_memory_Active_bytes{node="w3-k8s"}
PromQL] node_cpu_seconds_total{node="w3-k8s", mode="user"}
```

### 4.3.1. cgroup  

process에 할당하는 CPU, memory, network bandwidth와 같은 자원을 개별적으로 제한하기 위한 system kirnel의 기능  
container runtime은 systemd 혹은 cgroupfs으로 cgroup 자원을 할당  
```
docker info | grep Cgroup -F2 # 명령어로 cgroup 정보 확인가능
] Cgroup Driver: systemd
```

### 4.3.2. cgroup - cgroupfs  
/sys/fs/cgroup 경로에서 파일시스템 형태로 resource를 직접 매핑하는 구조로 관리  
```
tree /sys/fs/cgroup
] /sys/fs/cgroup
  |-- blkio
  |   |-- blkio.io_merged
  |   |-- blkio.io_merged_recursive
  |   |-- kubepods.slice
  |   |   |-- blkio.io_merged
  |   |   |-- blkio.io_merged_recursive
  ...
```
docker에서 사용(systemd로 변경 예정)  

### 4.3.2. cgroup - systemd  
slice > scope > service 단위의 계층 구조로 slice의 이름으로 된 파일 시스템 구조로 관리    
/sys/fs/cgroup/systemd 경로에서 계층 구조 확인 가능  
```
systemd-cgls
] ├─1 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
  ├─user.slice
  │ └─user-0.slice
  │   └─session-4.scope
  │     ├─13329 sshd: root@pts/0
  └─system.slice
    ├─systemd-logind.service
    │ └─32496 /usr/lib/systemd/systemd-logind
    ├─kubelet.service
    │ └─4333 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernete
    ├─docker.service
    │ └─3437 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
    ├─containerd.service
    │ ├─ 3428 /usr/bin/containerd
    │ ├─ 4013 /usr/bin/containerd-shim-runc-v2 -namespace k8s.io -id 9aa76cbf6daf29adb4140d548b086dc0ee96325ed455335af5
    │ ├─kubepods-besteffort-pod068e1c1b_4b04_4ee0_a9a5_91b79efc8447.slice:cri-containerd:ef8138f99e53bffaf9fcef4ac44839
    │ │ └─1555 /speaker --port=7472 --log-level=info
    │ ├─kubepods-burstable-podd5af555b67b9595de495fa4320f406a7.slice:cri-containerd:01360ee58caed091cf04b20aaf684a0fae8
    │ │ └─1356 kube-controller-manager --allocate-node-cidrs=true --authentication-kubeconfig=/etc/kubernetes/controlle
```
k8s에서 default 사용  


## 4.4. kube-state-metrics(cluster monitoring)
control plane에서 kube-api-server를 통해 kubenates object status의 메트릭 수집  
![image](https://user-images.githubusercontent.com/83441376/208758030-bcf29991-dc08-4284-8e15-d83fb50a2057.png)



graph tap에서 kube로 prefix주고 검색하면 kube-state-metrics로 수집된 데이터임.  
```
$ kubectl scale deploy deploy-app --replicas 6 # scale out deployment 후 replica된 pod 확인 가능
PromQL] kube_deployment_status_replicas_available{deployment="deploy-app"}
```


## 4.5. application 전용 exporter(application monitoring)
4.2. cAdvisor(container monitoring)과 비슷하나, 각 application에서 수집하고자 하는 추가된 adapter(or sidecar) container 전용 메트릭 e.g. HTTP request

graph tap에서 application(nginx)로 prefix주고 검색하면 해당 컨테이너로 수집된 데이터임.  
```
PromQL] nginx_up
PromQL] nginx_http_requests_total
```

deployment-nginx-exporter.yaml (nginx+exporter 컨테이너 두개)
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      annotations:
        # config default exec/logs connectivity for main nginx 
        kubectl.kubernetes.io/default-container: nginx
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.21
        name: nginx
        volumeMounts:
        - name: nginx-vol
          mountPath: /etc/nginx/templates
        env:
        - name: PORT
          value: '8080'
      - image: nginx/nginx-prometheus-exporter:0.10.0
        name: exporter
        ports:
        - containerPort: 9113
        args:
        - -nginx.scrape-uri=http://localhost:8080/stub_status # 8080 port의 nginx 데이터 긁어 옴
```


## 4.6. about embeded application(other application monitoring)
4.5. application 전용 exporter(application monitoring)와 비슷하고 이미 구현되어 있는 메트릭들 활성화 시켜주면 확인 됨 e.g. [metallb](https://github.com/metallb/metallb/blob/main/e2etest/bgptests/metrics.go)

graph tap에서 application(metallb)로 prefix주고 검색하면 해당 컨테이너로 수집된 데이터임.  
```
PromQL] metallb_allocator_addresses_in_use_total
```

해당 app spec을 보면 prometheus 수집 변수가 있음.  
```
kubectl edit -n metallb-system daemonsets.apps speaker
] spec:
    revisionHistoryLimit: 10
    selector:
      matchLabels:
        app: metallb
        component: speaker
    template:
      metadata:
        annotations:
          prometheus.io/port: "7472" # 이 port를 통해서
          prometheus.io/scrape: "true" # 수집할 것임
        creationTimestamp: null
        labels:
          app: metallb
          component: speaker
      spec:
        containers:
        - args:
          - --port=7472
          - --log-level=info
          env:
          - name: METALLB_NODE_NAME
            valueFrom:
              fieldRef:
                apiVersion: v1
```

configuration 탭에서도 확인 가능.  
```
- job_name: kubernetes-pods
  ...
  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
    separator: ;
    regex: "true" # 수집할 것
    replacement: $1
    action: keep
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_port]
    separator: ;
    regex: "true"
    replacement: $1
    action: drop
```

---

# 5. PromQL

## 5.1. Metric Type
### Gauge: 특정 시점 값 e.g. momory usage  
```
PromQL] node_memory_Active_bytes
```
### Counter: 누적된 값(total) e.g. event error  
```
PromQL] node_cpu_seconds_total
```
### Summary: 구간 내 빈도(duration_seconds) e.g. client request에 대한 response time  
```
PromQL] prometheus_target_interval_length_seconds
```
### Histogram: Summary를 위한 raw type으로 구간 별 빈도(bucket), PromQL상으로 표시 안됨  
```
PromQL] histogram_quantile(0.99, rate(prometheus_http_request_duration_seconds_bucket[5m]))
```


## 5.2. Label Matchers
중괄호 내 label로 filter  
```
PromQL] node_memory_Active_bytes{node!="w3-k8s"}
PromQL] node_memory_Active_bytes{node=~"w.+"} # 정규표현식 쓸때 w+가 아닌 w.+로 온점(.)써야함
PromQL] node_memory_Active_bytes{node="w3-k8s|w1-k8s"}
```

## 5.3. Binary Operators
PromQL 결과를 가공하는 방법 (Arithmetic binary operators, Comparison binary operators, Logical/set binary operators)  
```
PromQL] node_memory_Active_bytes/1024/1024 # MB 단위로 변경
PromQL] kube_pod_container_status_restarts_total > 3
PromQL] kube_pod_container_status_terminated > 0 or kube_pod_container_status_waiting > 0
```

## 5.4. [Aggregation Operators](https://prometheus.io/docs/prometheus/latest/querying/operators/#aggregation-operators)

```
PromQL] topk(3,node_cpu_seconds_total) # top 3위 뽑아서 보여줌
PromQL] bottomk(3,node_cpu_seconds_total > 0) # 0 보다 큰 bottom 3위 뽑아서 보여줌
PromQL] avg(node_cpu_seconds_total{mode="user"}) by (node)
PromQL] sum(kubelet_http_requests_total) without(path) # without도 by처럼 grouping 함
```

## 5.5. [TimeSeries Selector](https://prometheus.io/docs/prometheus/latest/querying/basics/#time-series-selectors)
시점 instant vector  
```
PromQL] node_memory_Active_bytes # default 시점 데이터
```
구간 range vector  
```
PromQL] node_memory_Active_bytes[2m] # 대괄호안에 시간정보를 넣으면 만약 1분에 1번씩 수집하는 config에서는 두개의 capture된 결과치가 나타남
```

## 5.6. [Modifier](https://prometheus.io/docs/prometheus/latest/querying/basics/#modifier)

```
PromQL] node_memory_Active_bytes offset 1m # 1분 전 데이터 값
PromQL] node_memory_Active_bytes @ 1609746000 # uninx_time으로 2021-01-04T07:40:00+00:00
```

## 5.7. Funtions

5.5 range vector값과 함께 쓰이는 경우가 많음  

변화율(rate): 구간의 시작과 끝  
순간변화율(irate): 구간 종료 바로 직전과 구간 종료 값 계산  

```
PromQL] rate(node_memory_Active_bytes[5m] @ 1609746000) # 2021-01-04T07:40:00+00:00 시점의 5분간 메모리 변화율
PromQL] predict_linear(node_memory_Active_bytes[5m],60*60*12)/1024/1024 # 5분간 변화율로 12시간 후 메모리 예측
```



---

# 6. Custom management

## 6.1. scrape interval 변경

### replace
현재 config map에서 scrap_interval 15s 변경 후 적용
```
kubectl create -f deployment.yaml
kubectl replace -f replace-scrape-interval-15s.yaml
] apiVersion: v1
  data:
    alerting_rules.yml: |
      {}
    alerts: |
      {}
    prometheus.yml: |
      global:
        evaluation_interval: 15s
        scrape_interval: 15s
        scrape_timeout: 10s
  kind: ConfigMap
  metadata:
    annotations:
      meta.helm.sh/release-name: prometheus
      meta.helm.sh/release-namespace: monitoring
    labels:
      app: prometheus
      app.kubernetes.io/managed-by: Helm
      chart: prometheus-15.8.5
      component: server
      heritage: Helm
      release: prometheus
    name: prometheus-server
    namespace: monitoring
```

### patch
```
kubectl patch -n monitoring configmap prometheus-server --patch-file patch-scrape-interval-15s.yaml 

```

## 6.2. target 추가
4.6. embeded된 exporter(application monitoring) 사용하도록 추가 설정을 통해 expose 시켜줘야 함. e.g.[kubemetrics](https://github.com/kubernetes/kubernetes/blob/master/pkg/proxy/metrics/metrics.go)  


kubeproxy 수동으로 anotations 넣기  
```
kubectl edit daemonset -n kube-system kube-proxy
] spec:
    revisionHistoryLimit: 10
    template:
      metadata:
        annotations:
          prometheus.io/port: "10249"      <<<
          prometheus.io/scrape: "true"     <<<
        creationTimestamp: null
      spec:
        containers:
        - command:
          ...
```

metrics config mapping 해주고 restart  
```
kubectl edit cm kube-proxy -n kube-system
] apiVersion: v1
data:
  config.conf: |-
    apiVersion: kubeproxy.config.k8s.io/v1alpha1
    ...
    kind: KubeProxyConfiguration
    metricsBindAddress: "" # "0.0.0.0:10249" 으로 변경
```

```
kubectl rollout restart daemonset kube-proxy  -n kube-system
```


graph tap에서 application(kubeproxy)로 prefix주고 검색하면 해당 컨테이너로 수집된 데이터임.  
```
PromQL] kubeproxy_sync_proxy_rules_iptables_total
```


## 6.3. external metric 추가

### Harbor?
Harbor is an open source cloud native registry that stores, signs, and scans container images. (like aws ecr)  



### Harbor 메트릭 수집하고 싶을 때 구성방법

Harbor server에 내 harbor 메트릭 추가
```
- job_name: harbor
      metrics_path: /metrics
      static_configs:
      - targets:
        - 192.168.1.63:9090
```

## 6.4. [recording rules](https://prometheus.io/docs/prometheus/latest/configuration/recording_rules) 추가
반복적으로 복잡한 PromQL 사용 시 미리 계산된 metric값으로 변수 customize  

### syntax checking  
```
$ install -m 755 prometheus-2.37.0.linux-amd64/promtool /usr/local/bin
$ promtool check rules /path/to/example.rules.yml
```

### recording_rules  
Prometheus Configuration에서 rule_files 경로 주고  
```
rule_files:
  - /etc/config/recording_rules.yml
```
rule_files 정의  
```
recording_rules.yml: |
  groups:
    - name: prometheus-recording.rules
      interval: 10s
      rules:
        - record: node:node_memory:usage
          expr: |-
            100 - 100 * (
              (node_memory_MemTotal_bytes
               - node_memory_MemFree_bytes
               - node_memory_Buffers_bytes
               - node_memory_Cached_bytes
               - node_memory_SReclaimable_bytes
              )
               /
               node_memory_MemTotal_bytes
            )
        - record: container:memory:topk3
          expr: topk(3,sum(container_memory_working_set_bytes{pod!=""}/1024/1024) by (pod))
```


## 6.5. [alerts](https://prometheus.io/docs/alerting/latest/configuration) 추가
기본적으로 recording rule과 같고 경고 메시지 서비스  

### alerting_rules  

helm upgrade를 통해서 alertmanager instance enable  
```
helm upgrade prometheus edu/prometheus \
--set pushgateway.enabled=false \
--set nodeExporter.tolerations[0].key="node-role.kubernetes.io/master" \
--set nodeExporter.tolerations[0].effect="NoSchedule" \
--set nodeExporter.tolerations[0].operator="Exists" \
--set nodeExporter.tolerations[1].key="node-role.kubernetes.io/control-plane" \
--set nodeExporter.tolerations[1].effect="NoSchedule" \
--set nodeExporter.tolerations[1].operator="Exists" \
--set server.service.type="LoadBalancer" \
--set alertmanager.service.type="LoadBalancer" \
--set alertmanager.service.loadBalancerIP="192.168.1.65" \
--set server.global.scrape_interval="15s" \
--set server.global.evaluation_interval="15s" \
--set server.extraFlags[0]="web.enable-lifecycle" \
--set server.extraFlags[1]="storage.tsdb.no-lockfile" \
--namespace=monitoring 
```

Prometheus Configuration에서 rule_files 경로와 alertmanager 서비스 띄우고  
```
rule_files:
  - /etc/config/alerting_rules.yml
```
rule_files 정의  
```
  alerting_rules.yml: |
    groups:
    - name: nginx-status.alert
      rules:
      - alert: '[P2] NginxDown'
        for: 30s
        annotations:
          title: 'Nginx pod down unexpectedly'
          description: 'Nginx가 비정상 종료됨. 빠른 조치 필요!'
          summary: '[P2, warnning]: Nginx pod has been shutdown unexpectedly.'
        expr: |
          (sum(nginx_up) OR vector(0)) == 0
```
Alerting Configuration에서 경고 메시지 보낼 곳 정의  
```
  alertmanager.yml: |
    global:
      resolve_timeout: 10m
      slack_api_url: "https://hooks.slack.com/services/..."# Slack-URL
    receivers:
    - name: default-receiver
    - name: slack
      slack_configs:
      - channel: #development
        send_resolved: true
        title: '[{{.Status | toUpper}}] {{ .CommonLabels.alertname }}'
        text: |
          *Description:* {{ .CommonAnnotations.description }}
    route:
      group_interval: 1m
      group_wait: 10s
      receiver: slack
      repeat_interval: 5m
```

---

# 6. Prometheus Operator
다양한 k8s cluster(eks,aks,gke)위에 
native prometheus와 prometheus operator 수집 대상이 다름  

## 6.1. exporter configuration
helm으로 operator(kube-prometheus-stack)설치  
```
helm install prometheus-stack edu/kube-prometheus-stack  \
--set defaultRules.create="false" \
--set alertmanager.enabled="false" \
--set grafana.enabled="false" \
--set prometheus.service.type="LoadBalancer" \
--set prometheus.service.port="80" \
--set prometheus.prometheusSpec.scrapeInterval="30s" \
--set prometheus.prometheusSpec.evaluationInterval="30s" \
--namespace=monitoring \
--create-namespace \
-f ~/prom-operator-config/set-sc-8Gi.yaml # volumn(StatefulSet) 정의
```
set-sc-8Gi.yaml 파일:  
```
prometheus:
  prometheusSpec:
    storageSpec:
      volumeClaimTemplate:
        spec:
          storageClassName: "managed-nfs-storage"
          accessModes:
          - "ReadWriteOnce"
          resources:
            requests:
              storage: "8Gi"
```

## 6.2. control plane exporter

## 6.3. service monitor exporter

## 6.4. pod monitor exporter





---
