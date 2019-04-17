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

## 选择网络插件

[参考](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#pod-network)
注意某些插件需要在`kubeadm init`步骤传入相关参数，例如：如果选择 flannel 插件,为了让 flannel 正常工作必须在`kubeadm init`时候传入`--pod-network-cidr=10.244.0.0/16`。

## kubeadm init

```bash
kubeadm init --kubernetes-version v1.14.1 --pod-network-cidr=10.244.0.0/16
```

## To make kubectl work for your non-root user

[脚本](./kubectl-non-root.sh)

## deploy a pod network to the cluster

Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

成功提示:

```bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.1.43:6443 --token wqj3aj.lfnk85vqi37ha9mf \
    --discovery-token-ca-cert-hash sha256:27166d951180b268309d0d458231d6920beba83cb999d2bdce1f48eabc669496
```

## troubleshooter

### docker 未开启

systemctl enable docker.service