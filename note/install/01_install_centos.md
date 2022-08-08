# Install Centos

* version Centos 7.9
* Server with GUI



`/etc/hosts` 

```bash
192.168.99.71 k8s-master01
192.168.99.72 k8s-master02
192.168.99.73 k8s-master03
192.168.99.74 k8s-node01
192.168.99.75 k8s-node02
```

## Yum repo for China installation(Optional)

```bash
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo


yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

```

## Yum install utils
```bash
yum install -y yum-utils device-mapper-persistent-data lvm2
yum install wget jq psmisc vim net-tools telnet git -y
```

## Disable firewall , SElinux and DNSmasq

```bash
systemctl disable --now firewalld
systemctl disable --now dnsmasq
systemctl disable --now NetworkManager

setenforce 0
sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/sysconfig/selinux
sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/selinux/config
```

## Disable Swap

```bash
swapoff -a && sysctl -w vm.swappiness=0
sed -ri '/^[^#]*swap/s@^@#@' /etc/fstab
```
## install NTPDate

> this step is optional, server with GUI already installed NTPdate
```bash
rpm -ivh http://mirrors.wlnmp.com/centos/wlnmp-release-centos.noarch.rpm
yum install ntpdate -y
```

Setup time zone and sync time
```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
echo 'Asia/Shanghai' > /etc/timezone
ntpdate 173.39.145.1 [other ntp: time2.aliyun.com]

```
Add to crontab

```bash
crontab -e
*/5 * * * * /usr/sbin/ntpdate 173.39.145.1
```

Config limit
```bash
ulimit -SHn 65535
vim /etc/security/limits.conf

* soft nofile 65536 
* hard nofile 131072
* soft nproc 65535
* hard nproc 655350
* soft memlock unlimited
* hard memlock unlimited
```
## config ssh-key-gen

```bash
su - root
ssh-keygen -t rsa
for i in k8s-master01 k8s-master02 k8s-master03 k8s-node01 k8s-node02; do ssh-copy-id -i .ssh/id_rsa.pub $i;done
```
