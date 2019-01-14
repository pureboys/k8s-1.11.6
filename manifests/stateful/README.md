### 制作一个stateful-demo


1、稳定且惟一的网络标识符；
2、稳定且持久的存储；
3、有序、平滑地部署和扩展；
4、有序、平滑地删除和终止；
5、有序的滚动更新；

三个组件：headless service、StatefulSet、volumeClaimTemplate

1. 前提：之前在 volumes 文件夹中我们创建了nfs目录，以及创建了5个pv。参见 volume目录中 pv-demo.yaml
2. stateful-demo.yaml 中的 volumeClaimTemplates 完成两个功能， 1.为每个pod完成生成一个volume 2.动态的生成一个pvc并连接到pv上， 等于volume目录中 pod-vol-pvc.yaml
3. kubectl get sts 查看 stateful 状态