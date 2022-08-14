     resourcequota.yaml

````yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: resource-test
  labels:
    app: resourcequota
spec:
  hard:
    pods: 50
    requests.cpu: 0.5
    requests.memory: 512Mi
    limits.cpu: 5
    limits.memory: 16Gi
    configmaps: 20
    requests.storage: 40Gi
    persistentvolumeclaims: 20
    replicationcontrollers: 20
    secrets: 20
    services: 50
    services.loadbalancers: "2"
    services.nodeports: "10"

````

```bash

[root@k8s-master01 ~]# kubectl create ns quota-example
namespace/quota-example created
[root@k8s-master01 ~]# kubectl create -f rq.yaml -n quota-example
kubectlresourcequota/resource-test created
[root@k8s-master01 ~]# kubectl get quota -n quota-example
NAME            AGE   REQUEST                                                                                                                                                                                                                                                    LIMIT
resource-test   25s   configmaps: 1/20, persistentvolumeclaims: 0/20, pods: 0/50, replicationcontrollers: 0/20, requests.cpu: 0/500m, requests.memory: 0/512Mi, requests.storage: 0/40Gi, secrets: 1/20, services: 0/50, services.loadbalancers: 0/2, services.nodeports: 0/10   limits.cpu: 0/5, limits.memory: 0/16Gi
```


​      quota-objects.yaml

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-quota-demo
spec:
  hard:
    persistentvolumeclaims: "1"

```

```bash

[root@k8s-master01 ~]# kubectl create -f rqo.yaml -n quota-example
resourcequota/object-quota-demo created
[root@k8s-master01 ~]# kubectl get quota -n quota-example -oyaml

```
```yaml
apiVersion: v1
items:
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    creationTimestamp: "2022-08-14T05:59:18Z"
    name: object-quota-demo
    namespace: quota-example
    resourceVersion: "1857880"
    uid: abcf9a86-c20a-46f4-bd0a-0288e361fa69
  spec:
    hard:
      persistentvolumeclaims: "1"
  status:
    hard:
      persistentvolumeclaims: "1"
    used:
      persistentvolumeclaims: "0"
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    creationTimestamp: "2022-08-14T05:57:43Z"
    labels:
      app: resourcequota
    name: resource-test
    namespace: quota-example
    resourceVersion: "1857476"
    uid: ab734b3e-d65c-43b0-982d-576820016e65
  spec:
    hard:
      configmaps: "20"
      limits.cpu: "5"
      limits.memory: 16Gi
      persistentvolumeclaims: "20"
      pods: "50"
      replicationcontrollers: "20"
      requests.cpu: 500m
      requests.memory: 512Mi
      requests.storage: 40Gi
      secrets: "20"
      services: "50"
      services.loadbalancers: "2"
      services.nodeports: "10"
  status:
    hard:
      configmaps: "20"
      limits.cpu: "5"
      limits.memory: 16Gi
      persistentvolumeclaims: "20"
      pods: "50"
      replicationcontrollers: "20"
      requests.cpu: 500m
      requests.memory: 512Mi
      requests.storage: 40Gi
      secrets: "20"
      services: "50"
      services.loadbalancers: "2"
      services.nodeports: "10"
    used:
      configmaps: "1"
      limits.cpu: "0"
      limits.memory: "0"
      persistentvolumeclaims: "0"
      pods: "0"
      replicationcontrollers: "0"
      requests.cpu: "0"
      requests.memory: "0"
      requests.storage: "0"
      secrets: "1"
      services: "0"
      services.loadbalancers: "0"
      services.nodeports: "0"
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""

```

​         **pvc.yaml**   

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-quota-demo
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi

```

```bash

[root@k8s-master01 ~]# kubectl get quota  object-quota-demo -n quota-example -oyaml
apiVersion: v1
kind: ResourceQuota
metadata:
  creationTimestamp: "2022-08-14T05:59:18Z"
  name: object-quota-demo
  namespace: quota-example
  resourceVersion: "1858301"
  uid: abcf9a86-c20a-46f4-bd0a-0288e361fa69
spec:
  hard:
    persistentvolumeclaims: "1"
status:
  hard:
    persistentvolumeclaims: "1"
  used:
    persistentvolumeclaims: "1"
```



​         **pvc2.yaml**   Create failure

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-quota-demo2
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi

```





