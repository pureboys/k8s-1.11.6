### 安装helm
1. https://github.com/helm/helm/releases 下载helm（下载的是2.12.3），并且解压，可以直接使用helm命令
2. 查看 https://github.com/helm/helm/blob/master/docs/rbac.md 查看 `Example: Service account with cluster-admin role` 并且制作自己的tiller-rbac的yaml文件
3. cd /vagrant/manifests/helm/ && kubectl apply -f tiller-rbac.yaml 
4. kubectl get sa -n kube-system 查看tiller账号
5. 在缺省配置下， Helm 会利用 "gcr.io/kubernetes-helm/tiller" 镜像在Kubernetes集群上安装配置 Tiller；并且利用 "https://kubernetes-charts.storage.googleapis.com" 作为缺省的 stable repository 的地址。由于在国内可能无法访问 "gcr.io", "storage.googleapis.com" 等域名，阿里云容器服务为此提供了镜像站点。
请执行如下命令利用阿里云的镜像来配置 Helm（修改对应的版本）

```
helm init --service-account tiller --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.12.3 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts

```
6. kubectl get pods -n kube-system 查看 tiller-deploy 是否运行
7. helm version 查看版本
8. helm repo update # 更新仓库
   
#### helm中 release管理   
1. helm install --name mem1 stable/memcached # 新增
2. helm delete mem1 # 删除
3. helm list # 查看
4. helm upgrade/rollback
5. helm histroy
   
#### helm中 chat 管理
1. helm create
2. helm get/fetch
3. helm inspect
4. helm package
5. helm verify