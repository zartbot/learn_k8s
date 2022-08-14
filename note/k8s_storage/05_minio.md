
Install minio
```bash
[root@k8s-master01 ceph]# kubectl minio init
namespace/minio-operator unchanged
-----------------
 
To open Operator UI, start a port forward using this command:
 
kubectl minio proxy -n minio-operator 
 
-----------------
[root@k8s-master01 ceph]# kubectl get pods -n minio-operator
NAME                              READY   STATUS              RESTARTS   AGE
console-68fb58cbcb-vwfm4          0/1     ContainerCreating   0          15s
minio-operator-6b8b678c66-jfr9v   0/1     ContainerCreating   0          19s
minio-operator-6b8b678c66-nz22p   0/1     ContainerCreating   0          19s
```
 
