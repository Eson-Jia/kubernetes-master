# kubeadm

install kubernetes by kubeadm[参考](https://kubernetes.io/zh/docs/setup/independent/install-kubeadm/)

## 系统初始化

centos7 网络每次重启都需要开启，设置为自动连接

## 安装前检测

检测网络联通状况

## 安装CRI

container runtime interface
[脚本](./docker-cri.sh)

## 将所需镜像下载到本地

因为墙的问题需要将镜像下到本地
[镜像列表](./image.md)

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
kubeadm init --kubernetes-version=v1.14.1 --pod-network-cidr=10.244.0.0/16
```

## To make kubectl work for your non-root user

[脚本](./kubectl-non-root.sh)

## deploy a pod network to the cluster

Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

上面我选的`flannel`所以我开启命令为

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml
```

`master`节点以配置完毕，接下来需要配置`node`节点

## 添加node节点

- 关闭node节点防火墙
- (可选)为了好辨识各个节点可以使用`hostnamectl set-hostname 节点名`修改节点的`hostname`否则`centos7`系统会显示为`bogon`
- 将`master`节点需要的`k8s.gcr.io/XXX`镜像同样`pull`到`node`节点上去
- 运行

```bash
kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
```

token 可以在`master`节点运行`kubeadm token list`获取

--discovery-token-ca-cert-hash 可以通过如下命令获取：

```bash
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
   openssl dgst -sha256 -hex | sed 's/^.* //'
```

[参考](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#join-nodes)

## troubleshooter

### docker 未开启

systemctl enable docker.service

### coreDNS 不能正常工作

使用命令`ubectl describe pod/coredns-xxxxxx -n kube-system`
发现`Warning  Unhealthy Readiness probe failed: HTTP probe failed with statuscode: 503`

将防火墙关闭后

### coreDNS 重启后报 crashloopbackoff

coreDNS 开始能正常工作但 master 节点重启之后就 crashloopbackoff

将防火墙关闭后问题解决，命令：

```bash
sudo systemctl disable firewalld
sudo systemctl stop firewalld
```

反思，这是因为没有验证防火墙对服务端口的影响，文档开头明明白白写着需要注意验证这些。

[参考](https://github.com/coredns/coredns/issues/2325)