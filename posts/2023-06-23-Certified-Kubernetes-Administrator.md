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

## 2.6. rolling update & roll back
### [rolling update](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)
```
] kubectl create deployment nginx-app --image=nginx:1.11.10-alpine --replicas=3 --dry-run=client -o yaml > nginx-app.yaml
] vi nginx-app.yaml
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - image: nginx:1.11.10-alpine
        name: nginx
] kubectl create deployment nginx-app --image=nginx:1.11.10-alpine --replicas=3
] kubectl set image deployment/nginx-app nginx(container name)=nginx:1.11.13-alpine(container image)
] kubectl rollout status deployment/nginx-app # check the status
] kubectl rollout history deployment/nginx-app # check the history
```

### [roll back](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-to-a-previous-revision)
```
] kubectl rollout undo deployment/nginx-app --to-revision=2 (go back to specific version)
] kubectl rollout history deployment/nginx-app
```

## 2.7. node selector
### [list and watch filtering](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#list-and-watch-filtering)
```
] kubectl get nodes --show-labels
] kubectl get nodes -L disktype (check only disktype column)
```

```
] kubectl run eshop-store --image=nginx --dry-run=client -o yaml > eshop-store.yaml
] vi eshop-store.yaml
metadata:
  name: eshop-store
spec:
  containers:
  - image: nginx
    name: eshop-store
```

### [add a label to a node](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/#add-a-label-to-a-node)
```
metadata:
  name: eshop-store
spec:
  containers:
  - image: nginx
    name: eshop-store
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
  nodeSelector:
    disktype: ssd
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
] kubectl apply -f eshop-store.yaml
] kubectl get pods -o wide eshop-store
```


## 2.8. node management

### [manual node administration](https://kubernetes.io/docs/concepts/architecture/nodes/#manual-node-administration)
```
] kubectl get nodes -o wide | grep worker1
] kubectl describe node k8s-worker1
] kubectl cordon k8s-worker1 # SchedulingDisabled on worker1 node
NAME          STATUS
k8s-worker1   Ready,SchedulingDisabled
] kubectl scale deployment nginx --replicas=6
] kubectl get pods -o wide # only deployed in worker2 node
```
```
] kubectl uncordon k8s-worker1
] kubectl get nodes -o wide
NAME          STATUS
k8s-worker1   Ready
```

### [remove a node from service](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/#use-kubectl-drain-to-remove-a-node-from-service)
```
] kubectl drain k8s-worker2 # relocate all pods from worker2 to worker1 # daemonsets running only specified worker2 node cannot be deleted
] kubectl drain k8s-worker2 --ignore-daemonsets # ignore daemonset e.g. kube-system/kube-flannel, kube-proxy
] kubectl drain k8s-worker2 --ignore-daemonsets --force # continue even if there are pods not managed by a ReplicationController, Job, or DaemonSet
] kubectl drain k8s-worker2 --ignore-daemonsets --force --delete-emptydir-data # approve to drain pods with local storage
] kubectl get nodes -o wide
NAME          STATUS
k8s-worker2   Ready,SchedulingDisabled
```
```
] kubectl uncordon k8s-worker2
] kubectl get nodes -o wide
NAME          STATUS
k8s-worker2   Ready
] kubectl scale deployment nginx --replicas=1
] kubectl scale deployment nginx --replicas=3
] kubectl get pods -o wide # only deployed in worker2 node
```



## 2.9. collect node info

```
] kubectl get nodes | grep -iw ready
k8s-master    Ready    control-plane,master   121d   v1.24.10
k8s-worker1   Ready    <none>                 121d   v1.24.10
] kubectl get nodes | grep -iw ready | wc -l > /var/CKA2023/RN0001
```
```
] kubectl describe node k8s-master | grep -i NoSchedule
Taints:             node-role.kubernetes.io/master:NoSchedule
] kubectl describe node k8s-worker1 | grep -i NoSchedule
] kubectl describe node k8s-worker1 | grep -i taints
Taints:             <none>
] echo "1" > /var/CKA2023/RN0001
```


## 2.10. deployment & expose the service

### [field spec ports](https://kubernetes.io/docs/concepts/services-networking/service/#field-spec-ports)
```
] kubectl get deployments.apps front-end -o yaml > front-end.yaml
] vi front-end.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: front-end
spec:
  replicas: 2
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx:stable
        name: nginx
        ports:
        - containerPort: 80
          name: http
---
apiVersion: v1
kind: Service
metadata:
  name: front-end-svc
spec:
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
  type: NodePort
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
  selector:
    run: nginx
  ports:
  - name: nginx-http
    protocol: TCP
    port: 80
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
    targetPort: http # not 80 since the port named http
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
```

```
] kubectl get deployments.apps front-end
] kubectl get svc front-end-svc
] curl k8s-worker1:31779 #nodename:nodeport
```



## 2.11. extract pod log

### [logging architecture](https://kubernetes.io/docs/concepts/cluster-administration/logging/)
```
] kubectl get pods custom-app
] kubectl logs custom-app | grep 'file not found' > /var/CKA2023/podlog
] cat /var/CKA2023/podlog
```

### [reference logs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs)
```
] kubectl logs custom-app --all-containers=true
] kubectl logs custom-app -p -c ruby web-1 # in previous terminated, container ruby
```



## 2.12. check pods overloaded cpu

### [reference tops](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#top)
```
] kubectl top nodes
] kubectl top pods
] kubectl top pods -l name=overloaded-cpu
] kubectl top pods -l name=overloaded-cpu --sort-by=cpu
```


## 2.13. init container
specialized containers that run before app containers in a Pod. Init containers can contain utilities or setup scripts not present in an app image.  
 Init containers always run to completion.  
 Each init container must complete successfully before the next one starts.  

### [init container in use](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#init-containers-in-use)
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

```
] cat /data/cka/webpod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: main
    image: busybox:1.28
    command: ['sh', '-c', 'if [ !-f /workdir/data/txt ];then exit 1;else sleept 300;fi']
    volumeMounts:
    - name: workdir
      mountPath: /workdir
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
  initContainers:
  - name: init
    image: busybox:1.28
    command: ['sh', '-c', 'touch /workdir/data.txt']
    volumeMounts:
    - name: workdir
      mountPath: /workdir
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
  volumes:
  - name: workdir
    emptyDir: {}
] kubectl apply -f /data/cka/webpod.yaml
] kubectl get pods
] kubectl exec web-pod -c main -- ls -l /workdir/data.txt
```


## 2.14. create service nodeport

### [nodeport](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport)
Each node proxies that port (the same port number on every Node) into your Service. Your Service reports the allocated port in its .spec.ports[*].nodePort field.
```
] 
] 
] 
```


