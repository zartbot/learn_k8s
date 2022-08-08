## sync configuration


on Master1
```bash
cd /etc/kubernetes

for NODE in k8s-master02 k8s-master03 k8s-node01 k8s-node02; do  for FILE in pki/ca.pem pki/ca-key.pem pki/front-proxy-ca.pem bootstrap-kubelet.kubeconfig; do scp /etc/kubernetes/$FILE $NODE:/etc/kubernetes/${FILE}; done done
```
All node mkdir

```bash
mkdir -p /var/lib/kubelet /var/log/kubernetes /etc/systemd/system/kubelet.service.d /etc/kubernetes/manifests/
```

create startup service
```bash

vim /usr/lib/systemd/system/kubelet.service

[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/local/bin/kubelet

Restart=always
StartLimitInterval=0
RestartSec=10

[Install]
WantedBy=multi-user.target
```

## kubulet service configuration

all node

```bash
vim /etc/systemd/system/kubelet.service.d/10-kubelet.conf

[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig --kubeconfig=/etc/kubernetes/kubelet.kubeconfig"
Environment="KUBELET_SYSTEM_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin"
Environment="KUBELET_CONFIG_ARGS=--config=/etc/kubernetes/kubelet-conf.yml --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.5"
Environment="KUBELET_EXTRA_ARGS=--node-labels=node.kubernetes.io/node='' "
ExecStart=
ExecStart=/usr/local/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_SYSTEM_ARGS $KUBELET_EXTRA_ARGS

```

all node
```bash
vim /etc/kubernetes/kubelet-conf.yml
```
change the 10.99.0.10 as the 10th IP in your service subnet
```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
address: 0.0.0.0
port: 10250
readOnlyPort: 10255
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.pem
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
cgroupDriver: systemd
cgroupsPerQOS: true
clusterDNS:
- 10.99.0.10
clusterDomain: cluster.local
containerLogMaxFiles: 5
containerLogMaxSize: 10Mi
contentType: application/vnd.kubernetes.protobuf
cpuCFSQuota: true
cpuManagerPolicy: none
cpuManagerReconcilePeriod: 10s
enableControllerAttachDetach: true
enableDebuggingHandlers: true
enforceNodeAllocatable:
- pods
eventBurst: 10
eventRecordQPS: 5
evictionHard:
  imagefs.available: 15%
  memory.available: 100Mi
  nodefs.available: 10%
  nodefs.inodesFree: 5%
evictionPressureTransitionPeriod: 5m0s
failSwapOn: true
fileCheckFrequency: 20s
hairpinMode: promiscuous-bridge
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 20s
imageGCHighThresholdPercent: 85
imageGCLowThresholdPercent: 80
imageMinimumGCAge: 2m0s
iptablesDropBit: 15
iptablesMasqueradeBit: 14
kubeAPIBurst: 10
kubeAPIQPS: 5
makeIPTablesUtilChains: true
maxOpenFiles: 1000000
maxPods: 110
nodeStatusUpdateFrequency: 10s
oomScoreAdj: -999
podPidsLimit: -1
registryBurst: 10
registryPullQPS: 5
resolvConf: /etc/resolv.conf
rotateCertificates: true
runtimeRequestTimeout: 2m0s
serializeImagePulls: true
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 4h0m0s
syncFrequency: 1m0s
volumeStatsAggPeriod: 1m0s
```

## start service
```bash
systemctl daemon-reload && systemctl enable --now kubelet
```

## Config KubeProxy

on Master01 generate config
```bash

cd /root/k8s-ha-install
kubectl -n kube-system create serviceaccount kube-proxy

kubectl create clusterrolebinding system:kube-proxy \
--clusterrole system:node-proxier \
--serviceaccount kube-system:kube-proxy

SECRET=$(kubectl -n kube-system get sa/kube-proxy \
 --output=jsonpath='{.secrets[0].name}')

JWT_TOKEN=$(kubectl -n kube-system get secret/$SECRET \
 --output=jsonpath='{.data.token}' | base64 -d)


[root@k8s-master01 k8s-ha-install]# echo $JWT_TOKEN
eyJhbGciOiJSUzI1NiIsImtpZCI6IjBZZkswR1FXZFowa2xBdjBBT2lQRThxVXVCSlNndDRtemZ5bFBUeDR2RzAifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlLXByb3h5LXRva2VuLTY1enQ3Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6Imt1YmUtcHJveHkiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI5OWIwYzFkOC02ZTBiLTRkM2QtODc4Ni00MGFhMDRlODE3NjgiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06a3ViZS1wcm94eSJ9.Livn1rrbkdDDtOXFyoWYIi_ZJse69pCSCdmZC_DsF9RgfYJUYsgl0nCyUG2zzJA2ktPTAvGfg2QCFKbz2nVq1u6DS9Zx4jOsnAe4jQ7ru6dWp3P9iB-VBzCsx_Mkim3XLqYaklNyrSEhx8RiNLDkrh71Oc6b8UTNM5p7d0UGglHJOiiIZmuCyFYoztzKw0nwpdz7x8Fgd0Uxr_rwT37u-OGDG0Yi4LNJwgRk9FSPJ8DTqEr-7gs-3RO2ViHgEUG34FSv7WEhUbTOsWuCdsBHZ8MIWJVOnlwfq850lKpWWXMuwiZWl_1XW9j0zjwE1jz6NcL3l-WBD4Sa5B-SAlaMhg



PKI_DIR=/etc/kubernetes/pki
K8S_DIR=/etc/kubernetes

kubectl config set-cluster kubernetes \
--certificate-authority=/etc/kubernetes/pki/ca.pem \
--embed-certs=true \
--server=https://192.168.99.70:16643 \
--kubeconfig=${K8S_DIR}/kube-proxy.kubeconfig

kubectl config set-credentials kubernetes \
--token=${JWT_TOKEN} \
--kubeconfig=${K8S_DIR}/kube-proxy.kubeconfig

kubectl config set-context  kubernetes \
--cluster=kubernetes  \
--user=kubernetes \
--kubeconfig=${K8S_DIR}/kube-proxy.kubeconfig


kubectl config use-context  kubernetes \
--kubeconfig=${K8S_DIR}/kube-proxy.kubeconfig

```

send to other nodes
```bash
for NODE in k8s-master02 k8s-master03 k8s-node01 k8s-node02; do scp /etc/kubernetes/kube-proxy.kubeconfig $NODE:/etc/kubernetes/kube-proxy.kubeconfig; done
```

setup service file on all node

```bash
vim /usr/lib/systemd/system/kube-proxy.service
```
```yaml
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-proxy \
  --config=/etc/kubernetes/kube-proxy.yaml \
  --v=2

Restart=always
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

```bash
vim /etc/kubernetes/kube-proxy.yaml
```

```yaml
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
clientConnection:
  acceptContentTypes: ""
  burst: 10
  contentType: application/vnd.kubernetes.protobuf
  kubeconfig: /etc/kubernetes/kube-proxy.kubeconfig
  qps: 5
clusterCIDR: 172.16.0.0/12 
configSyncPeriod: 15m0s
conntrack:
  max: null
  maxPerCore: 32768
  min: 131072
  tcpCloseWaitTimeout: 1h0m0s
  tcpEstablishedTimeout: 24h0m0s
enableProfiling: false
healthzBindAddress: 0.0.0.0:10256
hostnameOverride: ""
iptables:
  masqueradeAll: false
  masqueradeBit: 14
  minSyncPeriod: 0s
  syncPeriod: 30s
ipvs:
  masqueradeAll: true
  minSyncPeriod: 5s
  scheduler: "rr"
  syncPeriod: 30s
kind: KubeProxyConfiguration
metricsBindAddress: 127.0.0.1:10249
mode: "ipvs"
nodePortAddresses: null
oomScoreAdj: -999
portRange: ""
udpIdleTimeout: 250ms
```

```bash
systemctl daemon-reload && systemctl enable --now kube-proxy 
```