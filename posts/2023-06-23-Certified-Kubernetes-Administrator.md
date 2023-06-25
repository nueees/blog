---
aliases:
- /kubernetes/2023/06/23/Certified-Kubernetes-Administrator
categories:
- kubernetes
date: '2023-06-23'
description: Certified Kubernetes Administrator
layout: post
title: Certified Kubernetes Administrator
toc: false

---

[따배씨](https://www.youtube.com/watch?v=dv_5WCYS5P8&list=PLApuRlvrZKojqx9-wIvWP3MPtgy2B372f&index=4)


# 1. Control Plane

## 1.1. ETCD Backup&Restore

```
] ssh k8s-master
] etcd --version
] etcdctl version
```

### [snapshot using etcdctl options](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#snapshot-using-etcdctl-options)
```
] sudo ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=<trusted-ca-file> \
  --cert=<cert-file> --key=<key-file> \
  snapshot save <backup-file-location>
Snapshot saved at <backup-file-location>
```

### [restoring an etcd cluster](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#restoring-an-etcd-cluster)
```
] sudo ETCDCTL_API=3 etcdctl snapshot restore \
  --data-dir <new-data-dir-location> <backup-file-location>
```
```
] sudo vi /etc/kubernetes/manifests/etcd.yaml
volumes:
  - hostPath:
    path: /var/lib/<new-data-dir-location> # edit
] sudo docker ps -a | grep etcd
```


# 2. Worker Node

## 2.1. Creating Pod

```
] kubectl create namespace ecommerce
] kubectl -n ecommerce run eshop-main --image=nginx:1.17 --env=DB=mysql
```


## 2.2. Creating Static Pod
Static Pod : directly access to kubelet in worker node without control plane API
### yaml code extraction for web-pod
```
] kubectl -n ecommerce run web --image=nginx:1.17 -o yaml
```
save ouptut 

```
] ssh hk8s-w1
] sudo -i # root mode
] cat /var/lib/kubelet/config.yaml
staticPodPath: /etc/kubernetes/manifests
] cd /etc/kubernetes/manifests
] cat > web-pod.yaml
add saved output in the above step
] exit
] kubectl get pods # check the static pod
```



## 2.3. Creating Multi-container Pod

```
] kubectl run lab004 --image=nginx --dry-run=client -o yaml
] cat > multi.yaml
spec:
  containers:
  - image: nginx
    name: nginx
  - image: redis
    name: redis
  - image: memcached
    name: memcached
] kubectl apply -f multi.yaml
] kubectl describe pod lab004
```


## 2.4. Creating Sidecar container

Sidecar Container: built-in logging architecture  
1) log /var/log/html in nginx container (main container)  
2) store the log to a separated volume in /varlog  
3) access /varlog in side-car container(log analyzing purpose)  

```
] kubectl get pod eshop-cart-app # main container
] kubectl get pod eshop-cart-app -o yaml > eshop.yaml

```

### [sidecar container with logging agent](https://kubernetes.io/docs/concepts/cluster-administration/logging/#sidecar-container-with-logging-agent)

```
] vi eshop.yaml
spec:
  containers:
  - command:
    - /bin/sh
    - -c
    - 'i=1;while :;do  echo -e "$i: Price: $((RANDOM % 10000 + 1))" >> /var/log/cart-app.log;i=$((i+1)); sleep 2; done'
    image: busybox
    name: cart-app
    volumeMounts:
    - mountPath: /var/log
      name: varlog
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
  - name: sidecar
    image: busybox
    args: [/bin/sh, -c, 'tail -n+1 -F /var/log/cart-app.log']
    volumeMounts:
    - name: varlog
      mountPath: /var/log
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
  volumes:
  - emptyDir: {}
    name: varlog

] kubectl delete pod eshop-cart-app # (--force)
] kubectl apply -f eshop.yaml
] kubectl logs eshop-cart-app -c sidecar
```



## 2.5. Deployment & Pod Scale

[scale reference docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#scale)

### Pod scale out
```
] kubectl -n devops get pod deployments.apps
] kubectl -n devops scale deployment eshop-order --replicas=5
] kubectl -n devops get deploy
```

### create and scale out deployment
```
] kubectl create deployment webserver --image=nginx:1.14 --replicas=2 --dry-run=client -o yaml > webserver.yaml
] vi webserver.yaml
spec:
  replicas: 2
  selector:
    matchLabels:
      app_env_stage=dev
  template:
    metadata:
      labels:
        app_env_stage=dev
    spec:
      containers:
      - image: nginx:1.14
        name: webserver
        ports:
        - containerPort: 80
] kubectl apply -f webserver.yaml
] kubectl get deployments.apps -o wide
```

```
] kubectl scale deployment webserver --replicas=3
] kubectl get pods --show-labels
```


