### 部署一个metrics server 用于系统核心的监控，可以是用top
1. 下载 https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/metrics-server 中的yaml
2. metrics-server-deployment.yaml 中的 image 替换

```
k8s.gcr.io/metrics-server-amd64:v0.3.1 替换为 registry.cn-hangzhou.aliyuncs.com/mirror_googlecontainers/metrics-server-amd64:v0.3.1
# 这个没有找到阿里的原镜像。所以用的用户镜像

k8s.gcr.io/addon-resizer:1.8.4 替换为 registry.aliyuncs.com/google_containers/addon-resizer:1.8.4

```

4. 注意：metrics-server-deployment.yaml 不能直接运行 有个坑。addon-resizer 组件里面要修改参数
```

  command:
  - /pod_nanny
  - --config-dir=/etc/config
  - --cpu={{ base_metrics_server_cpu }}
  - --extra-cpu=0.5m
  - --memory={{ base_metrics_server_memory }}
  - --extra-memory={{ metrics_server_memory_per_node }}Mi
  - --threshold=5
  - --deployment=metrics-server-v0.3.1
  - --container=metrics-server
  - --poll-period=300000
  - --estimator=exponential
  # Specifies the smallest cluster (defined in number of nodes)
  # resources will be scaled to.
  - --minClusterSize={{ metrics_server_min_cluster_size }}


  containers:
  - name: metrics-server
    image: k8s.gcr.io/metrics-server-amd64:v0.3.1
    command:
    - /metrics-server
    - --metric-resolution=30s
    # These are needed for GKE, which doesn't support secure communication yet.
    # Remove these lines for non-GKE clusters, and when GKE supports token-based auth.
    - --kubelet-port=10255
    - --deprecated-kubelet-completely-insecure=true

########### 修改为 #############

  command:
  - /pod_nanny
  - --config-dir=/etc/config
  - --cpu=80m
  - --extra-cpu=0.5m
  - --memory=80Mi
  - --extra-memory=8Mi
  - --threshold=5
  - --deployment=metrics-server-v0.3.1
  - --container=metrics-server
  - --poll-period=300000
  - --estimator=exponential
  # Specifies the smallest cluster (defined in number of nodes)
  # resources will be scaled to.
  #- --minClusterSize={{ metrics_server_min_cluster_size }}


  containers:
  - name: metrics-server
    image: registry.cn-hangzhou.aliyuncs.com/mirror_googlecontainers/metrics-server-amd64:v0.3.1
    command:
    - /metrics-server
    - --metric-resolution=30s
    - --kubelet-insecure-tls
    - --kubelet-preferred-address-types=InternalIP,Hostname,InternalDNS,ExternalDNS,ExternalIP
    # These are needed for GKE, which doesn't support secure communication yet.
    # Remove these lines for non-GKE clusters, and when GKE supports token-based auth.
    #- --kubelet-port=10255
    #- --deprecated-kubelet-completely-insecure=true


```
5. resource-reader.yaml 中修改， 这也是一个坑 之前没加各种出错
```
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - nodes
  - nodes/stats    #新增这一行
  - namespaces
  verbs:
  - get
  - list
  - watch
```

6. kubectl apply -f .
7. kubectl get svc -n kube-system
8. kubectl get pods -n kube-system
9. 查看log日志 kubectl logs xxxxxxx(容器名) -c metrics-server -n kube-system。 不出错为没有问题
10. kubectl api-versions 中出现 metrics.k8s.io/v1beta1
11. kubectl proxy --port=8080 开启后 再开启一个终端运行 curl http://localhost:8080/apis/metrics.k8s.io/v1beta1/nodes 或者 curl http://localhost:8080/apis/metrics.k8s.io/v1beta1/pods
12. kubectl top nodes 或者 kubectl top pods 收集数据

#### 部署中的坑
```
Get https://k8s-master:10250/stats/summary/: dial tcp: lookup k8s-master on 10.96.0.10:53: no such host
```
+ 提示 无法解析节点的主机名，是metrics-server这个容器不能通过CoreDNS 10.96.0.10:53 解析各Node的主机名，metrics-server连节点时默认是连接节点的主机名，需要加个参数，让它连接节点的IP：
“--kubelet-preferred-address-types=InternalIP”

+ 因为10250是https端口，连接它时需要提供证书，所以加上--kubelet-insecure-tls，表示不验证客户端证书，此前的版本中使用--source=这个参数来指定不验证客户端证书。(1.11版本， --source=kubernetes.summary_api:https://kubernetes.default?kubeletHttps=true&kubeletPort=10250&insecure=true)
