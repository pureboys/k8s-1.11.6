### 高级调度方式

注意：使用的时候最好清除所有pod 防止实验出现问题

#### nodeSelector
1.  kubectl label nodes k8s-node2 disktype=harddisk
2.  kubectl get nodes --show-labels
3.  kubectl apply -f schdule-pod-demo.yaml
4.  kubectl get pods -o wide

#### nodeAffinity (节点亲和)
1. kubectl apply -f schdule-pod-nodeaffinity-demo.yaml # 硬亲和节点，没有挂起
2. kubectl apply -f schdule-pod-nodeaffinity-demo-2.yaml # 软亲和节点，尽量满足，没有也会在其他节点跑起来

#### podAffinity (pod亲和)
1. kubectl apply -f schdule-pod-podaffinity-demo.yaml
2. kubectl get pods -o wide
3. 删掉重建也是在同一个node上

#### podAntiAffinity (pod反亲和)
1. kubectl label nodes k8s-node1 zone=foo
2. kubectl label nodes k8s-node2 zone=foo
3. kubectl apply -f schdule-pod-podantiaffinity-demo.yaml
4. kubectl get pods -o wide (node2 被pending)

#### 管理节点的污点
1. kubectl taint node k8s-node1 node-type=production:NoSchedule # 打上污点
2. kubectl apply -f ../deploy-demo.yaml  #所有的pod会落到k8s-node2 因为不能容忍污点
3. kubectl taint node  k8s-node2 node-type=dev:NoExecute # 现在都不能为容忍了
4. kubectl apply -f taint-demo.yaml # 建一个可以容忍污点的pod
5. kubectl get pods -o wide # 查看一下
6. 删除污点: kubectl taint node k8s-node1 node-type-
7. kubectl taint node k8s-node2 node-type-

##### 注解1
```
- key: "node-type"
  perator: "Exists"
  value: ""
  effect: "NoSchedule"
```
+ 如果 perator 为 Exists， 只要存在污点的key为node-type就能容忍匹配上。 然后根据effect值判断。 并且节点上的effect要与pod中的effect完全匹配， effect 为空则表示匹配所有节点的effect

##### 注解2
```
- key: "node-type"
  perator: "Equal"
  value: "production"
  effect: "NoSchedule"
```
+ 如果 perator 为 Equal，则要容忍匹配上污点key为node-type，值为production的节点。然后根据effect值判断。 并且节点上的effect要与pod中的effect完全匹配， effect 为空则表示匹配所有节点的effect
