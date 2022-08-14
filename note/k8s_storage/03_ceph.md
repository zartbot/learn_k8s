
## Rook

https://rook.io/

Rook 是一个开源的云原生存储编排工具，提供平台、框架和对各种存储解决方案的支持，以和云原生环境进行本地集成。

Rook 将存储软件转变成自我管理、自我扩展和自我修复的存储服务，通过自动化部署、启动、配置、供应、扩展、升级、迁移、灾难恢复、监控和资源管理来实现。Rook 底层使用云原生容器管理、调度和编排平台提供的能力来提供这些功能，其实就是我们平常说的 Operator。Rook 利用扩展功能将其深度集成到云原生环境中，并为调度、生命周期管理、资源管理、安全性、监控等提供了无缝的体验。有关 Rook 当前支持的存储解决方案的状态的更多详细信息，可以参考 Rook 仓库 的项目介绍。

Rook 包含多个组件：

* Rook Operator：Rook 的核心组件，Rook Operator 是一个简单的容器，自动启动存储集群，并监控存储守护进程，来确保存储集群的健康。
* Rook Agent：在每个存储节点上运行，并配置一个 FlexVolume 或者 CSI 插件，和 Kubernetes 的存储卷控制框架进行集成。Agent 处理所有的存储操作，例如挂接网络存储设备、在主机上加载存储卷以及格式化文件系统等。
* Rook Discovers：检测挂接到存储节点上的存储设备。

Rook 还会用 Kubernetes Pod 的形式，部署 Ceph 的 MON、OSD 以及 MGR 守护进程。Rook Operator 让用户可以通过 CRD 来创建和管理存储集群。每种资源都定义了自己的 CRD：

* RookCluster：提供了对存储机群的配置能力，用来提供块存储、对象存储以及共享文件系统。每个集群都有多个 Pool。
* Pool：为块存储提供支持，Pool 也是给文件和对象存储提供内部支持。
* Object Store：用 S3 兼容接口开放存储服务。
* File System：为多个 Kubernetes Pod 提供共享存储。

## 环境
Rook Ceph 需要使用 RBD 内核模块，我们可以通过运行 

```bash
modprobe rbd 
```
来测试 Kubernetes 节点是否有该模块，如果没有，则需要更新下内核版本。

另外需要在节点上安装 lvm2 软件包：

```bash
# Centos
sudo yum install -y lvm2

# Ubuntu
sudo apt-get install -y lvm2
```

## 安装
我们这里部署最新的 release-1.2 版本的 Rook，部署清单文件地址：

https://github.com/rook/rook/tree/release-1.2/cluster/examples/kubernetes/ceph。

从上面链接中下载 common.yaml 与 operator.yaml 两个资源清单文件：


```bash
[root@k8s-master01 ~]# git clone --single-branch --branch v1.7.9 https://github.com/rook/rook.git

[root@k8s-master01 ~]# cd rook/cluster/examples/kubernetes/ceph


kubectl apply -f crds.yaml -f ./common.yaml -f ./operator.yaml

Wait for Running

[root@k8s-master01 ceph]# kubectl -n rook-ceph get pod
NAME                                  READY   STATUS    RESTARTS   AGE
rook-ceph-operator-54655cf4cd-shtjg   1/1     Running   0          3m37s

kubectl apply -f ./cluster.yaml

[root@k8s-master01 ceph]# kubectl -n rook-ceph get pod
NAME                                                     READY   STATUS      RESTARTS        AGE
csi-cephfsplugin-2vbrv                                   3/3     Running     0               14m
csi-cephfsplugin-6qwnb                                   3/3     Running     0               14m
csi-cephfsplugin-npg5c                                   3/3     Running     0               14m
csi-cephfsplugin-provisioner-689686b44-dc8z4             6/6     Running     8 (107s ago)    14m
csi-cephfsplugin-provisioner-689686b44-dhm2z             6/6     Running     16 (108s ago)   14m
csi-cephfsplugin-s8jmx                                   3/3     Running     0               14m
csi-cephfsplugin-sq4bx                                   3/3     Running     0               14m
csi-rbdplugin-h9bwl                                      3/3     Running     0               16m
csi-rbdplugin-hdcnw                                      3/3     Running     0               16m
csi-rbdplugin-jgprd                                      3/3     Running     0               16m
csi-rbdplugin-kfrnk                                      3/3     Running     0               16m
csi-rbdplugin-provisioner-5775fb866b-mq4ff               6/6     Running     1 (100s ago)    16m
csi-rbdplugin-provisioner-5775fb866b-xqm5l               6/6     Running     10 (113s ago)   16m
csi-rbdplugin-qfr4j                                      3/3     Running     0               16m
rook-ceph-crashcollector-k8s-master02-79f97896c4-g6prp   1/1     Running     0               5m13s
rook-ceph-crashcollector-k8s-master03-578fc5d89-65qvg    1/1     Running     0               5m28s
rook-ceph-crashcollector-k8s-node02-6d96b8b8-q97nj       1/1     Running     0               5m22s
rook-ceph-mgr-a-c776d8f67-nxxsd                          1/1     Running     0               5m29s
rook-ceph-mon-a-649dc8b4-xn25s                           1/1     Running     0               13m
rook-ceph-mon-b-7c4bdb8897-ml7c2                         1/1     Running     0               8m22s
rook-ceph-mon-c-69bccf66df-csxk6                         1/1     Running     0               7m11s
rook-ceph-operator-54655cf4cd-shtjg                      1/1     Running     0               23m
rook-ceph-osd-prepare-k8s-master01--1-ts75t              0/1     Completed   0               47s
rook-ceph-osd-prepare-k8s-master02--1-p5b6m              0/1     Completed   0               42s
rook-ceph-osd-prepare-k8s-master03--1-xp7sw              0/1     Completed   0               34s
rook-ceph-osd-prepare-k8s-node01--1-fs5wd                0/1     Completed   0               28s
rook-ceph-osd-prepare-k8s-node02--1-d4kc8                0/1     Completed   0               20s
```


## Verfiy
```bash
kubectl apply -f ./toolbox.yaml


[root@k8s-master01 ceph]# kubectl apply -f ./toolbox.yaml
deployment.apps/rook-ceph-tools created
You have new mail in /var/spool/mail/root
[root@k8s-master01 ceph]# kubectl exec -it $(kubectl -n rook-ceph get pod -l "app=rook-ceph-tools" -o jsonpath='{.items[0].metadata.name}') -n rook-ceph -- bash

[root@rook-ceph-tools-54474cfc96-vv4gj /]# ceph status
  cluster:
    id:     2565522f-d37b-45ed-a3de-062a562671de
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum a,b,c (age 25h)
    mgr: a(active, since 25h)
    osd: 5 osds: 5 up (since 25h), 5 in (since 41h)

  data:
    pools:   1 pools, 128 pgs
    objects: 0 objects, 0 B
    usage:   49 MiB used, 700 GiB / 700 GiB avail
    pgs:     128 active+clean

[root@rook-ceph-tools-54474cfc96-vv4gj /]# ceph osd status
ID  HOST           USED  AVAIL  WR OPS  WR DATA  RD OPS  RD DATA  STATE
 0  k8s-master01  7704k  99.9G      0        0       0        0   exists,up
 1  k8s-master03  7128k  99.9G      0        0       0        0   exists,up
 2  k8s-node01    12.9M   199G      0        0       0        0   exists,up
 3  k8s-node02    13.5M   199G      0        0       0        0   exists,up
 4  k8s-master02  7840k  99.9G      0        0       0        0   exists,up




```

* (Optional)Ceph handson

```bash
[root@rook-ceph-tools-54474cfc96-vv4gj /]# ceph fs ls
No filesystems enabled
rook-ceph-tools-54474cfc96-vv4gj /]# ceph osd pool create sharefs-data0 128 128
rook-ceph-tools-54474cfc96-vv4gj /]# ceph osd pool create sharefs-metadata 64 64


ceph fs new sharefs sharefs-metadata sharefs-data0

[root@rook-ceph-tools-54474cfc96-vv4gj /]# ceph fs ls
name: sharefs, metadata pool: sharefs-metadata, data pools: [sharefs-data0 ]

[root@rook-ceph-tools-54474cfc96-vv4gj /]# ceph auth get-key client.admin
AQD8nfNi+XruBhAAArJ+1NfugRicD4BSZjbVag==

[root@rook-ceph-tools-54474cfc96-vv4gj /]# ceph mon dump
epoch 3
fsid 2565522f-d37b-45ed-a3de-062a562671de
last_changed 2022-08-10T12:09:44.263382+0000
created 2022-08-10T12:06:53.033047+0000
min_mon_release 16 (pacific)
election_strategy: 1
0: [v2:10.99.189.111:3300/0,v1:10.99.189.111:6789/0] mon.a
1: [v2:10.99.193.78:3300/0,v1:10.99.193.78:6789/0] mon.b
2: [v2:10.99.98.59:3300/0,v1:10.99.98.59:6789/0] mon.c
dumped monmap epoch 3

[root@rook-ceph-tools-54474cfc96-vv4gj /]# ceph fsid
2565522f-d37b-45ed-a3de-062a562671de

```



Rook支持3种模式：

Block: Create block storage to be consumed by a pod (RWO)，块设备，就是PV / PVC
Shared Filesystem: Create a filesystem to be shared across multiple pods (RWX)，类似NFS
Object: Create an object store that is accessible inside or outside the Kubernetes cluster，对象存储，类似S3、OSS


## StorageClass

虽然PV和PVC屏蔽了一些存储上的细节，降低了使用复杂度，但是当PV过多后，难以对生命周期进行管理，所以实现了StorageClass 动态管理集群中的PV


```bash
kubectl apply -f ./csi/rbd/storageclass.yaml

[root@k8s-master01 ceph]# kubectl get sc -o wide
NAME              PROVISIONER                  RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
rook-ceph-block   rook-ceph.rbd.csi.ceph.com   Delete          Immediate           true                   9s
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
spec:
  storageClassName: rook-ceph-block
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
    tier: mysql
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
        - image: mysql:5.6
          name: mysql
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: cisco123
          ports:
            - containerPort: 3306
              name: mysql
          volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /var/lib/mysql
      volumes:
        - name: mysql-persistent-storage
          persistentVolumeClaim:
            claimName: mysql-pv-claim
```

```bash
[root@k8s-master01 kubernetes]# kubectl apply -f mysql.yaml
service/wordpress-mysql created
persistentvolumeclaim/mysql-pv-claim created
deployment.apps/wordpress-mysql created

[root@k8s-master01 kubernetes]# kubectl get pvc
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
mysql-pv-claim   Bound    pvc-d0c2ed36-43d5-4812-ae59-02f03566e44e   20Gi       RWO            rook-ceph-block   13s

[root@k8s-master01 kubernetes]# kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                    STORAGECLASS      REASON   AGE
pv0003                                     5Gi        RWO            Recycle          Available                            nfs-slow                   42h
pvc-d0c2ed36-43d5-4812-ae59-02f03566e44e   20Gi       RWO            Delete           Bound       default/mysql-pv-claim   rook-ceph-block            13s
[root@k8s-master01 kubernetes]#

```


[root@k8s-master01 ceph]# cat dashboard-external-https.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: rook-ceph-mgr-dashboard-external-https
  namespace: rook-ceph # namespace:cluster
  labels:
    app: rook-ceph-mgr
    rook_cluster: rook-ceph # namespace:cluster
spec:
  ports:
    - name: dashboard
      port: 8443
      protocol: TCP
      targetPort: 8443
  selector:
    app: rook-ceph-mgr
    rook_cluster: rook-ceph
  sessionAffinity: None
  type: NodePort

```  

```bash
[root@k8s-master01 ceph]# kubectl apply -f dashboard-external-https.yaml
service/rook-ceph-mgr-dashboard-external-https created
[root@k8s-master01 ceph]#
[root@k8s-master01 ceph]#
[root@k8s-master01 ceph]#
[root@k8s-master01 ceph]# kubectl get svc
NAME              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
kubernetes        ClusterIP   10.99.0.1      <none>        443/TCP    4d18h
my-service        ClusterIP   10.99.97.118   <none>        80/TCP     2d18h
wordpress-mysql   ClusterIP   None           <none>        3306/TCP   4m3s
[root@k8s-master01 ceph]# kubectl get svc -n rook-ceph
NAME                                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
csi-cephfsplugin-metrics                 ClusterIP   10.99.130.44    <none>        8080/TCP,8081/TCP   42h
csi-rbdplugin-metrics                    ClusterIP   10.99.42.13     <none>        8080/TCP,8081/TCP   42h
rook-ceph-mgr                            ClusterIP   10.99.221.122   <none>        9283/TCP            42h
rook-ceph-mgr-dashboard                  ClusterIP   10.99.45.221    <none>        8443/TCP            42h
rook-ceph-mgr-dashboard-external-https   NodePort    10.99.239.129   <none>        8443:31686/TCP      17s
rook-ceph-mon-a                          ClusterIP   10.99.189.111   <none>        6789/TCP,3300/TCP   42h
rook-ceph-mon-b                          ClusterIP   10.99.193.78    <none>        6789/TCP,3300/TCP   42h
rook-ceph-mon-c                          ClusterIP   10.99.98.59     <none>        6789/TCP,3300/TCP   42h
```

web login username is admin , password could be found in 
```bash

[root@k8s-master01 ceph]# kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath="{['data']['password']}" | base64 --decode && echo
*l#^}m!$+2^Qp@4W+=q?
```

## Object store
```bash
[root@k8s-master01 ceph]# kubectl apply -f object.yaml

[root@k8s-master01 ceph]# kubectl apply -f object-external.yaml
cephobjectstore.ceph.rook.io/external-store created

[root@k8s-master01 ceph]# kubectl create -f storageclass-bucket-delete.yaml
storageclass.storage.k8s.io/rook-ceph-delete-bucket created

[root@k8s-master01 ceph]# kubectl create -f rgw-external.yaml
service/rook-ceph-rgw-my-store-external created
[root@k8s-master01 ceph]#  kubectl -n rook-ceph get service rook-ceph-rgw-my-store rook-ceph-rgw-my-store-external
NAME                              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
rook-ceph-rgw-my-store            ClusterIP   10.99.150.99   <none>        80/TCP         6m
rook-ceph-rgw-my-store-external   NodePort    10.99.67.133   <none>        80:30311/TCP   10s


[root@k8s-master01 ceph]# curl 192.168.99.71:30311
<?xml version="1.0" encoding="UTF-8"?><ListAllMyBucketsResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/"><Owner><ID>anonymous</ID><DisplayName></DisplayName></Owner><Buckets></Buckets></ListAllMyBucketsResult>

[root@k8s-master01 ceph]#
```
## Install Ceph CSI by Helm

```bash
kubectl create namespace ceph-csi-cephfs
helm repo add ceph-csi https://ceph.github.io/csi-charts

helm install --namespace "ceph-csi-cephfs" "ceph-csi-cephfs" ceph-csi/ceph-csi-cephfs
[root@k8s-master01 ceph]# helm install --namespace "ceph-csi-cephfs" "ceph-csi-cephfs" ceph-csi/ceph-csi-cephfs
NAME: ceph-csi-cephfs
LAST DEPLOYED: Fri Aug 12 17:23:29 2022
NAMESPACE: ceph-csi-cephfs
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Examples on how to configure a storage class and start using the driver are here:
https://github.com/ceph/ceph-csi/tree/v3.6.2/examples/cephfs

[root@k8s-master01 ceph]# k get po -n ceph-csi-cephfs
NAME                                           READY   STATUS              RESTARTS   AGE
ceph-csi-cephfs-nodeplugin-5g4hj               0/3     ContainerCreating   0          40s
ceph-csi-cephfs-nodeplugin-6277w               0/3     ContainerCreating   0          40s
ceph-csi-cephfs-nodeplugin-9rp9g               0/3     ContainerCreating   0          40s
ceph-csi-cephfs-nodeplugin-nbbvz               0/3     ContainerCreating   0          40s
ceph-csi-cephfs-nodeplugin-rj4m7               0/3     ContainerCreating   0          41s
ceph-csi-cephfs-provisioner-75c87754f4-5whjf   0/6     ContainerCreating   0          41s
ceph-csi-cephfs-provisioner-75c87754f4-dtpss   0/6     ContainerCreating   0          42s
ceph-csi-cephfs-provisioner-75c87754f4-ntcb6   0/6     ContainerCreating   0          41s
```
