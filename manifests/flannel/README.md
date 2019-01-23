### 修改flannel通讯方式
1. 默认通讯方式是 vxlan 网络叠加，Directrouting 是关闭的。 要开启 网络叠加，Directrouting
2. 在 kube-flannel.yaml 增加 Directrouting：true （2,3只能选择其中一种）
3. 还可以 kube-flannel.yaml 把 "Type": "vxlan" 改为 "Type": "host-gw", 后面没有Directrouting选项，确保集群不能够跨网段，因为不支持跨网段

*** 以下操作最好不要执行，会有风险 ***
1. 如果要演示需要 kubectl delete -f kube-flannel.yaml
2. 然后 kubectl apply -f kube-flannel.yaml

### 抓包工具
1. yum install tcpdump
2. tcpdump -i eth1 -nn icmp
3. ip route show # 列出ip路由表

```
10.244.0.0/24 dev cni0  proto kernel  scope link  src 10.244.0.1 
10.244.1.0/24 via 10.244.1.0 dev flannel.1 onlink 
10.244.2.0/24 via 10.244.2.0 dev flannel.1 onlink 

==================================================================

会直接挂到具体的网卡，而不是在flannel.1 上

```

 