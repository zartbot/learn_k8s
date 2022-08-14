

```bash

[root@k8s-master01 ~]# kubectl label node k8s-node02 region=subnet7
node/k8s-node02 labeled
[root@k8s-master01 ~]# kubectl label node k8s-node01 region=subnet7
node/k8s-node01 labeled

[root@k8s-master01 ~]# kubectl get no -l region=subnet7
NAME         STATUS   ROLES    AGE   VERSION
k8s-node01   Ready    <none>   29h   v1.22.0
k8s-node02   Ready    <none>   29h   v1.22.0
```

```yaml
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 5
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
      dnsPolicy: ClusterFirst
      nodeSelector:
        region: subnet7        
```        

```bash
[root@k8s-master01 ~]# kubectl get po -o wide
NAME                               READY   STATUS              RESTARTS   AGE   IP              NODE         NOMINATED NODE   READINESS GATES
nginx-deployment-95b8b9d79-2q5cn   1/1     Running             0          6s    172.17.125.36   k8s-node01   <none>           <none>
nginx-deployment-95b8b9d79-grfl4   1/1     Running             0          6s    172.17.125.37   k8s-node01   <none>           <none>
nginx-deployment-95b8b9d79-k6mfk   0/1     ContainerCreating   0          5s    <none>          k8s-node01   <none>           <none>
nginx-deployment-95b8b9d79-ncw8h   1/1     Running             0          6s    172.27.14.209   k8s-node02   <none>           <none>
nginx-deployment-95b8b9d79-vfzn2   0/1     ContainerCreating   0          5s    <none>          k8s-node02   <none>           <none>
```

Service Label
```bash
[root@k8s-master01 ~]# kubectl get service --show-labels
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   LABELS
kubernetes   ClusterIP   10.99.0.1    <none>        443/TCP   2d    component=apiserver,provider=kubernetes
nginx        ClusterIP   None         <none>        80/TCP    95m   app=nginx
[root@k8s-master01 ~]#
[root@k8s-master01 ~]# kubectl label svc nginx env=kevin version=v1
service/nginx labeled
[root@k8s-master01 ~]# kubectl get service --show-labels
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   LABELS
kubernetes   ClusterIP   10.99.0.1    <none>        443/TCP   2d    component=apiserver,provider=kubernetes
nginx        ClusterIP   None         <none>        80/TCP    96m   app=nginx,env=kevin,version=v1


[root@k8s-master01 ~]# kubectl get service --show-labels --all-namespaces
NAMESPACE              NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE   LABELS
default                kubernetes                  ClusterIP   10.99.0.1       <none>        443/TCP                  2d    component=apiserver,provider=kubernetes
default                nginx                       ClusterIP   None            <none>        80/TCP                   97m   app=nginx,env=kevin,version=v1
kube-system            calico-typha                ClusterIP   10.99.219.47    <none>        5473/TCP                 29h   k8s-app=calico-typha
kube-system            kube-dns                    ClusterIP   10.99.0.10      <none>        53/UDP,53/TCP,9153/TCP   29h   k8s-app=kube-dns,kubernetes.io/cluster-service=true,kubernetes.io/name=CoreDNS
kube-system            metrics-server              ClusterIP   10.99.63.143    <none>        443/TCP                  28h   k8s-app=metrics-server
kubernetes-dashboard   dashboard-metrics-scraper   ClusterIP   10.99.174.216   <none>        8000/TCP                 28h   k8s-app=dashboard-metrics-scraper
kubernetes-dashboard   kubernetes-dashboard        NodePort    10.99.215.119   <none>        443:30251/TCP            28h   k8s-app=kubernetes-dashboard

[root@k8s-master01 ~]# kubectl get service --show-labels --all-namespaces -l version=v1
NAMESPACE   NAME    TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   LABELS
default     nginx   ClusterIP   None         <none>        80/TCP    97m   app=nginx,env=kevin,version=v1

```
Overwrite
```bash

[root@k8s-master01 ~]# kubectl label svc nginx env=kevin version=v2 --overwrite
service/nginx labeled
[root@k8s-master01 ~]#  kubectl get service --show-labels
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE    LABELS
kubernetes   ClusterIP   10.99.0.1    <none>        443/TCP   2d     component=apiserver,provider=kubernetes
nginx        ClusterIP   None         <none>        80/TCP    101m   app=nginx,env=kevin,version=v2
[root@k8s-master01 ~]# kubectl label svc nginx env=kevin version=v3
'env' already has a value (kevin), and --overwrite is false
'version' already has a value (v2), and --overwrite is false



```
Delete

```bash

[root@k8s-master01 ~]# kubectl label svc nginx env-
service/nginx labeled
[root@k8s-master01 ~]#  kubectl get service --show-labels
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE    LABELS
kubernetes   ClusterIP   10.99.0.1    <none>        443/TCP   2d     component=apiserver,provider=kubernetes
nginx        ClusterIP   None         <none>        80/TCP    102m   app=nginx,version=v2
```

### Selector

```bash

[root@k8s-master01 ~]# kubectl get service --show-labels --all-namespaces -l 'k8s-app in (metrics-server,calico-typha)'
NAMESPACE     NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE   LABELS
kube-system   calico-typha     ClusterIP   10.99.219.47   <none>        5473/TCP   29h   k8s-app=calico-typha
kube-system   metrics-server   ClusterIP   10.99.63.143   <none>        443/TCP    29h   k8s-app=metrics-server
```


