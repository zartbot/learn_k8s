headerless service format

```yaml
statefulSetName-{0..N-1}.serviceName.namespace.svc.cluster.local
```
for example redis-master has id: redis-ms-0, then the pod-name set same as id,  the slave node could use  the same service nameredis-ms-0.redis-ms.public-service.svc.cluster.local point to the master.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
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
        image: nginx
        ports:
        - containerPort: 80
          name: web
```
          
```bash

[root@k8s-master01 ~]# kubectl create -f sts.yaml
service/nginx created
statefulset.apps/web created

[root@k8s-master01 ~]# kubectl get sts
NAME   READY   AGE
web    1/2     9s

[root@k8s-master01 ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.99.0.1    <none>        443/TCP   46h
nginx        ClusterIP   None         <none>        80/TCP    13s


[root@k8s-master01 ~]# kubectl get pod
NAME    READY   STATUS    RESTARTS   AGE
web-0   1/1     Running   0          17s
web-1   1/1     Running   0          10s
```
Notice the pod name,especially during rescale

```bash

[root@k8s-master01 ~]# kubectl scale sts web --replicas=10
statefulset.apps/web scaled

Pod create one by one


[root@k8s-master01 ~]# kubectl get pod
NAME    READY   STATUS    RESTARTS   AGE
web-0   1/1     Running   0          3m33s
web-1   1/1     Running   0          3m26s
web-2   1/1     Running   0          2m7s
web-3   1/1     Running   0          117s
web-4   1/1     Running   0          108s
web-5   1/1     Running   0          100s
web-6   1/1     Running   0          90s
web-7   1/1     Running   0          82s
web-8   1/1     Running   0          60s
web-9   1/1     Running   0          51s
You have new mail in /var/spool/mail/root


```

### use patch modify

```bash
kubectl patch sts web -p '{"spec":{"replicas":3}}'
```

## Roll update

```bash
kubectl patch statefulset web -p '{"spec":{"updateStrategy":{"type":"RollingUpdate"}}}'
kubectl get sts web -o yaml | grep -A 1 "updateStrategy"

kubectl patch statefulset web --type='json' -p='[{"op": "replace","path":"/spec/template/spec/containers/0/image","value":"dotbalo/canary:v1"}]'


upgrade from N-1  to 1 , one by one
[root@k8s-master01 ~]# kubectl get pod
NAME     READY   STATUS              RESTARTS   AGE
web-0    1/1     Running             0          11m
web-1    1/1     Running             0          11m
web-10   0/1     ContainerCreating   0          17s
web-11   1/1     Running             0          38s
web-2    1/1     Running             0          9m48s
web-3    1/1     Running             0          5m6s
web-4    1/1     Running             0          4m58s
web-5    1/1     Running             0          4m49s
web-6    1/1     Running             0          4m40s
web-7    1/1     Running             0          4m32s
web-8    1/1     Running             0          4m24s
web-9    1/1     Running             0          4m15s

[root@k8s-master01 ~]# kubectl get pod
NAME     READY   STATUS        RESTARTS   AGE
web-0    1/1     Running       0          11m
web-1    1/1     Running       0          11m
web-10   1/1     Running       0          18s
web-11   1/1     Running       0          39s
web-2    1/1     Running       0          9m49s
web-3    1/1     Running       0          5m7s
web-4    1/1     Running       0          4m59s
web-5    1/1     Running       0          4m50s
web-6    1/1     Running       0          4m41s
web-7    1/1     Running       0          4m33s
web-8    1/1     Running       0          4m25s
web-9    1/1     Terminating   0          4m16s

[root@k8s-master01 ~]# kubectl get pod
NAME     READY   STATUS        RESTARTS   AGE
web-0    1/1     Running       0          11m
web-1    1/1     Running       0          11m
web-10   1/1     Running       0          21s
web-11   1/1     Running       0          42s
web-2    1/1     Running       0          9m52s
web-3    1/1     Running       0          5m10s
web-4    1/1     Running       0          5m2s
web-5    1/1     Running       0          4m53s
web-6    1/1     Running       0          4m44s
web-7    1/1     Running       0          4m36s
web-8    1/1     Running       0          4m28s
web-9    0/1     Terminating   0          4m19s



[root@k8s-master01 ~]# kubectl rollout status sts/web
Waiting for 1 pods to be ready...
Waiting for partitioned roll out to finish: 3 out of 12 new pods have been updated...
Waiting for 1 pods to be ready...
Waiting for 1 pods to be ready...
Waiting for partitioned roll out to finish: 4 out of 12 new pods have been updated...
Waiting for 1 pods to be ready...
Waiting for 1 pods to be ready...
Waiting for partitioned roll out to finish: 5 out of 12 new pods have been updated...
Waiting for 1 pods to be ready...
Waiting for 1 pods to be ready...

```

## partitioned upgrade

```bash
kubectl patch statefulset web -p '{"spec":{"updateStrategy":{"type":"RollingUpdate":{"partition":3}}}}'

[root@k8s-master01 ~]# for p in {0..11}; do kubectl get po web-$p --template '{{range $i,$c := .spec.containers}}{{$c.image}}{{end}}' ; echo; done
dotbalo/canary:v1
dotbalo/canary:v1
dotbalo/canary:v1
dotbalo/canary:v1
dotbalo/canary:v1
dotbalo/canary:v1
dotbalo/canary:v1
dotbalo/canary:v1
dotbalo/canary:v1
dotbalo/canary:v1
dotbalo/canary:v1
dotbalo/canary:v1

kubectl patch statefulset web --type='json' -p='[{"op": "replace","path":"/spec/template/spec/containers/0/image","value":"k8s.gcr.io/nginx-slim:0.7"}]'

[root@k8s-master01 ~]# for p in {0..11}; do kubectl get po web-$p --template '{{range $i,$c := .spec.containers}}{{$c.image}}{{end}}' ; echo; done
dotbalo/canary:v1
dotbalo/canary:v1
dotbalo/canary:v1
dotbalo/canary:v1
dotbalo/canary:v1
dotbalo/canary:v1
dotbalo/canary:v1
dotbalo/canary:v1
dotbalo/canary:v1
k8s.gcr.io/nginx-slim:0.7
k8s.gcr.io/nginx-slim:0.7
k8s.gcr.io/nginx-slim:0.7
```

```bash
kubectl delete statefulset web --cascade=false 
// use cascade=false will not trigger pod delete, then pod became standalone , you could delete them manually 

kubectl delete statefulset web 

```
