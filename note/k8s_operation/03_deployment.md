Deploymnet is normally used for stateless services , it has  stateless migration , autoscale, disaster recovery, one command rollback functions. 
Most Microservice could use "Deployment"

```yaml
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

```bash
[root@k8s-master01 ~]# kubectl create -f dep.yaml
deployment.apps/nginx-deployment created

[root@k8s-master01 ~]# kubectl rollout status deployment/nginx-deployment
deployment "nginx-deployment" successfully rolled out

[root@k8s-master01 ~]# kubectl get rs -l app=nginx
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-5d59d67564   3         3         3       2m3s


[root@k8s-master01 ~]# kubectl get pod -o wide
NAME                                READY   STATUS              RESTARTS   AGE   IP       NODE           NOMINATED NODE   READINESS GATES
nginx-deployment-5d59d67564-7dshf   0/1     ContainerCreating   0          8s    <none>   k8s-node01     <none>           <none>
nginx-deployment-5d59d67564-g9xp9   0/1     ContainerCreating   0          8s    <none>   k8s-node02     <none>           <none>
nginx-deployment-5d59d67564-svgfl   0/1     ContainerCreating   0          8s    <none>   k8s-master02   <none>           <none>

[root@k8s-master01 ~]# kubectl get deploy
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/3     3            2           26s

[root@k8s-master01 ~]# kubectl get po --show-labels
NAME                                READY   STATUS    RESTARTS   AGE     LABELS
nginx-deployment-5d59d67564-7dshf   1/1     Running   0          2m56s   app=nginx,pod-template-hash=5d59d67564
nginx-deployment-5d59d67564-g9xp9   1/1     Running   0          2m56s   app=nginx,pod-template-hash=5d59d67564
nginx-deployment-5d59d67564-svgfl   1/1     Running   0          2m56s   app=nginx,pod-template-hash=5d59d67564
```

## Upgrade Deployment


Modify
```bash
[root@k8s-master01 ~]# kubectl edit deployment.v1.apps/nginx-deployment

for example change the deployment replicas...
```


```bash
kubectl set image deployment nginx-deployment nginx=nginx:1.9.1 --record 

kubectl rollout status deployment/nginx-deployment 《-check process

[root@k8s-master01 ~]# kubectl rollout status deployment/nginx-deployment

Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
deployment "nginx-deployment" successfully rolled out


[root@k8s-master01 ~]# kubectl describe deploy nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Tue, 09 Aug 2022 17:06:00 +0800
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 2
                        kubernetes.io/change-cause: kubectl set image deployment nginx-deployment nginx=nginx:1.9.1 --record=true
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.9.1
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-69c44dfb78 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  7m    deployment-controller  Scaled up replica set nginx-deployment-5d59d67564 to 3
  Normal  ScalingReplicaSet  2m3s  deployment-controller  Scaled up replica set nginx-deployment-69c44dfb78 to 1
  Normal  ScalingReplicaSet  96s   deployment-controller  Scaled down replica set nginx-deployment-5d59d67564 to 2
  Normal  ScalingReplicaSet  95s   deployment-controller  Scaled up replica set nginx-deployment-69c44dfb78 to 2
  Normal  ScalingReplicaSet  61s   deployment-controller  Scaled down replica set nginx-deployment-5d59d67564 to 1
  Normal  ScalingReplicaSet  61s   deployment-controller  Scaled up replica set nginx-deployment-69c44dfb78 to 3
  Normal  ScalingReplicaSet  39s   deployment-controller  Scaled down replica set nginx-deployment-5d59d67564 to 0


```


### 回滚操作



```bash
[root@k8s-master01 ~]# kubectl edit deployment.v1.apps/nginx-deployment --record=true
Flag --record has been deprecated, --record will be removed in the future
deployment.apps/nginx-deployment edited

[root@k8s-master01 ~]# kubectl rollout history deployment/nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl edit deployment.v1.apps/nginx-deployment --record=true
3         kubectl set image deployment nginx-deployment nginx=nginx:1.9.2 --record=true
```

```bash
[root@k8s-master01 ~]# kubectl rollout history deployment/nginx-deployment --revision=2
deployment.apps/nginx-deployment with revision #2
Pod Template:
  Labels:       app=nginx
        pod-template-hash=69c44dfb78
  Annotations:  kubernetes.io/change-cause: kubectl edit deployment.v1.apps/nginx-deployment --record=true
  Containers:
   nginx:
    Image:      nginx:1.9.1
    Port:       80/TCP
    Host Port:  0/TCP
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>
```
```bash

[root@k8s-master01 ~]# kubectl rollout undo deployment/nginx-deployment
deployment.apps/nginx-deployment rolled back
```
after roll back
```bash
[root@k8s-master01 ~]# kubectl rollout history deployment/nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
1         <none>
3         kubectl set image deployment nginx-deployment nginx=nginx:1.9.2 --record=true
4         kubectl edit deployment.v1.apps/nginx-deployment --record=true
```

## 扩容
```bash
kubectl scale deployment.v1.apps/nginx-deployment --replicas=10


[root@k8s-master01 ~]# kubectl get po
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-69c44dfb78-78t2l   1/1     Running   0          14s
nginx-deployment-69c44dfb78-94g6w   1/1     Running   0          14s
nginx-deployment-69c44dfb78-9n2mt   1/1     Running   0          102s
nginx-deployment-69c44dfb78-bk6rb   1/1     Running   0          112s
nginx-deployment-69c44dfb78-gfdgn   1/1     Running   0          111s
nginx-deployment-69c44dfb78-hr9z6   1/1     Running   0          14s
nginx-deployment-69c44dfb78-ksgt8   1/1     Running   0          107s
nginx-deployment-69c44dfb78-lvwwb   1/1     Running   0          15s
nginx-deployment-69c44dfb78-pfbh5   1/1     Running   0          15s
nginx-deployment-69c44dfb78-wthgz   1/1     Running   0          15s
```

## Pause Deployment
```bash
kubectl rollout pause deployment/nginx-deployment


kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1
kubectl set resources deployment.v1.apps/nginx-deployment -c=nginx --limits=cpu=200m,memory=512Mi

kubectl rollout history deployment.v1.apps/nginx-deployment 
发现并没有更新

kubectl rollout resume deployment.v1.apps/nginx-deployment
kubectl rollout history deployment.v1.apps/nginx-deployment 

```

```bash

[root@k8s-master01 ~]# kubectl describe deploy nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Tue, 09 Aug 2022 17:06:00 +0800
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 5
                        kubernetes.io/change-cause: kubectl edit deployment.v1.apps/nginx-deployment --record=true
Selector:               app=nginx
Replicas:               10 desired | 10 updated | 10 total | 10 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:      nginx:1.9.1
    Port:       80/TCP
    Host Port:  0/TCP
    Limits:
      cpu:        200m
      memory:     512Mi
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-f7fdb4ccd (10/10 replicas created)
Events:
  Type    Reason             Age                 From                   Message
  ----    ------             ----                ----                   -------
  Normal  ScalingReplicaSet  29m                 deployment-controller  Scaled up replica set nginx-deployment-5d59d67564 to 3
  Normal  ScalingReplicaSet  24m                 deployment-controller  Scaled down replica set nginx-deployment-5d59d67564 to 2
  Normal  ScalingReplicaSet  24m                 deployment-controller  Scaled up replica set nginx-deployment-69c44dfb78 to 2
  Normal  ScalingReplicaSet  23m                 deployment-controller  Scaled down replica set nginx-deployment-5d59d67564 to 1
  Normal  ScalingReplicaSet  23m                 deployment-controller  Scaled down replica set nginx-deployment-5d59d67564 to 0
  Normal  ScalingReplicaSet  16m                 deployment-controller  Scaled up replica set nginx-deployment-69c44dfb78 to 5
  Normal  ScalingReplicaSet  13m                 deployment-controller  Scaled down replica set nginx-deployment-69c44dfb78 to 4
  Normal  ScalingReplicaSet  10m (x2 over 24m)   deployment-controller  Scaled up replica set nginx-deployment-69c44dfb78 to 1
  Normal  ScalingReplicaSet  10m (x2 over 23m)   deployment-controller  Scaled up replica set nginx-deployment-69c44dfb78 to 3
  Normal  ScalingReplicaSet  36s (x19 over 12m)  deployment-controller  (combined from similar events): Scaled down replica set nginx-deployment-69c44dfb78 to 7
```
