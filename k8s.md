# K8s 1.11.6 配置

### Master/Node:

1. wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/
2. vim /etc/yum.repos.d/kubernetes.repo

```
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
enabled=1
```

3. yum install --setopt=obsoletes=0 docker-ce-17.03.1.ce-1.el7.centos
4. yum install kubelet-1.11.6  kubeadm-1.11.6  kubectl-1.11.6
5. systemctl enable docker kubelet
6. systemctl start docker
7. /etc/sysconfig/kubelet

```
KUBELET_EXTRA_ARGS="--fail-swap-on=false"
```
8. 确保为1
```
[root@k8s-manager labs]# cat /proc/sys/net/bridge/bridge-nf-call-iptables
1
[root@k8s-manager labs]# cat /proc/sys/net/bridge/bridge-nf-call-ip6tables
1
```

9. 下载镜像
./pull_k8s_images.sh

10. 
```
 kubeadm init --kubernetes-version=v1.11.6  --apiserver-advertise-address=192.168.205.50 --pod-network-cidr=10.244.0.0/16 servicecidr=10.96.0.0/12 --ignore-preflight-errors=Swap
```
11.
```
 mkdir -p $HOME/.kube
 sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

12. 添加加flannel网络附件
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kubeflannel.yml
```

13. 验正master节点已经就绪
```
# kubectl get nodes
```


### Nodes:


1. wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/
2. vim /etc/yum.repos.d/kubernetes.repo

```
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
enabled=1
```

3. yum install --setopt=obsoletes=0 docker-ce-17.03.1.ce-1.el7.centos
4. yum install kubelet-1.11.6  kubeadm-1.11.6  kubectl-1.11.6
5. systemctl enable docker kubelet
6. systemctl start docker
7. /etc/sysconfig/kubelet

```
KUBELET_EXTRA_ARGS="--fail-swap-on=false"
```
8. 确保为1
```
[root@k8s-manager labs]# cat /proc/sys/net/bridge/bridge-nf-call-iptables
1
[root@k8s-manager labs]# cat /proc/sys/net/bridge/bridge-nf-call-ip6tables
1
```

9. 下载镜像
./pull_k8s_images.sh

10. 加入网络
```
 kubeadm join 192.168.205.50:6443 --token syk7vu.cjt65h4kpuobtgvf --discovery-token-ca-cert-hash sha256:82addd9d45a114dbbf00721e3e72990cb2bc0f750898ee28691120aabe3d14dc --ignore-preflight-errors=Swap
``` 