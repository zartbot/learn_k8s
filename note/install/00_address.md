## Address allocation

|addresss|nodename| comments|
|-----|-----|---|
|192.168.99.71~73| k8s-master01~03| master*3|
|192.168.99.74~75| k8s-node01~02| node*3|
|192.168.99.70| VIP | keepalived virtual IP|


## Subnets

|Prefix | Usage|
|-|-|
|10.99.0.0/16| Service|
|172.16.0.0/12| Pod|
