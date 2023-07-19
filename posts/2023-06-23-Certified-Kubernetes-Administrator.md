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


# 1. Control Plane (Master Node)

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


## 1.2. [upgrade kubernetes](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
1.24.X -> 1.25.X
```
] kubectl get nodes
NAME          STATUS   ROLES                  AGE    VERSION
k8s-master    Ready    control-plane,master   126d   v1.24.10
k8s-worker1   Ready    <none>                 126d   v1.24.10
k8s-worker2   Ready    <none>                 126d   v1.24.10
```

### upgrade kubeadm
```
] sudo -i
] apt update
] apt-cache madison kubeadm # find the latest 1.25 version in the list (1.25.x-00, where x is the latest patch)
] apt-mark unhold kubeadm && \
  apt-get update && apt-get install -y kubeadm=1.25.x-00 && \
  apt-mark hold kubeadm # replace x in 1.25.x-00 with the latest patch version
] kubeadm version
] kubeadm upgrade plan
] sudo kubeadm upgrade apply v1.25.x # replace x with the patch version you picked for this upgrade
```

### drain the master node
```
] kubectl drain k8s-master --ignore-daemonsets
] kubectl get nodes
NAME          STATUS                     ROLES                  AGE    VERSION
k8s-master    Ready,SchedulingDisabled   control-plane,master   126d   v1.24.10
k8s-worker1   Ready                      <none>                 126d   v1.24.10
k8s-worker2   Ready                      <none>                 126d   v1.24.10
```

### upgrade kubelet and kubectl
```
] apt-mark unhold kubelet kubectl && \
  apt-get update && apt-get install -y kubelet=1.25.x-00 kubectl=1.25.x-00 && \
  apt-mark hold kubelet kubectl # replace x in 1.25.x-00 with the latest patch version
] sudo systemctl daemon-reload
] sudo systemctl restart kubelet
] kubectl uncordon k8s-master
] kubectl get nodes
NAME          STATUS                     ROLES                  AGE    VERSION
k8s-master    Ready                      control-plane,master   126d   v1.25.x
k8s-worker1   Ready                      <none>                 126d   v1.24.10
k8s-worker2   Ready                      <none>                 126d   v1.24.10
```


### option: upgrade worker nodes
```
] apt-mark unhold kubeadm && \
  apt-get update && apt-get install -y kubeadm=1.25.x-00 && \
  apt-mark hold kubeadm # replace x in 1.25.x-00 with the latest patch version
] sudo kubeadm upgrade node
] kubectl drain k8s-worker1 --ignore-daemonsets
] apt-mark unhold kubelet kubectl && \
  apt-get update && apt-get install -y kubelet=1.25.x-00 kubectl=1.25.x-00 && \
  apt-mark hold kubelet kubectl # replace x in 1.25.x-00 with the latest patch version
] sudo systemctl daemon-reload
] sudo systemctl restart kubelet
] kubectl uncordon k8s-worker1
] kubectl get nodes
NAME          STATUS                     ROLES                  AGE    VERSION
k8s-master    Ready                      control-plane,master   126d   v1.25.x
k8s-worker1   Ready                      <none>                 126d   v1.25.x
```

## 1.3. diagnosis and troubleshooting worker node

```
] ssh k8s-wocker1
] sudo -i
] systemctl status docker
enabled
] systemctl status kubelet
disabled
] systemctl enable --now kubelet
] systemctl status kubelet
enabled
```

```
] kubectl get pod -n kube-system -o wide # check CNI, kube-proxy
flannel or calico is running
kube-proxy is running
] kubectl get nodes # worker node should be ready
```

```
] ssh k8s-wocker1
] sudo -i
] systemctl status docker
inactive
] systemctl enable --now docker
] 
```



# 2. Worker Node

## 2.1. creating Pod

```
] kubectl create namespace ecommerce
] kubectl -n ecommerce run eshop-main --image=nginx:1.17 --env=DB=mysql
```


## 2.2. creating Static Pod
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



## 2.3. creating Multi-container Pod

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


## 2.4. creating Sidecar container

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



## 2.5. deployment & Pod Scale

[scale reference docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#scale)

### pod scale out
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
] kubectl get pod -l app=webui
] kubectl get pod -l app=webui --show-labels
] kubectl describe pod nginx-86f46f47c4-vfw8f
Containers:
  nginx:
    Port: if not exist, default 80
] cat my-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
  type: NodePort
  selector:
    app: webui
  ports:
    - port: 80
      targetPort: 80
      nodePort: 32767
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
] kubectl apply -f my-service.yaml
] kubectl get svc my-service
] kubectl get node
] curl k8s-worker1:32767 # check connection
```


## 2.15. manage configMap

### [create configmaps from literal values](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#create-configmaps-from-literal-values)
```
] kubectl create ns ckad
] kubectl get ns ckad
] kubectl -n ckad create configmap web-config --from-literal=connection_string=localhost:80 --from-literal=external_url=cncf.io
] kubectl -n ckad describe configmaps web-config
```
### opt1 [define a container env variable with data from a single configmap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#define-a-container-environment-variable-with-data-from-a-single-configmap)
```
] kubectl -n ckad run web-pod --image=nginx:1.19.8-alpine --port=80 --dry-run=client -o yaml > web-pod.yaml
] cat web-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
  namespace: ckad
spec:
  containers:
    - name: web-pod
      image: nginx:1.19.8-alpine
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
      env:
        - name: CONNECTION_STRING
          valueFrom:
            configMapKeyRef:
              name: web-config
              key: connection_string
        - name: EXTERNAL_URL
          valueFrom:
            configMapKeyRef:
              name: web-config
              key: external_url
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
      ports:
      - containerPort: 80
] kubectl apply -f web-pod.yaml
] kubectl -n ckad exec web-pod -- env # check env
```
### opt2 [configure all key value pairs in a configmap as container env variables](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables)
```
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
  namespace: ckad
spec:
  containers:
    - name: web-pod
      image: nginx:1.19.8-alpine
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
      envFrom:
      - configMapRef:
          name: web-config
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
      ports:
      - containerPort: 80
```

### opt3 [add configmap data to a specific path in the volume](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#add-configmap-data-to-a-specific-path-in-the-volume)
```
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
  namespace: ckad
spec:
  containers:
    - name: web-pod
      image: nginx:1.19.8-alpine
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: web-config
        items:
        - key: CONNECTION_STRING
          path: connection_string
        - key: EXTERNAL_URL
          path: external_url
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
```


## 2.16. manage secret

### [define container env variables using secret data](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#define-container-environment-variables-using-secret-data)
```
] kubectl create secret --help
docker-registry
generic
tls
] kubectl create secret generic super-secret --from-literal=password=secretpass
] kubectl get secrets super-secret -o yaml
```

### opt1 [using secrets as file from a pod](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-files-from-a-pod)
```
] cat pod-secrets-via-file.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-secrets-via-file
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: secret-data
      mountPath: "/secrets"
      readOnly: true
  volumes:
  - name: secret-data
    secret:
      secretName: super-secret
      optional: true
] kubectl apply -f pod-secrets-via-file.yaml
] kubectl exec pod-secrets-via-file -- env
```

### opt2 [using secrets as env variables](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables)
```
] cat pod-secrets-via-env.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-secrets-via-env
spec:
  containers:
  - name: mypod
    image: redis
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
      env:
        - name: PASSWORD
          valueFrom:
            secretKeyRef:
              name: super-secret
              key: password
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
] kubectl apply -f pod-secrets-via-env.yaml
] kubectl exec pod-secrets-via-env -- env
```



## 2.17. configure ingress
Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

```
] kubectl -n ingress-nginx get pod
] kubectl -n ingress-nginx run nginx --image=nginx --labels=app=nginx --dry-run=client -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
  namespace: ingress-nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
] kubectl -n ingress-nginx run nginx --image=nginx --labels=app=nginx
] kubectl -n ingress-nginx expose pod nginx --port=80 --target-port=80 --dry-run=client -o yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
  namespace: ingress-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
status:
  loadBalancer: {}
] kubectl -n ingress-nginx expose pod nginx --port=80 --target-port=80
] kubectl -n ingress-nginx get svc
```

### [ingress resource](https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource)
```
] kubectl -n ingress-nginx get svc # check port
] vi app-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: ingress-nginx
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    kubernetes.id/ingress.class: nginx
spec:
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
      - path: /app
        pathType: Prefix
        backend:
          service:
            name: appjs-service
            port:
              number: 80
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
] kubectl -n ingress-nginx apply -f app-ingress.yaml
] kubectl -n ingress-nginx get ingress
] kubectl get nodes # check node name
] curl k8s-worker1:30080/
] curl k8s-worker1:30080/app
```



## 2.18. create persistent volume 
A [PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes) (PV) is a piece of storage in the cluster that has been provisioned by an administrator.
A [PersistentVolumeClaim](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim) (PVC) is a request for storage by a user.
A [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/#the-storageclass-resource) provides a way for administrators to describe the "classes" of storage they offer. It contains the fields provisioner, parameters, and reclaimPolicy, which are used when a PersistentVolume belonging to the class needs to be dynamically provisioned.


### Types of Persistent Volumes 
PersistentVolume types are implemented as plugins. Kubernetes currently supports the following plugins:

cephfs - CephFS volume
csi - Container Storage Interface (CSI)
fc - Fibre Channel (FC) storage
[hostPath](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath) - HostPath volume (for single node testing only; WILL NOT WORK in a multi-node cluster; consider using local volume instead)
iscsi - iSCSI (SCSI over IP) storage
local - local storage devices mounted on nodes.
nfs - Network File System (NFS) storage
rbd - Rados Block Device (RBD) volume


```
] vi app-config-pv.yaml
] kubectl apply -f app-config-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: app-config
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  storageClassName: az-c
  hostPath:
    path: /srv/app-config
] kubectl get pv
NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM     STORAGECLASS    REASON    AGE
app-config  1Gi        RWX            Retain           Available            az-c                      5s
```

```
] vi app-volume-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-volume
spec:
  storageClassName: app-hostpath-sc
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Mi
] kubectl apply -f app-volume-pvc.yaml
] kubectl get pvc # since no 10Mi, allot 1Gi
NAMESPACE       NAME                  STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS        AGE
default         app-volume            Bound    pv1      1Gi        ROX,RWX        app-hostpaht-sc     5s
] kubectl get pv # status to Bound
NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM               STORAGECLASS    REASON    AGE
pv1         1Gi        ROX,RWX        Recycle          Bound      default/app-volume  app-hostpath-sc           5m
```

### [claim as volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes)
```
] vi web-server-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-server-pod
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
      - mountPath: "/usr/share/nginx/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: app-volume
] kubectl apply -f web-server-pod.yaml
] kubectl describe pod web-server-pod
```


## 2.19. [viewing and finding resources](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#viewing-and-finding-resources)

```
] kubectl get pv --sort-by=.metadata.name
] kubectl get pv --sort-by=.metadata.name > /var/CKA2023/my_volumes
```






