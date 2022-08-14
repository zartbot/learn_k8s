## install Helms

```bash
 curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
 chmod 700 get_helm.sh
 ./get_helm.sh
```


## install krew
 
```bash
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)
```
 
add path to .bashrc
 
```bash
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
 
[root@k8s-master01 ceph]# kubectl krew update
Updated the local copy of plugin index.
 
[root@k8s-master01 ceph]# kubectl krew install minio
Updated the local copy of plugin index.
Installing plugin: minio
Installed plugin: minio
\
| Use this plugin:
|  kubectl minio
| Documentation:
|  https://github.com/minio/operator/tree/master/kubectl-minio
| Caveats:
| \
|  | * For resources that are not in default namespace, currently you must
|  |   specify -n/--namespace explicitly (the current namespace setting is not
|  |   yet used).
| /
/
WARNING: You installed plugin "minio" from the krew-index plugin repository.
   These plugins are not audited for security by the Krew maintainers.
   Run them at your own risk.
 
```
 