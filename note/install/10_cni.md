## Calico

```bash
cd /root/k8s-ha-install/calico/
sed -i "s#POD_CIDR#172.16.0.0/12#g" calico.yaml

grep "CALICO_IPV4POOL_CIDR" -A 1 calico.yaml
            - name: CALICO_IPV4POOL_CIDR
              value: "172.16.0.0/12"

kubectl apply -f calico.yaml
```

Verify
```bash
[root@k8s-master01 calico]# kubectl get po -n kube-system
NAME                                       READY   STATUS    RESTARTS       AGE
calico-kube-controllers-66686fdb54-ng5tq   1/1     Running   0              4m33s
calico-node-44mh2                          1/1     Running   0              4m35s
calico-node-kddr8                          1/1     Running   0              4m35s
calico-node-lr5qn                          1/1     Running   1 (2m9s ago)   4m35s
calico-node-mwsf8                          1/1     Running   0              4m35s
calico-node-xqg4p                          1/1     Running   0              4m35s
calico-typha-67c6dc57d6-6hsh6              1/1     Running   0              4m35s
calico-typha-67c6dc57d6-bhwjv              1/1     Running   0              4m35s
calico-typha-67c6dc57d6-f4rmt              1/1     Running   0              4m35s
[root@k8s-master01 calico]# kubectl get node
NAME           STATUS   ROLES    AGE   VERSION
k8s-master01   Ready    <none>   53m   v1.22.0
k8s-master02   Ready    <none>   53m   v1.22.0
k8s-master03   Ready    <none>   53m   v1.22.0
k8s-node01     Ready    <none>   53m   v1.22.0
k8s-node02     Ready    <none>   53m   v1.22.0
```

