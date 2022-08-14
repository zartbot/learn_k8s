Some micro service application may require external configuration file. If we copy them during docker image build, it may lose flexibility. However, the apps will be bring up in any Node, if we have a centralized config pool, it could be easy for app mount.


## Create ConfigMap

```bash
kubectl create configmap <map-name> <data-src>

* data-src: coulde be directory , file  or string

configmap backend storage is a k-v based map.
```


## directory mapping

```bash
[root@k8s-master01 ~]# cat conf/game.porperties
A=b
b=c
secret.code.passphras=FJSLAjasdlkjsa

[root@k8s-master01 ~]# cat conf/ui.properties
 hell = world

[root@k8s-master01 ~]# kubectl create configmap temp-game-config --from-file=conf
configmap/temp-game-config created

[root@k8s-master01 ~]# kubectl get cm temp-game-config -oyaml
apiVersion: v1
data:
  game.porperties: |+
    A=b
    b=c
    secret.code.passphras=FJSLAjasdlkjsa

  ui.properties: |2+
     hell = world

kind: ConfigMap
metadata:
  creationTimestamp: "2022-08-09T13:16:04Z"
  name: temp-game-config
  namespace: default
  resourceVersion: "342655"
  uid: 690092a8-121c-4dd2-a63f-44aa0e7953d9

```

## file based configuration

```bash

[root@k8s-master01 ~]# kubectl create configmap temp-game-config2 --from-file=conf/game.porperties
configmap/temp-game-config2 created
[root@k8s-master01 ~]# kubectl get cm temp-game-config2 -oyaml
apiVersion: v1
data:
  game.porperties: |+
    A=b
    b=c
    secret.code.passphras=FJSLAjasdlkjsa

kind: ConfigMap
metadata:
  creationTimestamp: "2022-08-09T13:18:05Z"
  name: temp-game-config2
  namespace: default
  resourceVersion: "342928"
  uid: 3a38374b-e013-4c9d-a1d5-8e8a69b789f5
```

## key based
```bash
[root@k8s-master01 ~]# kubectl create configmap temp-game-config3 --from-file=self-key=conf/game.porperties
configmap/temp-game-config3 created
[root@k8s-master01 ~]# kubectl get cm temp-game-config3 -oyaml
apiVersion: v1
data:
  self-key: |+
    A=b
    b=c
    secret.code.passphras=FJSLAjasdlkjsa

kind: ConfigMap
metadata:
  creationTimestamp: "2022-08-09T13:18:49Z"
  name: temp-game-config3
  namespace: default
  resourceVersion: "343028"
  uid: f9dac9e3-8223-4a65-befa-0083197bfac8
```

## ENV configmap

some application may use env as config. we could use the following cmmand

```bash

[root@k8s-master01 ~]# kubectl create configmap temp-game-env-config --from-env-file=conf/game.porperties
configmap/temp-game-env-config created

[root@k8s-master01 ~]# kubectl get cm temp-game-env-config -oyaml
apiVersion: v1
data:
  A: b
  b: c
  secret.code.passphras: FJSLAjasdlkjsa
kind: ConfigMap
metadata:
  creationTimestamp: "2022-08-09T13:21:20Z"
  name: temp-game-env-config
  namespace: default
  resourceVersion: "343369"
  uid: 487b70dc-c371-43fb-bcb3-b1b78a46bfc7
```

## from literal

```bash

[root@k8s-master01 ~]# kubectl create configmap temp-game-txt-config --from-literal=special.how=very --from-literal=special.type=cha
configmap/temp-game-txt-config created
[root@k8s-master01 ~]# kubectl get cm temp-game-txt-config -oyaml
apiVersion: v1
data:
  special.how: very
  special.type: cha
kind: ConfigMap
metadata:
  creationTimestamp: "2022-08-09T13:22:41Z"
  name: temp-game-txt-config
  namespace: default
  resourceVersion: "343552"
  uid: 78266683-246d-401b-9073-15e793fc254f
```


## Delete Config
```bash
kubectl delete -n default configmap temp-game-config3


[root@k8s-master01 ~]# kubectl get cm
NAME                DATA   AGE
kube-root-ca.crt    1      2d1h
temp-game-config    2      21m
temp-game-config2   1      19m

[root@k8s-master01 ~]# kubectl delete -n default configmap temp-game-config
configmap "temp-game-config" deleted

[root@k8s-master01 ~]# kubectl delete -n default configmap temp-game-config2
configmap "temp-game-config2" deleted

```

## use configmap


```bash
kubectl create configmap special-config --from-literal=special.how=very --from-literal=special.level=info
```

**valueFrom**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: env-valuefrom
  name: env-valuefrom
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: env-valuefrom
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: env-valuefrom
    spec:
      containers:
      - command:
        - sh
        - -c
        - env
        env:
        - name: TZ
          value: Asia/Shanghai
        - name: LANG
          value: C.UTF-8
        - name: SPECIAL_LEVEL_KEY 
          valueFrom:
            configMapKeyRef:
              key: special.how
              name: special-config
        image: busybox
        imagePullPolicy: IfNotPresent
        name: env-valuefrom
        resources:
          limits:
            cpu: 100m
            memory: 100Mi
          requests:
            cpu: 10m
            memory: 10Mi
      dnsPolicy: ClusterFirst
      restartPolicy: Always

```

```bash

[root@k8s-master01 ~]# k logs env-valuefrom-84659f974b-ck5k5
...
SPECIAL_LEVEL_KEY=very
...

TZ=Asia/Shanghai
```

**envFrom**

````yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: env-valuefrom
  name: env-valuefrom
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: env-valuefrom
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: env-valuefrom
    spec:
      containers:
      - command:
        - sh
        - -c
        - env
        env:
        - name: TZ
          value: Asia/Shanghai
        - name: LANG
          value: C.UTF-8
        envFrom:
        - configMapRef:
            name: game-config-env-file
          prefix: fromCm_
        image: busybox
        imagePullPolicy: IfNotPresent
        name: env-valuefrom
        resources:
          limits:
            cpu: 100m
            memory: 100Mi
          requests:
            cpu: 10m
            memory: 10Mi
      dnsPolicy: ClusterFirst
      restartPolicy: Never

````

**文件挂载**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: busybox
      command: [ "/bin/sh", "-c", "ls /etc/config/" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        # Provide the name of the ConfigMap containing the files you want
        # to add to the container
        name: special-config
  restartPolicy: Never

```

**自定义文件名**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: busybox
      command: [ "/bin/sh","-c","cat /etc/config/keys" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: special-config
        items:
        - key: special.how
          path: keys
  restartPolicy: Never

```

**指定文件权限**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: busybox
      command: [ "/bin/sh","-c","ls -l /etc/config/..data/" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: special-config
        items:
        - key: special.how
          path: keys
        defaultMode: 0666
  restartPolicy: Never

```

## Secret

some app password 

```bash

echo -n "admin" > ./username.txt
echo -n "1a2b3c4deadbeef" > ./password.txt

kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt

[root@k8s-master01 ~]# kubectl get secrets db-user-pass
NAME           TYPE     DATA   AGE
db-user-pass   Opaque   2      10s


[root@k8s-master01 ~]# kubectl describe secrets/db-user-pass
Name:         db-user-pass
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password.txt:  15 bytes
username.txt:  5 bytes
```

secret from yaml


```bash

[root@k8s-master01 ~]# echo -n "admin" | base64
YWRtaW4=

[root@k8s-master01 ~]# echo -n "1a2b3c4deadbeef" | base64
MWEyYjNjNGRlYWRiZWVm

```
```yaml
apiVersion: v1
kind:Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWEyYjNjNGRlYWRiZWVm
```  

```bash
kubectl create -f db-u.yaml

[root@k8s-master01 ~]# kubectl get secret mysecret -o yaml
apiVersion: v1
data:
  password: MWEyYjNjNGRlYWRiZWVm
  username: YWRtaW4=
kind: Secret
metadata:
  creationTimestamp: "2022-08-10T07:42:23Z"
  name: mysecret
  namespace: default
  resourceVersion: "492394"
  uid: 271801d9-842a-4123-936b-936ee00e1bf7
type: Opaque


[root@k8s-master01 ~]# echo "MWEyYjNjNGRlYWRiZWVm" | base64 --decode
1a2b3c4deadbeef

```

