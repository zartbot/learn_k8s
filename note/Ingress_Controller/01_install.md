```bash

[root@k8s-master01 efk-7.10.2]# helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
"ingress-nginx" has been added to your repositories

[root@k8s-master01 efk-7.10.2]# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "ingress-nginx" chart repository
...Successfully got an update from the "ceph-csi" chart repository
Update Complete. ⎈Happy Helming!⎈

[root@k8s-master01 efk-7.10.2]# helm pull ingress-nginx/ingress-nginx --version 4.0.1
[root@k8s-master01 efk-7.10.2]# tar xf ingress-nginx-4.0.1.tgz
[root@k8s-master01 efk-7.10.2]# cd ingress-nginx/

```
edit values.yaml

* dnsPolicy: ClusterFirstWithHostNet
* hostNetwork: true
* NodeSelector
* kind --> Daemon
*  ingressClassResource:
    name: nginx
    enabled: true
    default: true

```bash

[root@k8s-master01 ingress-nginx]#
[root@k8s-master01 ingress-nginx]# kubectl label node k8s-node01 ingress=true
node/k8s-node01 labeled
[root@k8s-master01 ingress-nginx]# kubectl label node k8s-node02 ingress=true
node/k8s-node02 labeled
[root@k8s-master01 ingress-nginx]# kubectl create ns ingress-nginx
namespace/ingress-nginx created


[root@k8s-master01 ingress-nginx]# helm install ingress-nginx -n ingress-nginx .
NAME: ingress-nginx
LAST DEPLOYED: Sun Aug 14 15:30:11 2022
NAMESPACE: ingress-nginx
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace ingress-nginx get services -o wide -w ingress-nginx-controller'

An example Ingress that makes use of the controller:

  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    annotations:
      kubernetes.io/ingress.class:
    name: example
    namespace: foo
  spec:
    rules:
      - host: www.example.com
        http:
          paths:
            - backend:
                serviceName: exampleService
                servicePort: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
        - hosts:
            - www.example.com
          secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
[root@k8s-master01 ingress-nginx]#
```


```bash
[root@k8s-master01 ingress-nginx]# kubectl create ns demo-ingress
namespace/demo-ingress created
[root@k8s-master01 ingress-nginx]# kubectl create deploy nginx --image=registry.cn-beijing.aliyuncs.com/dotbalo/nginx:1.15.12 -n demo-ingress
deployment.apps/nginx created
```
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: demo-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: nginx.test.com
    http:
      paths:
      - backend:
          service:
            name: nginx
            port:
              number: 80
        path: /
        pathType: ImplementationSpecific 
```

```bash

[root@k8s-master01 ingress-nginx]# kubectl create -f web-ingress.yaml
ingress.networking.k8s.io/nginx-ingress created
```
