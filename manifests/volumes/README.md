### nfs 搭建

1. store节点和各Node节点安装nfs  
```
yum -y install nfs-utils
```
2. store节点准备共享目录 
```
mkdir -pv /data/volumes
vim /etc/exports

/data/volumes   11.11.11.0/24(rw,no_root_squash)

systemctl start nfs

ss -tnl (2049端口)
```
3. node节点进行挂载
```
mount -t nfs 11.11.11.114:/data/volumes /mnt

mount 检测一下是否挂载成功

umount /mnt 卸载挂载
```
4. 运行 pod-vol-nfs.yaml 可以绑定


### 搭建 pvc

pvc 和 pv 一一对应， 容器挂载到pvc上。 pv 连接存储节点。

1. 在store节点创建多个目录
    ```
    cd /data/volumes && mkdir v{1,2,3,4,5}
    vim /etc/exports

    /data/volumes/v1   11.11.11.0/24(rw,no_root_squash)
    /data/volumes/v2   11.11.11.0/24(rw,no_root_squash)
    /data/volumes/v3   11.11.11.0/24(rw,no_root_squash)
    /data/volumes/v4   11.11.11.0/24(rw,no_root_squash)
    /data/volumes/v5   11.11.11.0/24(rw,no_root_squash)


    exportfs -arv
    showmount -e

    ```
2. 创建pv:   pv-demo.yaml
3. 查看pv:   kubectl get pv 是否创建成功
4. 创建pvc:  pod-vol-pvc.yaml
5. 查看pv:   kubectl get pv 查看是否被绑定上
6. 查看pvc:  kubectl get pvc