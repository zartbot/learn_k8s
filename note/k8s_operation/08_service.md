pod has its own lifecycle, however some service require fixed IP+ports. You may create service as an abstract layer to handle inter-pods communication.

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```
表示my-service：80可以访问app=myapp的9376端口
带有选择器的Service创建后会在所在的namespace创建一个同名的Endpoint


以Metrics-server为例，可以看到同名的ep和pod ip地址相同

```bash
[root@k8s-master01 ~]# kubectl get svc -n kube-system metrics-server -oyaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2022-08-08T06:50:01Z"
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
  resourceVersion: "92703"
  uid: b94a23e4-3376-46b2-9dd0-0591928ab567
spec:
  clusterIP: 10.99.63.143
  clusterIPs:
  - 10.99.63.143
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - name: https
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    k8s-app: metrics-server
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}

[root@k8s-master01 ~]# kubectl get pod -n kube-system -l k8s-app=metrics-server -owide
NAME                              READY   STATUS    RESTARTS   AGE   IP               NODE           NOMINATED NODE   READINESS GATES
metrics-server-5c8b499fd7-2n7wz   1/1     Running   0          29h   172.25.244.193   k8s-master01   <none>           <none>

[root@k8s-master01 ~]# kubectl get ep -n kube-system metrics-server
NAME             ENDPOINTS             AGE
metrics-server   172.25.244.193:4443   29h
```

生产环境中可能需要以某个固定的名称访问， 或者希望service 指向另一个namespace或其它集群的业务，或者工作负载转移到K8S过程中有一部分服务还在外部, require 无Selector Service

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
---
kind: Endpoints
apiVersion: v1
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 1.2.3.4
    ports:
      - port: 9376
```

但是必须手工创建同名ep,如上，当然也可用ExternalName来支持

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```

## Service代理模式

每个节点都有一个kube-proxy可以负责为Service实现一种虚拟IP的形式， K8S 1.0全userspace，1.1增加iptables 1.2默认iptables  1.8开始增加IPVS


## 创建多个端口Service

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: myapp
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 9376
  - name: https
    protocol: TCP
    port: 443
    targetPort: 9377
```

## Service类型

* ClusterIP：用于集群内部访问
* NodePort：在安装Kube-Proxy的节点上打开一个端口代理到后端Pod， --service-node-port-range 默认30000-32767
* LoadBalancer： 使用云服务提供商的负载均衡器
* ExternalName：通过返回定义的CNAME别名，将Service映射到可被DNS解析的其它域名

```bash
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30000
  selector:
    k8s-app: kubernetes-dashboard
```

## Service 发现

1. 基于环境变量
```bash

[root@k8s-master01 ~]# kubectl get svc -n kube-system
NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                  AGE
calico-typha     ClusterIP   10.99.219.47   <none>        5473/TCP                 30h
kube-dns         ClusterIP   10.99.0.10     <none>        53/UDP,53/TCP,9153/TCP   30h
metrics-server   ClusterIP   10.99.63.143   <none>        443/TCP                  30h

[root@k8s-master01 ~]# kubectl get po -n kube-system -l k8s-app=metrics-server
NAME                              READY   STATUS    RESTARTS   AGE
metrics-server-5c8b499fd7-2n7wz   1/1     Running   0          30h


[root@k8s-master01 ~]# kubectl exec -it metrics-server-5c8b499fd7-2n7wz -n kube-system -- env | grep KUBE_DNS
KUBE_DNS_PORT_53_UDP_PROTO=udp
KUBE_DNS_SERVICE_HOST=10.99.0.10
KUBE_DNS_PORT_53_TCP_PROTO=tcp
KUBE_DNS_PORT_53_UDP=udp://10.99.0.10:53
KUBE_DNS_PORT_53_UDP_PORT=53
KUBE_DNS_PORT_53_TCP=tcp://10.99.0.10:53
KUBE_DNS_PORT_9153_TCP_PORT=9153
KUBE_DNS_SERVICE_PORT_METRICS=9153
KUBE_DNS_PORT_53_TCP_PORT=53
KUBE_DNS_PORT_53_UDP_ADDR=10.99.0.10
KUBE_DNS_PORT_9153_TCP_ADDR=10.99.0.10
KUBE_DNS_SERVICE_PORT_DNS=53
KUBE_DNS_SERVICE_PORT=53
KUBE_DNS_PORT_53_TCP_ADDR=10.99.0.10
KUBE_DNS_PORT_9153_TCP=tcp://10.99.0.10:9153
KUBE_DNS_PORT_9153_TCP_PROTO=tcp
KUBE_DNS_SERVICE_PORT_DNS_TCP=53
KUBE_DNS_PORT=udp://10.99.0.10:53

```


2. 基于DNS

```bash

[root@k8s-master01 ~]# kubectl run redis --image=redis:3.2.10-alpine
pod/redis created

[root@k8s-master01 ~]# k get po
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-95b8b9d79-2q5cn   1/1     Running   0          72m
nginx-deployment-95b8b9d79-grfl4   1/1     Running   0          72m
nginx-deployment-95b8b9d79-k6mfk   1/1     Running   0          72m
nginx-deployment-95b8b9d79-ncw8h   1/1     Running   0          72m
nginx-deployment-95b8b9d79-vfzn2   1/1     Running   0          72m
redis                              1/1     Running   0          27s
[root@k8s-master01 ~]# kubectl exec -it redis -- nslookup kube-dns.kube-system
nslookup: can't resolve '(null)': Name does not resolve
Name:      kube-dns.kube-system
Address 1: 10.99.0.10 kube-dns.kube-system.svc.cluster.local
```

