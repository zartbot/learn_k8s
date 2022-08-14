
### docker search

```bash

zartbot@kevin-4GPU:~$ docker search nginx
NAME                                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
nginx                                             Official build of Nginx.                        17213     [OK]
linuxserver/nginx                                 An Nginx container, brought to you by LinuxS…   173
bitnami/nginx                                     Bitnami nginx Docker Image                      138                  [OK]
ubuntu/nginx                                      Nginx, a high-performance reverse proxy & we…   56
bitnami/nginx-ingress-controller                  Bitnami Docker Image for NGINX Ingress Contr…   19                   [OK]
rancher/nginx-ingress-controller                                                                  10
webdevops/nginx                                   Nginx container                                 9                    [OK]
ibmcom/nginx-ingress-controller                   Docker Image for IBM Cloud Private-CE (Commu…   4
bitnami/nginx-ldap-auth-daemon                                                                    3
rancher/nginx                                                                                     2
vmware/nginx                                                                                      2
kasmweb/nginx                                     An Nginx image based off nginx:alpine and in…   2
rancher/nginx-ingress-controller-defaultbackend                                                   2
rapidfort/nginx                                   RapidFort optimized, hardened image for NGINX   2
bitnami/nginx-exporter                                                                            2
vmware/nginx-photon                                                                               1
wallarm/nginx-ingress-controller                  Kubernetes Ingress Controller with Wallarm e…   1
bitnami/nginx-intel                                                                               1
rancher/nginx-conf                                                                                0
ibmcom/nginx-ppc64le                              Docker image for nginx-ppc64le                  0
rapidfort/nginx-ib                                RapidFort optimized, hardened image for NGIN…   0
rancher/nginx-ssl                                                                                 0
continuumio/nginx-ingress-ws                                                                      0
rancher/nginx-ingress-controller-amd64                                                            0
ibmcom/nginx-ingress-controller-ppc64le           Docker Image for IBM Cloud Private-CE (Commu…   0
```

### docker pull

```bash
docker pull nginx
docker pull nginx:1.15
```

### docker run

容器调试，带-it参数起动,例如进入bash
```bash
docker run -it nginx bash 

zartbot@kevin-4GPU:~$ docker run -it nginx bash
root@004dba0b7ca9:/# ls

```

端口mapping
```bash
docker run -it -p 1111:80 nginx bash 
```
File mapping
```bash
docker run -it -p 1111:80 -v /etc/hosts:/etc/hosts nginx bash
```

### docker ps
```bash
zartbot@kevin-4GPU:~$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                        PORTS     NAMES
b3ca9cceda47   nginx     "/docker-entrypoint.…"   46 seconds ago   Exited (130) 4 seconds ago              sharp_elbakyan
004dba0b7ca9   nginx     "/docker-entrypoint.…"   2 minutes ago    Exited (127) 54 seconds ago             affectionate_moore
```
get by container id
```bash
zartbot@kevin-4GPU:~$ docker ps -aq
b3ca9cceda47
004dba0b7ca9
```
### Run as daemon
```bash
zartbot@kevin-4GPU:~$ docker run -d nginx
5fb2fea7c60983cdd362bfbd7d7014670abe90cecb53b1d6c5ad943ffbcb65cc

zartbot@kevin-4GPU:~$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
5fb2fea7c609   nginx     "/docker-entrypoint.…"   5 seconds ago   Up 4 seconds   80/tcp    musing_driscoll
```
### attach cli
```bash
docker exec -it 5fb2fea7c609 bash 
```

### copy File
docker cp a.cu 5fb2fea7c609:/tmp
docker exec 5fb2fea7c609 cat /tmp/a.cu


### Remove 
```bash
docker stop 5fb2fea7c609
docker rm 5fb2fea7c609


zartbot@kevin-4GPU:~$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS                       PORTS     NAMES
9294c124d450   nginx     "/docker-entrypoint.…"   4 minutes ago   Exited (0) 4 minutes ago               dazzling_hellman
b3ca9cceda47   nginx     "/docker-entrypoint.…"   7 minutes ago   Exited (130) 6 minutes ago             sharp_elbakyan
004dba0b7ca9   nginx     "/docker-entrypoint.…"   8 minutes ago   Exited (127) 7 minutes ago             affectionate_moore

zartbot@kevin-4GPU:~$ docker rm 9294c124d450 004dba0b7ca9 b3ca9cceda47


```


### docker image 

```bash

zartbot@kevin-4GPU:~$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
nginx        latest    b692a91e4e15   6 days ago   142MB
zartbot@kevin-4GPU:~$ docker rmi b692a91e4e15
Untagged: nginx:latest
```
