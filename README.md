# kubernetes

## 安装centos7

[iso](http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso)
安装选择有界面的，安装完之后需要设置网络，需要网络连接正常。

- [停止防火墙](./stop-firewall.sh)
- [安装](./setup.sh)
- [开启服务](./start-kube.sh)

## troublesshoot

### no API token found for service account default/default

[solution](https://github.com/kubernetes/kubernetes/issues/11355#issuecomment-127378691)

### SchedulerPredicates failed due to PersistentVolumeClaim

需要学习k8s中的[volume](https://kubernetes.io/docs/concepts/storage/volumes/)
