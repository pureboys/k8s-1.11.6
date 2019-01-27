### 增加资源限制
1. kubectl  apply -f metrics-pod-demo.yaml
2. kubectl exec metrics-pod-demo top 查看效果

### ~~配置influxdb~~
1. wget https://raw.githubusercontent.com/kubernetes-retired/heapster/master/deploy/kube-config/influxdb/influxdb.yaml
2. 修改镜像地址为阿里源镜像地址 
3. apiVersion 改为 apps/v1 后 spec 增加
```
spec:
  replicas: 1
  # 新增部分
  selector:
    matchLabels:
      task: monitoring
      k8s-app: influxdb
``` 
4. kubectl apply -f influxdb.yaml
5. kubectl get svc -n kube-system # 查看service是否启动
6. kubectl get pods -n kube-system # 查看Pod是否启动

### ~~部署heapster （已经被k8s放弃）,可以用Prometheus代替~~
1. wget https://raw.githubusercontent.com/kubernetes-retired/heapster/master/deploy/kube-config/rbac/heapster-rbac.yaml
2. kubectl apply -f heapster-rbac.yaml
3. wget https://raw.githubusercontent.com/kubernetes-retired/heapster/master/deploy/kube-config/influxdb/heapster.yaml
4. 修改镜像地址为阿里源镜像地址
5. 第二个组件的apiVersion 改为 apps/v1 后 spec 增加
```
# 第二个组件
---
apiVersion: apps/v1 # -> 修改成apps/v1
kind: Deployment
metadata:
  name: heapster
  namespace: kube-system
spec:
  replicas: 1
  # 新增selector选项
  selector:
    matchLabels:
      task: monitoring
      k8s-app: heapster
```  

1. kubectl apply -f heapster.yaml
2. kubectl get svc -n kube-system 查看heapster

### ~~配置安装 grafana~~
1. wget https://raw.githubusercontent.com/kubernetes-retired/heapster/master/deploy/kube-config/influxdb/grafana.yaml
2. 修改镜像地址为阿里源镜像地址
3. 第一个组件的apiVersion 改为 apps/v1 后 spec 增加
```
spec:
  replicas: 1
  # 新增selector选项
  selector:
    matchLabels:
      task: monitoring
      k8s-app: grafana
```
4. 第二个组件 spec增加 type: Nodeport
```
  ports:
  - port: 80
    targetPort: 3000
  selector:
    k8s-app: grafana
  ### 新增type: NodePort  
  type: NodePort
```
5.  kubectl  apply -f grafana.yaml 
6.  kubectl get svc -n kube-system
7.  外部直接访问 不需要https
