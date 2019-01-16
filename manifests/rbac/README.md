### apiserver 模拟获取

1. kubectl proxy --port=8080 在master主机上代理
2. curl http://172.20.0.70:6443/apis/apps/v1/namespaces/default/deployments/myapp-deploy/