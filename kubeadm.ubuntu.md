# kubeadm

install kubernetes by kubeadm[参考](https://kubernetes.io/zh/docs/setup/independent/install-kubeadm/)

## 系统初始化

## 安装前检测

检测网络联通状况

## 安装CRI

container runtime interface
[脚本](./docker-cri.ubuntu.sh)

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

[脚本](./install-kubeadm-kubelet-kubectl.ubuntu.sh)

## 选择网络插件

[参考](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#pod-network)
注意某些插件需要在`kubeadm init`步骤传入相关参数，例如：如果选择 flannel 插件,为了让 flannel 正常工作必须在`kubeadm init`时候传入`--pod-network-cidr=10.244.0.0/16`。

## kubeadm init

```bash
kubeadm init --kubernetes-version=v1.14.1 --pod-network-cidr=10.244.0.0/16
```

完后成会在终端打印

## To make kubectl work for your non-root user

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## deploy a pod network to the cluster

Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

上面我选的`flannel`所以我开启命令为

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml
```

`master`节点以配置完毕，接下来需要配置`node`节点

## 添加node节点

- 关闭node节点防火墙(ubuntu默认没有开)
- 修改`/etc/resolvconf/resolv.conf.d/head`在里面追加 `nameserver 8.8.8.8(或者其他非本地dns服务器)`这是为了解决[coredns forward loop](### coredns forward loop)
- (可选)为了好辨识各个节点可以使用`hostnamectl set-hostname 节点名`修改节点的`hostname`
- 将`master`节点需要的`k8s.gcr.io/XXX`镜像同样`pull`到`node`节点上去
- 运行下面命令加入集群

```bash
kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
```

成功的话输出最后一行会是`Run 'kubectl get nodes'..... cluster.`

ps:
token 可以在`master`节点运行`kubeadm token list`获取

--discovery-token-ca-cert-hash 可以通过如下命令获取：

```bash
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
   openssl dgst -sha256 -hex | sed 's/^.* //'
```

[参考](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#join-nodes)

## 使用sonobuoy测试集群是否正常

[sonobuoy](https://github.com/heptio/sonobuoy)

## troubleshooter

### docker 未开启

systemctl enable docker.service

### coreDNS 不能正常工作

使用命令`ubectl describe pod/coredns-xxxxxx -n kube-system`
发现`Warning  Unhealthy Readiness probe failed: HTTP probe failed with statuscode: 503`

将防火墙关闭后

### coreDNS 状态是 crashloopbackoff

当CoreDNS日志包含消息时Loop ... detected ...，这意味着loop检测插件已在其中一个上游DNS服务器中检测到无限转发循环。这是一个致命的错误，因为使用无限循环操作将占用内存和CPU，直到主机最终导致内存不足。
转发循环通常由以下原因引起：
最常见的是，CoreDNS直接向自己转发请求。例如，通过环回地址，例如127.0.0.1，::1或127.0.0.53
不常见的是，CoreDNS转发到上游服务器，而上游服务器又将请求转发回CoreDNS。
[参考](https://coredns.io/plugins/loop/#troubleshooting)
解决方法:

1. 临时方法: 将`k8s slave`节点主机中`/etc/resolv.conf`文件内`nameserver`改为`8.8.8.8`或者其他非本地`dns`服务器,但是重启后就覆盖了。

2. ubuntu 16.04 永久方法: 修改 `/etc/resolvconf/resolv.conf.d/head`文件，在里面添加`nameserver 8.8.8.8(或其他非本地 dns 服务器)`

### couldn't validate the identity of the API Server

详情如下:

```bash
[preflight] Running pre-flight checks
    [WARNING Hostname]: hostname "node-5" could not be reached
    [WARNING Hostname]: hostname "node-5": lookup node-5 on 127.0.1.1:53: no such host
error execution phase preflight: couldn't validate the identity of the API Server: abort connecting to API servers after timeout of 5m0s
```

原因是`token`失效，在`master`节点使用命令`kubectl token create`重新创建 token
[参考](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#join-nodes)