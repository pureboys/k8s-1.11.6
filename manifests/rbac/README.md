### apiserver 模拟获取

1. kubectl proxy --port=8080 在master主机上代理
2. curl http://172.20.0.70:6443/apis/apps/v1/namespaces/default/deployments/myapp-deploy/
3. k8s 上的权限可以分为User用户对api访问的权限和内部pods（serviceaccount）对api访问的权限。

### serviceaccount

1. 用来管理pod的权限
2. kubectl create serviceaccount mysa -o yaml --dry-run 不会执行，会列出清单如何去写
3. kubectl get sa
4. kubectl config view 查看配置文件

### 创建一套自己的账户链接apiserver
1. cd /etc/kubernetes/pki
2. (umask 077; openssl genrsa -out oliver.key 2048)
3. openssl req -new -key oliver.key -out oliver.csr -subj "/CN=oliver"
4. openssl x509 -req  -in oliver.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out oliver.crt -days 365 
5. openssl x509 -in oliver.crt -text -noout # 来查看证书
6. kubectl config  set-credentials oliver  --client-certificate=./oliver.crt --client-key=./oliver.key --embed-certs=true
7. kubectl config view # 查看下配置
8. kubectl config set-context oliver@kubernetes --cluster=kubernetes --user=oliver
9. kubectl config view # 查看下配置
10. kubectl config use-context oliver@kubernetes# 切换用户
11. 这里没有设置cluster,只是在本机上做了演示：
kubectl config set-cluster mycluster --kubeconfig=/tmp/test.conf --server="https://11.11.11.111:6443" --certificate-authority=/etc/kubernetes/pki/ca.crt --embed-certs=true
12. kubectl config view --kubeconfig=/tmp/test.conf 进行查看

### 创建role权限
1. kubectl create role pods-reader --verb=get,list,watch --resource=pods --dry-run -o yaml
2. kubectl apply -f role-demo.yaml 
3. kubectl get role
4. kubectl create rolebinding oliver-read-pods --role=pods-reader --user=oliver --dry-run -o yaml
5. kubectl apply -f rolebinding-demo
6. 切换到 oliver 这个账号看下： kubectl config use-context oliver@kubernetes. 分别查看有权限的和没有权限的。 kubectl get pods 与 kubectl get pods -n kube-system

### 定义个clusterrole
1. 需要将之前的用户切换到admin用户，kubectl config use-context kubernetes-admin@kubernetes
2. 自己新建个用户设置ik8s，把oliver@kubernetes分配到这个用户上
   
```
useradd ik8s
cp -rp ~/.kube/ /home/ik8s/
chown -R ik8s.ik8s /home/ik8s/
kubectl config  use-context oliver@kubernetes
kubectl config view
```
3. kubectl create clusterrole cluster-reader --verb=get,list,watch --resource=pods -o yaml --dry-run > clusterrole-demo.yaml
4. kubectl apply -f clusterrole-demo.yaml
5. kubectl get rolebinding # 查看是否有绑定过权限。如果有 kubectl delete rolebinding oliver-read-pods 删除绑定
6. 查看系统上创建的所有clusterrole.  kubectl get clusterrole
7. kubectl  create clusterrolebinding oliver-read-all-pods --clusterrole=cluster-reader --user=oliver --dry-run -o yaml > clusterrolebinding-demo.yaml

### 使用rolebinding 绑定 clusterrole
1. 只对 rolebinding 所在的namespace中有权限效果 
2. 首先删除之前创建的 kubectl delete clusterrolebinding oliver-read-all-pods
3. kubectl create rolebinding oliver-read-pods --clusterrole=cluster-reader --user=oliver --dry-run -o yaml >  rolebinding-clusterrole-demo.yaml
4. kubectl apply -f rolebinding-clusterrole-demo.yaml

### 创建一个在default namespeace中的默认管理员
1. 用rolebinding 去绑定 clusterrole 中的 admin 权限（admin 系统已经给生成好了，直接绑定就行了）
2. kubectl create rolebinding default-ns-admin --clusterrole=admin --user=oliver

