#### 配置calico
1. 官方文档地址 https://docs.projectcalico.org/v3.4/getting-started/kubernetes/installation/flannel
2. 下载镜像，并且安装在每个节点上
```
  docker pull quay-mirror.qiniu.com/calico/cni:v3.4.0
  docker pull quay-mirror.qiniu.com/calico/node:v3.4.0

  docker tag quay-mirror.qiniu.com/calico/cni:v3.4.0 quay.io/calico/cni:v3.4.0
  docker tag quay-mirror.qiniu.com/calico/node:v3.4.0 quay.io/calico/node:v3.4.0

  docker save quay.io/calico/cni:v3.4.0 quay.io/calico/node:v3.4.0 -o calico.tar.gz
  
  scp ...
```
