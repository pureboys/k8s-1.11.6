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