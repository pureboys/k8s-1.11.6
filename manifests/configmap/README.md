### 手动配置 configmap

1. kubectl create configmap nginx-config  --from-literal=nginx_port=80 --from-literal=server_name=myapp.oliver.com 
2. kubectl create configmap nginx-www --from-file=./www.conf 
3. 运行 pod-configmap.yaml，容器中引用的环境变量只有启动时才会生效。 修改 kubectl edit cm nginx-config，不起作用。
4. 运行 pod-configmap-2.yaml 会在容器中生成变量的配置文件。 修改 kubectl edit cm nginx-config。 配置文件里面的值会改变。
5. pod-configmap-3.yaml 为一个真实案例， 如果不想取全部数据。 可以使用 pods.spec.volumes.configMap.items 里面的配置选取部分key值


### 手动配置 secret
1. kubectl create secret generic mysql-root-password --from-literal=password=MyP@ss123
2. kubectl get secret
3. kubectl get secret mysql-root-password -o yaml
```
password: TXlQQHNzMTIz

echo TXlQQHNzMTIz | base64 -d # 可以解密 -_-#

```
4. secret与configmap加载配置方式大致相同
