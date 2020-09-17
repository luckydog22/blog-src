---
title: 安装k8s集群
date: 2020-04-16 04:00:48
categories: 虚拟化/容器
tags:
- Kubernetes
- Docker
- ProjectEdge
---

@[TOC]

## 一、关闭防火墙

### CentOS

```bash
systemctl stop firewalld
systemctl disable firewalld
systemctl status firewalld
```

### Ubuntu

```bash
ufw disable
```

## 二、配置apt/yum源（云主机可跳过此步）

### CentOS 7

```bash
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum makecache
setenforce 0
```

### CentOS 8

```bash
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-8.repo
yum makecache
setenforce 0
```

### Ubuntu 16.04

```bash
cat << EOF > /etc/apt/sources.list
deb http://mirrors.aliyun.com/ubuntu/ xenial main
deb-src http://mirrors.aliyun.com/ubuntu/ xenial main

deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main

deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates universe

deb http://mirrors.aliyun.com/ubuntu/ xenial-security main
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main
deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security universe
EOF
sudo apt update
```

### Ubuntu 18.04

```bash
cat << EOF > /etc/apt/sources.list
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
EOF
sudo apt update
```

## 三、安装kubernetes部署工具

### CentOS

```bash
curl http://mirrors.cloud.aliyuncs.com/docker-ce/linux/centos/docker-ce.repo > /etc/yum.repos.d/docker-ce.repo

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum install net-tools sshpass docker-ce docker-ce-cli containerd.io kubelet kubeadm kubectl --nogpgcheck -y
```

### Ubuntu

```bash
apt-get update && apt-get install -y install apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 

sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF

apt install net-tools sshpass docker-ce docker-ce-cli containerd.io kubelet kubeadm kubectl -y
```

## 四、配置Docker镜像源

```bash
echo '{"registry-mirrors": ["https://3m7egqv1.mirror.aliyuncs.com"]}' > /etc/docker/daemon.json
systemctl daemon-reload
systemctl restart docker
```

## 五、配置系统

```bash
setenforce 0
# 重置路由表
iptables -P FORWARD ACCEPT
echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
echo 1 > /proc/sys/net/bridge/bridge-nf-call-ip6tables
# 关闭交换分区
swapoff -a
# 还要修改/etc/fstab文件注释掉swap分区
```

## 六、配置k8s

```yaml
# init-config.yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
imageRepository: gcr.azk8s.cn/google_containers
kubernetesVersion: v1.17.0
networking:
  podSubnet: "172.16.0.0/16"
```

```bash
# 下载所需镜像
kubeadm config images pull --config=init-config.yaml
# 初始化kubernetes
kubeadm init --config=init-config.yaml --ignore-preflight-errors=NumCPU
# 配置kubectl
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
# 配置kubernetes网络
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# 设置master为可执行node节点（可选）
kubectl taint nodes --all node-role.kubernetes.io/master-

# 配置ApiServer SSL证书
grep 'client-certificate-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d > kubecfg.crt
grep 'client-key-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d > kubecfg.key
openssl pkcs12 -export -clcerts -inkey kubecfg.key -in kubecfg.crt -out kubecfg.p12 -name "kubernetes-client"
```

## 七、安装k8s Dashboard

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}') > token.txt
```

客户端安装CA证书

浏览器访问：https://\<master-ip\>:6443/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/