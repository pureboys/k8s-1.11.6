# K8s 1.13.1 配置

### Master/Node:

1. cd  /etc/yum.repos.d/ && wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
2. 
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```
3. yum install -y --setopt=obsoletes=0 docker-ce-17.03.1.ce-1.el7.centos kubelet kubeadm kubectl
4. systemctl enable docker kubelet && systemctl start docker
5. swapoff -a  # 临时关闭swap, 永久关闭 注释/etc/fstab文件里swap相关的行
6. 
```
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
vm.swappiness=0
EOF

sysctl --system

[root@k8s-manager labs]# cat /proc/sys/net/bridge/bridge-nf-call-iptables
1
[root@k8s-manager labs]# cat /proc/sys/net/bridge/bridge-nf-call-ip6tables
1
```

7. vi /etc/sysconfig/kubelet

```
KUBELET_EXTRA_ARGS="--node-ip={YOUR_IP}"
```

8. 
```
 kubeadm init --kubernetes-version=v1.13.1  --apiserver-advertise-address=11.11.11.111 --pod-network-cidr=10.244.0.0/16 servicecidr=10.96.0.0/12 --image-repository=registry.aliyuncs.com/google_containers
```
9. 
```
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```
10. 添加加flannel网络附件
```
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# 如果Node有多个网卡的话，参考flannel issues 39701，
# https://github.com/kubernetes/kubernetes/issues/39701
# 目前需要在kube-flannel.yml中使用--iface参数指定集群主机内网网卡的名称，
# 否则可能会出现dns无法解析。容器无法通信的情况，需要将kube-flannel.yml下载到本地，
# flanneld启动参数加上--iface=<iface-name>
    containers:
      - name: kube-flannel
        image: quay.io/coreos/flannel
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface=enp0s8

# 启动
kubectl apply -f kube-flannel.yml

# 查看
kubectl get pods --namespace kube-system
kubectl get svc --namespace kube-system

```

11.  docker save $(docker images | sed '1d' | awk '{print $1 ":" $2 }') -o k8s.tar.gz


### Node:

1. 执行 1-7 步骤后
2. 执行 docker load -i k8s.tar.gz
3. kubeadm join 11.11.11.111:6443 --token n546cq.2b6og68bc4753rhi --discovery-token-ca-cert-hash sha256:073320ca3ea5447fe93d9f52b09f2c1480a3d83528feb82b32e9450dc53d50a1


### issue:
1. 有时候join失败, 肯能是防火墙没有关
```
systemctl disable firewalld
systemctl stop firewalld
```

2. 有时候拉资源非常的慢直接替换成镜像网址的域名

```
下拉镜像：quay.io/coreos/flannel:v0.10.0-s390x
如果拉取较慢，可以改为：quay-mirror.qiniu.com/coreos/flannel:v0.10.0-s390x
或者quay.mirrors.ustc.edu.cn镜像

下拉镜像：gcr.io/google_containers/kube-proxy
可以改为： registry.aliyuncs.com/google_containers/kube-proxy
```