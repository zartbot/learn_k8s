```bash
[root@k8s-master01 ~]# git clone https://github.com/dotbalo/k8s.git
Cloning into 'k8s'...
remote: Enumerating objects: 1118, done.
remote: Counting objects: 100% (73/73), done.
remote: Compressing objects: 100% (25/25), done.
remote: Total 1118 (delta 57), reused 51 (delta 48), pack-reused 1045
Receiving objects: 100% (1118/1118), 5.27 MiB | 5.70 MiB/s, done.
Resolving deltas: 100% (497/497), done.
[root@k8s-master01 ~]# cd k8s/efk-7.10.2/


[root@k8s-master01 efk-7.10.2]# kubectl create -f create-logging-namespace.yaml
namespace/logging created

[root@k8s-master01 efk-7.10.2]# kubectl create -f es-service.yaml
service/elasticsearch-logging created

[root@k8s-master01 efk-7.10.2]# kubectl create -f es-statefulset.yaml
serviceaccount/elasticsearch-logging created
clusterrole.rbac.authorization.k8s.io/elasticsearch-logging created
clusterrolebinding.rbac.authorization.k8s.io/elasticsearch-logging created
statefulset.apps/elasticsearch-logging created

[root@k8s-master01 efk-7.10.2]# kubectl create -f kibana-deployment.yaml -f kibana-service.yaml
deployment.apps/kibana-logging created
service/kibana-logging created

[root@k8s-master01 efk-7.10.2]#
```

```bash
[root@k8s-master01 efk-7.10.2]# grep "nodeSelector" fluentd-es-ds.yaml -A 3
      nodeSelector:
        fluentd: "true"
      volumes:
      - name: varlog
[root@k8s-master01 efk-7.10.2]# kubectl label node k8s-node01 fluentd=true
node/k8s-node01 labeled
[root@k8s-master01 efk-7.10.2]# kubectl label node k8s-node02 fluentd=true
node/k8s-node02 labeled


[root@k8s-master01 efk-7.10.2]# kubectl get node -l fluentd=true --show-labels
NAME         STATUS   ROLES    AGE    VERSION   LABELS
k8s-node01   Ready    <none>   6d1h   v1.22.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,fluentd=true,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node01,kubernetes.io/os=linux,node.kubernetes.io/node=,region=subnet7
k8s-node02   Ready    <none>   6d1h   v1.22.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,fluentd=true,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node02,kubernetes.io/os=linux,node.kubernetes.io/node=,region=subnet7
[root@k8s-master01 efk-7.10.2]#

[root@k8s-master01 efk-7.10.2]# kubectl create -f fluentd-es-ds.yaml -f fluentd-es-configmap.yaml
serviceaccount/fluentd-es created
clusterrole.rbac.authorization.k8s.io/fluentd-es created
clusterrolebinding.rbac.authorization.k8s.io/fluentd-es created
daemonset.apps/fluentd-es-v3.1.1 created
configmap/fluentd-es-config-v0.2.1 created


```

Get Kibana ports
```bash

[root@k8s-master01 efk-7.10.2]# kubectl get pod -n logging
NAME                              READY   STATUS              RESTARTS      AGE
elasticsearch-logging-0           1/1     Running             0             3m53s
fluentd-es-v3.1.1-5dzrv           0/1     Running             0             45s
fluentd-es-v3.1.1-87jzz           0/1     ContainerCreating   0             45s
kibana-logging-5f48c7cb86-twxgz   1/1     Running             2 (84s ago)   3m24s
[root@k8s-master01 efk-7.10.2]# kubectl get svc -n logging
NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
elasticsearch-logging   ClusterIP   None            <none>        9200/TCP,9300/TCP   4m11s
kibana-logging          NodePort    10.99.122.222   <none>        5601:32080/TCP      3m36s

```
http://192.168.99.71:32080/kibana/




