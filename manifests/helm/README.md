### 安装helm
1. https://github.com/helm/helm/releases 下载helm（下载的是2.12.3），并且解压，可以直接使用helm命令
2. 查看 https://github.com/helm/helm/blob/master/docs/rbac.md 查看 `Example: Service account with cluster-admin role` 并且制作自己的tiller-rbac的yaml文件
3. cd /vagrant/manifests/helm/ && kubectl apply -f tiller-rbac.yaml 
4. kubectl get sa -n kube-system 查看tiller账号
5. 在缺省配置下， Helm 会利用 "gcr.io/kubernetes-helm/tiller" 镜像在Kubernetes集群上安装配置 Tiller；并且利用 "https://kubernetes-charts.storage.googleapis.com" 作为缺省的 stable repository 的地址。由于在国内可能无法访问 "gcr.io", "storage.googleapis.com" 等域名，阿里云容器服务为此提供了镜像站点。
请执行如下命令利用阿里云的镜像来配置 Helm（修改对应的版本）（不过阿里镜像好像好久不更新了，参见 https://github.com/BurdenBear/kube-charts-mirror）
```
helm init --service-account tiller --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.12.3 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
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