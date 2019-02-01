### 安装helm
1. https://github.com/helm/helm/releases 下载helm（下载的是2.12.3），并且解压，可以直接使用helm命令
2. 查看 https://github.com/helm/helm/blob/master/docs/rbac.md 查看 `Example: Service account with cluster-admin role` 并且制作自己的tiller-rbac的yaml文件
3. cd /vagrant/manifests/helm/ && kubectl apply -f tiller-rbac.yaml 
4. kubectl get sa -n kube-system 查看tiller账号
5. 在缺省配置下， Helm 会利用 "gcr.io/kubernetes-helm/tiller" 镜像在Kubernetes集群上安装配置 Tiller；并且利用 "https://kubernetes-charts.storage.googleapis.com" 作为缺省的 stable repository 的地址。由于在国内可能无法访问 "gcr.io", "storage.googleapis.com" 等域名，微软azure为此提供了镜像站点。（阿里的好久没有更新了。然后就删除了）
```
helm init --service-account tiller --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.12.3 --stable-repo-url http://mirror.azure.cn/kubernetes/charts/
```
1. kubectl get pods -n kube-system 查看 tiller-deploy 是否运行
2. helm version 查看版本
3. helm repo update # 更新仓库
   
#### helm中 release管理   
1. helm install --name mem1 stable/memcached # 新增
2. helm delete mem1 # 删除
3. helm list # 查看
4. helm upgrade
5. helm rollback
6. helm history redis1
7. helm status mem1 # 获取之前的状态信息，比如redis密码的获取等 （重要！）
   
#### helm中 chat 管理
1. helm create
2. helm fetch stable/redis # 获取char的tgz压缩包
3. helm get
4. helm inspect
5. helm package
6. helm verify

### 修改一个redis的value文件，生成自己heml特性
1. 复制  ~/.helm/cache/archive/redis/values.yaml （redis文件夹是自己解压后生成的目录）
2. helm install --name redis1 -f values.yaml stable/redis 自定模板创建镜像

### 获取一个helm清单列表
1. helm fetch stable/redis
2. tar xf redis-5.4.0.tgz 获取里面的清单

### 创建一个helm的清单
1. helm create myapp
2. 修改myapp/里面的属性
3. helm lint myapp/ # 检查语法格式
4. helm package myapp # 如果检查语法没有问题，打包helm
5. ss -tnl 查看本地是否开启8879端口，如果没有开启可以使用 helm serve 开启服务
6. 启用另一个终端 helm search myapp 可以查看到本地myapp
7. 现在就可以部署一个应用了: helm install --name myapp3 local/myapp
8. helm status myapp3 可以再次获取状态
9. helm delete --purge myapp3 彻底删除应用
10. helm list -a 查看资源

### 使用一个新仓库
1. 官方仓库地址：https://hub.kubeapps.com/
2. 如果要添加 Incubator Charts 使用： helm repo add incubator http://mirror.azure.cn/kubernetes/charts-incubator/

### EFK
1. helm fetch stable/elasticsearch
2. tar xf elasticsearch-1.18.0.tgz
3. cd elasticsearch 并且修改value.yaml 文件, 这里关闭了持久化存储，并且把master,client,data的replicas数都改为了1, MINIMUM_MASTER_NODES 改为1，persistence 改为 false, 关闭持久存储。 建议先把镜像拖下来，加快速度
4. kubectl create namespace efk
5. helm install --name els1 --namespace efk -f values.yaml stable/elasticsearch
6. 下载工具类测试是否跑通： kubectl run cirror-$RANDOM --rm -it --image=cirros -- /bin/sh
7. helm status els1 查看访问接口 els1-elasticsearch-client.efk.svc
8. 在cirros终端中：nslookup els1-elasticsearch-client.efk.svc
9. curl els1-elasticsearch-client.efk.svc.cluster.local:9200 可以访问接口是否成功 curl els1-elasticsearch-client.efk.svc.cluster.local:9200/_cat/nodes 查看节点
10. 安装 fluentd-elasticsearch： helm fetch stable/fluentd-elasticsearch, 编辑values.yaml文件，建议更改镜像文件为 gcr.akscn.io/google-containers/fluentd-elasticsearch, 把 elasticsearch.host 改为 els1-elasticsearch-client.efk.svc.cluster.local
如果要收集主节点需要开启 tolerations 下面的所有,允许污点主机。 prometheus 允许监控的话，需要开启 podAnnotations 下的数据。 并且开启 service 下的数据
11. helm install --name flu1 --namespace efk -f values.yaml stable/fluentd-elasticsearch
12. curl els1-elasticsearch-client.efk.svc.cluster.local:9200/_cat/indices
13. helm fetch stable/kibana 修改values, elasticsearch.url 改为 els1-elasticsearch-client.efk.svc.cluster.local:9200, serviec:改为NodePort
14. helm install --name kib1 --namespace efk -f values.yaml stable/kibana