## Install CFSSL

on master 01
```bash
wget "https://pkg.cfssl.org/R1.2/cfssl_linux-amd64" -O /usr/local/bin/cfssl

wget "https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64" -O /usr/local/bin/cfssljson

chmod a+x /usr/local/bin/cfssl /usr/local/bin/cfssljson

```

on all master node
```bash

mkdir -p /etc/etcd/ssl
```

on all node
```bash

mkdir -p /etc/kubernetes/pki
```

on master 01

```bash
cd /root/k8s-ha-install/
git checkout manual-installation-v1.22.x
cd pki


[root@k8s-master01 pki]# cat etcd-ca-csr.json
{
  "CN": "etcd",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Beijing",
      "L": "Beijing",
      "O": "etcd",
      "OU": "Etcd Security"
    }
  ],
  "ca": {
    "expiry": "876000h"
  }
}




cfssl gencert -initca etcd-ca-csr.json | cfssljson -bare /etc/etcd/ssl/etcd-ca
```

generate Cert
```bash

[root@k8s-master01 pki]# cat ca-config.json
{
  "signing": {
    "default": {
      "expiry": "876000h"
    },
    "profiles": {
      "kubernetes": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "876000h"
      }
    }
  }
}


cfssl gencert \
-ca=/etc/etcd/ssl/etcd-ca.pem \
-ca-key=/etc/etcd/ssl/etcd-ca-key.pem \
-config=ca-config.json \
-hostname=127.0.0.1,k8s-master01,k8s-master02,k8s-master03,192.168.99.71,192.168.99.72,192.168.99.73 \
-profile=kubernetes \
etcd-csr.json | cfssljson -bare /etc/etcd/ssl/etcd
```

复制证书到其它Master
```bash
MasterNode='k8s-master02 k8s-master03'
for NODE in $MasterNode; do  for FILE in etcd-ca-key.pem etcd-ca.pem etcd-key.pem etcd.pem; do scp /etc/etcd/ssl/${FILE} $NODE:/etc/etcd/ssl/${FILE} ;done done
```

## Generate K8S cert

on master01
```bash
cd /root/k8s-ha-install/pki 
cfssl gencert -initca ca-csr.json | cfssljson -bare /etc/kubernetes/pki/ca
```

### Generate kube-apiserver cert

* 10.99.0.0/16是Service网段
* 192.168.99.70是HA的VIP
* 192.168.99.71~73是Master

```bash
cfssl gencert \
-ca=/etc/kubernetes/pki/ca.pem \
-ca-key=/etc/kubernetes/pki/ca-key.pem \
-config=ca-config.json \
-hostname=10.99.0.1,192.168.99.70,127.0.0.1,kubernetes,kubernetes.default,kubernetes.default.svc,kubernetes.default.svc.cluster,kubernetes.default.svc.cluster.local,192.168.99.71,192.168.99.72,192.168.99.73 \
-profile=kubernetes \
apiserver-csr.json | cfssljson -bare /etc/kubernetes/pki/apiserver

```

### generate front-proxy cert 

```bash
cfssl gencert -initca front-proxy-ca-csr.json | cfssljson -bare /etc/kubernetes/pki/front-proxy-ca


cfssl gencert \
-ca=/etc/kubernetes/pki/front-proxy-ca.pem \
-ca-key=/etc/kubernetes/pki/front-proxy-ca-key.pem \
-config=ca-config.json \
-profile=kubernetes \
front-proxy-client-csr.json | cfssljson -bare /etc/kubernetes/pki/front-proxy-client

```
### Generate Controller-manage cert 

```bash
cfssl gencert \
-ca=/etc/kubernetes/pki/ca.pem \
-ca-key=/etc/kubernetes/pki/ca-key.pem \
-config=ca-config.json \
-profile=kubernetes \
manager-csr.json | cfssljson -bare /etc/kubernetes/pki/controller-manager
```

由于 Controller Manager 需要和 APIServer 进行通信， 此时可以使用`kubeconfig`配置链接APIServer的信息， 比如需要用到的证书、 APIServer的地址等。 接下来使用`Kubectl`生成一个Controller Manager 所用的`kubeconfig`。 

```bash
kubectl config set-cluster kubernetes \
--certificate-authority=/etc/kubernetes/pki/ca.pem \
--embed-certs=true \
--server=https://192.168.99.70:16643 \
--kubeconfig=/etc/kubernetes/controller-manager.kubeconfig

kubectl config set-context system:kube-controller-manager@kubernetes \
--cluster=kubernetes \
--user=system:kube-controller-manager \
--kubeconfig=/etc/kubernetes/controller-manager.kubeconfig

kubectl config set-credentials system:kube-controller-manager \
--client-certificate=/etc/kubernetes/pki/controller-manager.pem \
--client-key=/etc/kubernetes/pki/controller-manager-key.pem \
--embed-certs=true \
--kubeconfig=/etc/kubernetes/controller-manager.kubeconfig

kubectl config use-context system:kube-controller-manager@kubernetes \
--kubeconfig=/etc/kubernetes/controller-manager.kubeconfig

```

### Scheduler kubeconfig 

```bash
cfssl gencert \
-ca=/etc/kubernetes/pki/ca.pem \
-ca-key=/etc/kubernetes/pki/ca-key.pem \
-config=ca-config.json \
-profile=kubernetes \
scheduler-csr.json | cfssljson -bare /etc/kubernetes/pki/scheduler


kubectl config set-cluster kubernetes \
--certificate-authority=/etc/kubernetes/pki/ca.pem \
--embed-certs=true \
--server=https://192.168.99.70:16643 \
--kubeconfig=/etc/kubernetes/scheduler.kubeconfig

kubectl config set-credentials system:kube-scheduler \
--client-certificate=/etc/kubernetes/pki/scheduler.pem \
--client-key=/etc/kubernetes/pki/scheduler-key.pem \
--embed-certs=true \
--kubeconfig=/etc/kubernetes/scheduler.kubeconfig



kubectl config set-context system:kube-scheduler@kubernetes \
--cluster=kubernetes \
--user=system:kube-scheduler \
--kubeconfig=/etc/kubernetes/scheduler.kubeconfig



kubectl config use-context system:kube-scheduler@kubernetes \
--kubeconfig=/etc/kubernetes/scheduler.kubeconfig

```

### APIServer kubeconfig

```bash
cfssl gencert \
-ca=/etc/kubernetes/pki/ca.pem \
-ca-key=/etc/kubernetes/pki/ca-key.pem \
-config=ca-config.json \
-profile=kubernetes \
admin-csr.json | cfssljson -bare /etc/kubernetes/pki/admin


kubectl config set-cluster kubernetes \
--certificate-authority=/etc/kubernetes/pki/ca.pem \
--embed-certs=true \
--server=https://192.168.99.70:16643 \
--kubeconfig=/etc/kubernetes/admin.kubeconfig

kubectl config set-credentials system:kube-admin \
--client-certificate=/etc/kubernetes/pki/admin.pem \
--client-key=/etc/kubernetes/pki/admin-key.pem \
--embed-certs=true \
--kubeconfig=/etc/kubernetes/admin.kubeconfig



kubectl config set-context system:kube-admin@kubernetes \
--cluster=kubernetes \
--user=system:kube-admin \
--kubeconfig=/etc/kubernetes/admin.kubeconfig



kubectl config use-context system:kube-admin@kubernetes \
--kubeconfig=/etc/kubernetes/admin.kubeconfig

```

### Create ServiceAccount Secret Token and verification key
```bash

openssl genrsa -out /etc/kubernetes/pki/sa.key 2048
openssl rsa -in /etc/kubernetes/pki/sa.key -pubout -out /etc/kubernetes/pki/sa.pub
```

### Dispatch all cert to other master
```bash

MasterNode='k8s-master02 k8s-master03'
for NODE in $MasterNode; do  for FILE in $(ls /etc/kubernetes/pki | grep -v etcd); do scp /etc/kubernetes/pki/${FILE} $NODE:/etc/kubernetes/pki/${FILE} ;done; for FILE in admin.kubeconfig controller-manager.kubeconfig scheduler.kubeconfig; do  scp /etc/kubernetes/${FILE} $NODE:/etc/kubernetes/${FILE} ; done; done
```
