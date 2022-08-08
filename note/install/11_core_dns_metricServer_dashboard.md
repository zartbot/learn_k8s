## CORE-DNS

```bash
cd /root/k8s-ha-install/CoreDNS
COREDNS_SERVICE_IP=`kubectl get svc | grep kubernetes | awk '{print $3}' `0
sed -i "s#192.168.0.10#${COREDNS_SERVICE_IP}#g" coredns.yaml

kubectl create -f coredns.yaml
```

## Metric Server

```bash
cd /root/k8s-ha-install/metrics-server
kubectl create -f .


kubectl top node

[root@k8s-master01 metrics-server]# kubectl top node
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
k8s-master01   404m         5%     2500Mi          15%
k8s-master02   180m         2%     1840Mi          11%
k8s-master03   193m         2%     1872Mi          11%
k8s-node01     122m         1%     1091Mi          6%
k8s-node02     137m         1%     1166Mi          7%

```

## Dashboard
```bash
cd /root/k8s-ha-install/dashboard
kubectl create -f .

[root@k8s-master01 dashboard]# kubectl get svc kubernetes-dashboard -n kubernetes-dashboard
NAME                   TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
kubernetes-dashboard   NodePort   10.99.215.119   <none>        443:30251/TCP   7m50s


use any kube-proxy IP access
https://192.168.99.71:30251/
```

Check Token Access
```bash
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')


Name:         admin-user-token-9m4st
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: 45ad6218-7e41-471e-a162-e0c0f12b8cbf

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1411 bytes
namespace:  11 bytes
token:      

```


