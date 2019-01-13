### ingres-nginx 安装配置

1. 获取并且安装ingres-nginx-controller等必要的配置
```
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml
kubectl apply -f mandatory.yaml

# 查看下是否运行
kubectl get pods -n ingress-nginx -o wide

```

2. 如果是自己搭建的服务还需要在 ingres-nginx-controller 前面布置外部可以访问的节点IP
```
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/baremetal/service-nodeport.yaml
kubectl apply -f service-nodeport.yaml

# 查看下是否运行
kubectl get svc -n ingress-nginx

```

3. 需要构建自己的Deployment和Service。

4. 构建ingress可以使外部通过域名访问内部pod (deploy-demo.yaml && ingress-myapp || ingress-tomcat.yaml && tomcat-deploy)


### 自建证书

1. openssl genrsa -out tls.key 2048

2. openssl req -new -x509 -key tls.key -out tls.crt -subj /C=CN/ST=Beijing/L=Beijing/O=DevOps/CN=tomcat.oliver.com  （域名填写自己的域名）

3. kubectl create secret tls tomcat-ingress-secret --cert=tls.crt --key=tls.key

4. kubectl get secret # 查看密钥

5. 修改ingress配置文件 （ingress-tomcat-tls.yaml） 支持ssl格式