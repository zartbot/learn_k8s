Replication Controller用来确保Replicas定义的数量

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

```bash
[root@k8s-master01 ~]# kubectl create -f rc.yaml
replicationcontroller/nginx created
[root@k8s-master01 ~]# kubectl get po -n default
NAME          READY   STATUS              RESTARTS   AGE
nginx-6s2q4   0/1     ContainerCreating   0          10s
nginx-b5qds   0/1     ContainerCreating   0          10s
nginx-zlk5t   0/1     ContainerCreating   0          10s
```

```bash
[root@k8s-master01 ~]# kubectl get po   -n default -o wide
NAME          READY   STATUS    RESTARTS   AGE     IP             NODE           NOMINATED NODE   READINESS GATES
nginx-6s2q4   1/1     Running   0          4m58s   172.17.125.3   k8s-node01     <none>           <none>
nginx-b5qds   1/1     Running   0          4m58s   172.25.92.66   k8s-master02   <none>           <none>
nginx-zlk5t   1/1     Running   0          4m58s   172.17.125.2   k8s-node01     <none>           <none>

[root@k8s-master01 ~]# kubectl delete pod nginx-zlk5t
pod "nginx-zlk5t" deleted


[root@k8s-master01 ~]# kubectl get po   -n default -o wide
NAME          READY   STATUS              RESTARTS   AGE     IP             NODE           NOMINATED NODE   READINESS GATES
nginx-2vpt9   0/1     ContainerCreating   0          6s      <none>         k8s-master02   <none>           <none>
nginx-6s2q4   1/1     Running             0          7m28s   172.17.125.3   k8s-node01     <none>           <none>
nginx-b5qds   1/1     Running             0          7m28s   172.25.92.66   k8s-master02   <none>           <none>

```

```bash
[root@k8s-master01 ~]# kubectl describe replicationcontrollers/nginx
Name:         nginx
Namespace:    default
Selector:     app=nginx
Labels:       app=nginx
Annotations:  <none>
Replicas:     5 current / 5 desired
Pods Status:  5 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                    Message
  ----    ------            ----   ----                    -------
  Normal  SuccessfulCreate  17m    replication-controller  Created pod: nginx-zlk5t
  Normal  SuccessfulCreate  17m    replication-controller  Created pod: nginx-b5qds
  Normal  SuccessfulCreate  17m    replication-controller  Created pod: nginx-6s2q4
  Normal  SuccessfulCreate  9m48s  replication-controller  Created pod: nginx-2vpt9
  Normal  SuccessfulCreate  115s   replication-controller  Created pod: nginx-wkqz5
  Normal  SuccessfulCreate  115s   replication-controller  Created pod: nginx-fk9fz
```
Modify replicas to 1, then apply

```bash
[root@k8s-master01 ~]# kubectl apply -f rc.yaml
replicationcontroller/nginx configured
[root@k8s-master01 ~]# kubectl describe replicationcontrollers/nginx
Name:         nginx
Namespace:    default
Selector:     app=nginx
Labels:       app=nginx
Annotations:  <none>
Replicas:     1 current / 1 desired
Pods Status:  1 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                    Message
  ----    ------            ----   ----                    -------
  Normal  SuccessfulCreate  18m    replication-controller  Created pod: nginx-zlk5t
  Normal  SuccessfulCreate  18m    replication-controller  Created pod: nginx-b5qds
  Normal  SuccessfulCreate  18m    replication-controller  Created pod: nginx-6s2q4
  Normal  SuccessfulCreate  10m    replication-controller  Created pod: nginx-2vpt9
  Normal  SuccessfulCreate  2m53s  replication-controller  Created pod: nginx-wkqz5
  Normal  SuccessfulCreate  2m53s  replication-controller  Created pod: nginx-fk9fz
  Normal  SuccessfulDelete  5s     replication-controller  Deleted pod: nginx-wkqz5
  Normal  SuccessfulDelete  5s     replication-controller  Deleted pod: nginx-2vpt9
  Normal  SuccessfulDelete  5s     replication-controller  Deleted pod: nginx-6s2q4
  Normal  SuccessfulDelete  5s     replication-controller  Deleted pod: nginx-fk9fz
```

## ReplicaSet
```bash




ReplicaSet

```yaml
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
      - {key: tier, operator: In, values: [frontend]}
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        env:
        - name: GET_HOSTS_FROM
          value: dns
        ports:
        - containerPort: 80
```

```bash

[root@k8s-master01 ~]# kubectl create -f rs.yaml
replicaset.apps/frontend created
You have new mail in /var/spool/mail/root
[root@k8s-master01 ~]# kubectl get pod
NAME             READY   STATUS              RESTARTS   AGE
frontend-6flb6   0/1     ContainerCreating   0          5s
frontend-csmdh   0/1     ContainerCreating   0          5s
frontend-vqpws   0/1     ContainerCreating   0          5s
[root@k8s-master01 ~]# kubectl get pod -o wide
NAME             READY   STATUS              RESTARTS   AGE   IP       NODE           NOMINATED NODE   READINESS GATES
frontend-6flb6   0/1     ContainerCreating   0          10s   <none>   k8s-master02   <none>           <none>
frontend-csmdh   0/1     ContainerCreating   0          10s   <none>   k8s-master02   <none>           <none>
frontend-vqpws   0/1     ContainerCreating   0          10s   <none>   k8s-node01     <none>           <none>
```


Replication controller  基本上在生产环境中看不到了，  replication set也很少单独使用，而更多的是配合Deployment、DaemonSet、StatefulSet管理POd

