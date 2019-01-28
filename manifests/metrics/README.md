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


### HPA 水平Pod横向扩展1
1. 手动创建一个 deployment:
kubectl run myapp --image=ikubernetes/myapp:v1 --replicas=1 --requests='cpu=50m,memory=256Mi' --limits='cpu=50m,memory=256Mi' --labels='app=myapp' --expose --port=80
2. kubectl get pods
3. kubectl autoscale deployment myapp --min=1 --max=8 --cpu-percent=60
4. kubectl get hpa
5. kubectl patch svc myapp -p '{"spec":{"type":"NodePort"}}'
6. kubectl get svc
7. 本地发起压测  ab -c 100 -n 500000 http://11.11.11.111:31199/index.html
8. kubectl describe hpa 可以看效果是否pod横向扩展

### HPA 水平Pod横向扩展2
1. kubectl delete hpa myapp
2. kubectl apply -f hpa-v2-demo.yaml
3. kubectl get hpa
4. 本地发起压测  ab -c 100 -n 500000 http://11.11.11.111:31199/index.html
5. kubectl describe hpa

### HPA 水平Pod横向扩展3（未测试）
1. 创建一个 https://hub.docker.com/r/ikubernetes/metrics-app 镜像的deployment
2. kubectl apply -f hpa-v2-custom.yaml
3. 发起压力测试后，查看hpa的情况