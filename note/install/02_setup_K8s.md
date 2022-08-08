## Clone source code

on Master01
```bash
cd /root/; git clone https://gitee.com/dukuan/k8s-ha-install.git
```

## upgrade Kernel

on all server instances

```bash
yum upgrade -y && reboot
```

on Master1 wget kernel file
```bash
wget http://193.49.22.109/elrepo/kernel/el7/x86_64/RPMS/kernel-ml-devel-4.19.12-1.el7.elrepo.x86_64.rpm

wget http://193.49.22.109/elrepo/kernel/el7/x86_64/RPMS/kernel-ml-4.19.12-1.el7.elrepo.x86_64.rpm


for i in k8s-master02 k8s-master03 k8s-node01 k8s-node02; do scp  kernel-ml-4.19.12-1.el7.elrepo.x86_64.rpm kernel-ml-devel-4.19.12-1.el7.elrepo.x86_64.rpm $i:/root/ ;done

```

on all server 
```bash
cd /root && yum localinstall -y kernel-ml*
grub2-set-default 0 && grub2-mkconfig -o /etc/grub2.cfg
 grubby --args="user_namespace.enable = 1
" --update-kernel="$(grubby --default-kernel)"

grubby --default-kernel
> check version to 4.19

```
reboot then check version with uname -a
```bash
uname -a
```

## install IPVS
all nodes
```bash
yum install ipvsadm ipset sysstat conntrack libseccomp -y

vim /etc/modules-load.d/ipvs.conf
>>append
ip_vs
ip_vs_lc
ip_vs_wlc
ip_vs_rr
ip_vs_wrr
ip_vs_lblc
ip_vs_lblcr
ip_vs_dh
ip_vs_sh
ip_vs_fo
ip_vs_nq
ip_vs_sed
ip_vs_ftp
ip_vs_sh
nf_conntrack
ip_tables
ip_set
xt_set 
ipt_set 
ipt_rpfilter 
ipt_REJECT
ipip

```

然后执行

```bash
systemctl enable --now systemd-modules-load.service
```

开启一些K8s集群中必需的内核参数，所有节点配置K8s内核：

```bash
cat<<EOF>/etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
fs.may_detach_mounts = 1
net.ipv4.conf.all.route_localnet = 1
vm.overcommit_memory = 1
vm.panic_on_oom = 0
fs.inotify.max_user_watches=89100
fs.file-max=52706963
fs.nr_open=52706963
net.netfilter.nf_conntrack_max=2310720
net.ipv4.tcp_keepalive_time=600
net.ipv4.tcp_keepalive_probes=3
net.ipv4.tcp_keepalive_intvl = 15
net.ipv4.tcp_max_tw_buckets=36000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_orphans=327680
net.ipv4.tcp_orphan_retries=3
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.ip_conntrack_max=65536
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.tcp_timestamps=0
net.core.somaxconn = 16384
EOF




```

所有节点配置完内核后，重启服务器，保证重启后内核依旧加载：

```bash
sysctl --system
reboot

lsmod | grep --color=auto -e ip_vs -e nf_conntrack
```

