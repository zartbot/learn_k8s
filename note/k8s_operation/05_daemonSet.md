Daemon Set is used as a daemon service which need to be deployed in every node.


Like fluentd,node exporter,logstash,collectd etc...

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app.kubernetes.io/component: exporter
    app.kubernetes.io/name: node-exporter
  name: node-exporter
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app.kubernetes.io/component: exporter
      app.kubernetes.io/name: node-exporter
  template:
    metadata:
      labels:
        app.kubernetes.io/component: exporter
        app.kubernetes.io/name: node-exporter
    spec:
      containers:
      - args:
        - --path.sysfs=/host/sys
        - --path.rootfs=/host/root
        - --no-collector.wifi
        - --no-collector.hwmon
        - --collector.filesystem.ignored-mount-points=^/(dev|proc|sys|var/lib/docker/.+|var/lib/kubelet/pods/.+)($|/)
        - --collector.netclass.ignored-devices=^(veth.*)$
        name: node-exporter
        image: prom/node-exporter
        ports:
          - containerPort: 9100
            protocol: TCP
        resources:
          limits:
            cpu: 250m
            memory: 180Mi
          requests:
            cpu: 102m
            memory: 180Mi
        volumeMounts:
        - mountPath: /host/sys
          mountPropagation: HostToContainer
          name: sys
          readOnly: true
        - mountPath: /host/root
          mountPropagation: HostToContainer
          name: root
          readOnly: true
      volumes:
      - hostPath:
          path: /sys
        name: sys
      - hostPath:
          path: /
        name: root
```          

```bash
kubectl create namespace monitoring
[root@k8s-master01 ~]# kubectl create -f daemon.yaml
daemonset.apps/node-exporter created

[root@k8s-master01 ~]# kubectl get po -n monitoring -o wide
NAME                  READY   STATUS              RESTARTS   AGE   IP              NODE           NOMINATED NODE   READINESS GATES
node-exporter-4ptdz   1/1     Running             0          24s   172.27.14.208   k8s-node02     <none>           <none>
node-exporter-bpgpz   0/1     ContainerCreating   0          24s   <none>          k8s-master02   <none>           <none>
node-exporter-nsh4r   0/1     ContainerCreating   0          24s   <none>          k8s-master01   <none>           <none>
node-exporter-pdgsm   1/1     Running             0          24s   172.17.125.33   k8s-node01     <none>           <none>
node-exporter-ws5kg   1/1     Running             0          24s   172.18.195.10   k8s-master03   <none>           <none>



```


```bash
查看 DaemonSet 更新方式： 


[root@k8s-master01 ~]#  kubectl get ds node-exporter -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}'  -n monitoring
RollingUpdate


RollingUpdate 命令式更新， 和之前的 Deployment、 StatefulSet 方式 一致： 

# kubectl edit ds/<daemonset-name> 
# kubectl patch ds/<daemonset-name> -p=<strategic-merge-patch> 

更新镜像： 
# kubectl set image ds/<daemonset-name><container-name>=<containernew-image> --record=true

查看更新状态：

# kubectl rollout status ds/<daemonset-name> 

列出所有修订版本： 
# kubectl rollout history daemonset <daemonset-name> 
回滚到指定 revision： 
# kubectl rollout undo daemonset <daemonset-name> --to-revision=<revision>
```
