### 配置calico (Calico for policy and flannel for networking)
1. 官方文档地址 https://docs.projectcalico.org/v3.4/getting-started/kubernetes/installation/flannel
2. 下载镜像，并且安装在每个节点上
```
  docker pull quay-mirror.qiniu.com/calico/cni:v3.4.0
  docker pull quay-mirror.qiniu.com/calico/node:v3.4.0
  docker pull quay-mirror.qiniu.com/coreos/flannel:v0.9.1

  docker tag quay-mirror.qiniu.com/calico/cni:v3.4.0 quay.io/calico/cni:v3.4.0
  docker tag quay-mirror.qiniu.com/calico/node:v3.4.0 quay.io/calico/node:v3.4.0
  docker tag quay-mirror.qiniu.com/coreos/flannel:v0.9.1 quay.io/coreos/flannel:v0.9.1

  docker save quay.io/calico/cni:v3.4.0 quay.io/calico/node:v3.4.0 quay.io/coreos/flannel:v0.9.1  -o calico.tar.gz

  scp ...

  wget https://docs.projectcalico.org/v3.4/getting-started/kubernetes/installation/hosted/canal/canal.yaml
 
  ###
  修改 canal.yaml 文件
  1. 把 image: quay.io/coreos/flannel:v0.9.1 可以换成和之前flannel一样的版本，quay.io/coreos/flannel:v0.10.0-amd64
  2.  command: [ "/opt/bin/flanneld", "--ip-masq", "--kube-subnet-mgr"] 后面 加上 "--iface=eth1"，特定的网卡和flannel方法一样

```

#### 案例1
1. kubectl explain networkpolicy
2. kubectl create namespace dev
3. kubectl create namespace prod
4. kubectl apply -f ingress-def.yaml -n dev
5. kubectl get netpol -n dev
6. kubectl apply -f pod-a.yaml -n dev
7.  kubectl get pods -n dev -o wide
8. curl 10.244.2.2（pod的IP地址） # 应该是访问不到的
9. kubectl apply -f pod-a.yaml -n prod # prod上没有定义规则
10. kubectl get pods -n prod -o wide
11. curl 10.244.1.2 （pod的IP地址） # 可以访问

#### 案例2 （制定入站规则）
1. kubectl label pods pod1 app=myapp -n dev
2. kubectl apply -f allow-netpol-demo.yaml -n dev
3. kubectl get netpol -n dev
4. curl 10.244.2.2 (dev中的pod) 是可以访问的 但是 10.244.2.2：443 会被阻塞掉。而不是返回错误。

#### 案例3 （制定出站规则）
1. kubectl apply -f egress-def.yaml -n prod
2. kubectl get pods -n prod (如果没有，自己建一个)
3. kubectl exec pod1  -it -n prod -- /bin/sh 进入交互式
4. 找一个其他pod的ip地址，去ping下， 应该是ping不通的。