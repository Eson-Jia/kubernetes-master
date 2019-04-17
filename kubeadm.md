# kubeadm

install kubernetes by kubeadm[参考](https://kubernetes.io/zh/docs/setup/independent/install-kubeadm/)

## 系统初始化

centos7 网络每次重启都需要开启，设置为自动连接

## 安装前检测

检测网络联通状况

## 安装CRI

container runtime interface
[脚本](./docker-cri.sh)

## 关闭swap分区

```bash
# swapoff -a

vim /etc/fstab
# 注释掉 swap那一行
```

## 安装 kubeadm, kubelet 和 kubectl

[脚本](./install-kubeadm-kubelet-kubectl.sh)

## 设置iptables

```bash
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

## kubeadm init

```bash
kubeadm init --kubernetes-version v1.14.1 --pod-network-cidr=10.244.0.0/16
```