用于Pod内容器之间共享数据

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
  namespace: default
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
      - image: nginx:1.15.2
        imagePullPolicy: IfNotPresent
        name: nginx
        volumeMounts:
        - mountPath: /opt
          name: share-volume
      - image: nginx:1.15.2
        imagePullPolicy: IfNotPresent
        name: nginx2
        command:
        - sh
        - -c
        - sleep 3600
        volumeMounts:
        - mountPath: /mnt
          name: share-volume
      volumes:
      - name: share-volume
        emptyDir: {}
          #medium: Memory
```

```bash
[root@k8s-master01 ~]# kubectl describe po
Name:         nginx-67948c7d6b-nxcvn
Namespace:    default
Priority:     0
Node:         k8s-master02/192.168.99.72
Start Time:   Wed, 10 Aug 2022 16:33:48 +0800
Labels:       app=nginx
              pod-template-hash=67948c7d6b
Annotations:  cni.projectcalico.org/podIP: 172.25.92.88/32
              cni.projectcalico.org/podIPs: 172.25.92.88/32
Status:       Running
IP:           172.25.92.88
IPs:
  IP:           172.25.92.88
Controlled By:  ReplicaSet/nginx-67948c7d6b
Containers:
  nginx:
    Container ID:   docker://569d9cc3a22479c9dbf2e6010bede557ed83cf61913c04d557636055e0c91d18
    Image:          nginx:1.15.2
    Image ID:       docker-pullable://nginx@sha256:d85914d547a6c92faa39ce7058bd7529baacab7e0cd4255442b04577c4d1f424
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 10 Aug 2022 16:34:15 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /opt from share-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-mwc6w (ro)
  nginx2:
    Container ID:  docker://9fe6a9561569bf4ee75fa7c4c545f07fe41e8a62cc150b0354727c6ad4bb5855
    Image:         nginx:1.15.2
    Image ID:      docker-pullable://nginx@sha256:d85914d547a6c92faa39ce7058bd7529baacab7e0cd4255442b04577c4d1f424
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      sleep 3600
    State:          Running
      Started:      Wed, 10 Aug 2022 16:34:16 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /mnt from share-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-mwc6w (ro)
```

```bash

[root@k8s-master01 ~]# kubectl exec --stdin --tty nginx-67948c7d6b-nxcvn -c nginx2 -- /bin/bash
root@nginx-67948c7d6b-nxcvn:/opt# cd /mnt
root@nginx-67948c7d6b-nxcvn:/mnt# ls
root@nginx-67948c7d6b-nxcvn:/mnt# touch bbb
exit


[root@k8s-master01 ~]# kubectl exec --stdin --tty nginx-67948c7d6b-nxcvn -c nginx -- /bin/bash
root@nginx-67948c7d6b-nxcvn:/# cd /opt
root@nginx-67948c7d6b-nxcvn:/opt# ls
bbb
 
```

## hostPath挂载宿主文件

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
  namespace: default
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
      - image: nginx:1.15.2
        imagePullPolicy: IfNotPresent
        name: nginx
        volumeMounts:
        - mountPath: /opt
          name: share-volume
        - mountPath: /etc/timezone
          name: timezone
      - image: nginx:1.15.2
        imagePullPolicy: IfNotPresent
        name: nginx2
        command:
        - sh
        - -c
        - sleep 3600
        volumeMounts:
        - mountPath: /mnt
          name: share-volume
      volumes:
      - name: share-volume
        emptyDir: {}
          #medium: Memory
      - name: timezone
        hostPath:
          path: /etc/timezone
          type: File
```

type coulde be mount as File/Socket/CharDevice/BlockDevice

