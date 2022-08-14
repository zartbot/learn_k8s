
## Auto Scale
Deployment

```yaml
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
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
        resources:
          limits:
            cpu: 20m
            memory: 200Mi
          requests:
            cpu: 10m
            memory: 100Mi        
        ports:
        - containerPort: 80
```

```bash

[root@k8s-master01 ~]# kubectl apply -f dep.yaml
deployment.apps/nginx-deployment created
[root@k8s-master01 ~]# kubectl get po
NAME                                READY   STATUS              RESTARTS   AGE
nginx-deployment-6bff76cf94-7w2cb   0/1     ContainerCreating   0          10s

[root@k8s-master01 ~]# kubectl expose deployment nginx-deployment --port=80
service/nginx-deployment exposed


[root@k8s-master01 ~]# kubectl autoscale deployment nginx-deployment --cpu-percent=10 --min=1 --max=10
horizontalpodautoscaler.autoscaling/nginx-deployment autoscaled

[root@k8s-master01 ~]# kubectl get hpa
NAME               REFERENCE                     TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx-deployment   Deployment/nginx-deployment   0%/10%    1         10        1          19s

```


another tty 
```bash

[root@k8s-master01 ~]# kubectl get svc
NAME               TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes         ClusterIP   10.99.0.1      <none>        443/TCP   6d17h
my-service         ClusterIP   10.99.97.118   <none>        80/TCP    4d17h
nginx-deployment   ClusterIP   10.99.155.30   <none>        80/TCP    2m35s
[root@k8s-master01 ~]# while true; do wget -q -O- http://10.99.155.30 > /dev/null;done


[root@k8s-master01 ~]# kubectl get hpa
NAME               REFERENCE                     TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
nginx-deployment   Deployment/nginx-deployment   190%/10%   1         10        4          2m35s
[root@k8s-master01 ~]# kubectl get hpa
NAME               REFERENCE                     TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
nginx-deployment   Deployment/nginx-deployment   160%/10%   1         10        8          2m50s

[root@k8s-master01 ~]# kubectl get po
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6bff76cf94-2kqgg   1/1     Running   0          35s
nginx-deployment-6bff76cf94-2sbj2   1/1     Running   0          49s
nginx-deployment-6bff76cf94-44nfx   1/1     Running   0          19s
nginx-deployment-6bff76cf94-6jd5m   1/1     Running   0          34s
nginx-deployment-6bff76cf94-7w2cb   1/1     Running   0          5m11s
nginx-deployment-6bff76cf94-88k4z   1/1     Running   0          19s
nginx-deployment-6bff76cf94-8m2zb   1/1     Running   0          34s
nginx-deployment-6bff76cf94-jz8xx   1/1     Running   0          49s
nginx-deployment-6bff76cf94-sf976   1/1     Running   0          49s
nginx-deployment-6bff76cf94-zxl5l   1/1     Running   0          34s


```

