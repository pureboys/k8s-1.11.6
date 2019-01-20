### 搭建dashboard
1. 通过 registry.aliyuncs.com/google_containers/kubernetes-dashboard-amd64:v1.10.1 把镜像拉下来
2. docker tag registry.aliyuncs.com/google_containers/kubernetes-dashboard-amd64:v1.10.1 k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1
3. scp 子节点中
4. kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
5. kubectl get pods -n kube-system 查看是否运行起来
6. kubectl patch svc kubernetes-dashboard -p '{"spec":{"type":"NodePort"}}' -n kube-system
7. kubectl get svc -n kube-system 查看外部访问端口


### token生成总管理员权限

1. kubectl create serviceaccount dashboard-admin -n kube-system
2. kubectl get serviceaccount -n kube-system
3. kubectl create clusterrolebinding dashboard-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
4. kubectl get secret -n kube-system 查找到 dashboard-admin-token-* 的secret (这个是主机自动生成的)
5. kubectl describe secret dashboard-admin-token-xxtqv -n kube-system 可以查看到token，就是令牌认证,此用户是具有超级管理员权限的。


### token生成命名空间管理员权限
1. kubectl create serviceaccount def-ns-admin -n default
2. kubectl create rolebinding def-ns-admin --clusterrole=admin --serviceaccount=default:def-ns-admin
3. kubectl get secret 能够发现（def-ns-admin-token-*)
4. kubectl describe secret def-ns-admin-token-s5xg9 可以查看到token，此用户是只具有default命名空间的权限


### 根据Kubeconfig获取权限

1. kubectl config set-cluster kubernetes  --certificate-authority=./ca.crt --server=https://11.11.11.111:6443 --embed-certs=true --kubeconfig=/vagrant/labs/def-ns-admin.conf
2. kubectl config view --kubeconfig=/vagrant/labs/def-ns-admin.conf
3. kubectl get secret
4. kubectl describe secret def-ns-admin-token-s5xg9 可以查看到token，此用户是只具有default命名空间的权限
5.  kubectl config set-credentials def-ns-admin --token=xxx --kubeconfig=/vagrant/labs/def-ns-admin.conf
6.  kubectl config view --kubeconfig=/vagrant/labs/def-ns-admin.conf
7.  kubectl config set-context def-ns-admin@kubernetes --cluster=kubernetes --user=def-ns-admin --kubeconfig=/vagrant/labs/def-ns-admin.conf
8.  kubectl config view --kubeconfig=/vagrant/labs/def-ns-admin.conf
9.  kubectl config use-context def-ns-admin@kubernetes --kubeconfig=/vagrant/labs/def-ns-admin.conf
10. kubectl config view --kubeconfig=/vagrant/labs/def-ns-admin.conf


