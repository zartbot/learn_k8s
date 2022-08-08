It has 2 option for containerd or docker as runtime

## docker as Runtime

all node install docker-ce-20.10

```bash

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum install docker-ce-20.10.* docker-ce-cli-20.10.* -y

mkdir /etc/docker
cat > /etc/docker/daemon.json <<EOF
{
    "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

systemctl daemon-reload && systemctl enable --now docker
```

