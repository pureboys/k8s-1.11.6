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