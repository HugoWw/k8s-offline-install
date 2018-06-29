# Kubernetes 1.10.0 离线部署
————————————————————————————————————————————————————

DEPRECATED: This script is used to install Kubernetes 1.10.0 without of network.
该脚本用于在没网络环境下离线部署安装kubernetes 1.10.0。脚本一共有8个模块组成分别是：

1)、服务器初始化
2)、依赖包安装
3)、docker安装
4)、导入镜像
5)、kubernetes组建安装
6)、kubernetes主节点初始化
7)、dashboard安装
8)、heapster监控安装


#### 适用环境：
————————————————————————————————————————————————————

centos 7.2(1511)/centos 7.3(1611) 


#### 使用方法：
————————————————————————————————————————————————————

安装集群至少需要两个节点，将安装包下载到各个节点上，解压安装包进入安装包执行install脚本，
按照序列号选项一步步进行安装，若以跳序安装则不能正常完成安装。

setp 1:
Master安装：(Masetr的机器上安装1-5功能模块)
```
[root@k8s-master1 k8s_1.10.0_file]# ./install.sh
1) initial_host       4) load_images        7) install_dashboard
2) install_dep        5) install_k8s        8) install_heapster
3) install_docker     6) k8s_mster_initial
Enter a Number:[1-5]
```
注意1：install脚本中的6-8的模块需要在Node节点上完成1-5模块部署后并且加入集群后才能正常安装，
否则由于集群还没形成不能正常部署dashboard和监控服务。

Node安装：(Node的机器上安装1-5功能模块)
```
[root@k8s-node1 k8s_1.10.0_file]# ./install.sh
1) initial_host       4) load_images        7) install_dashboard
2) install_dep        5) install_k8s        8) install_heapster
3) install_docker     6) k8s_mster_initial
Enter a Number:[1-5]
```
setp 2：

Master和Node的/etc/hosts文件中添加：
Master的IP  刚刚设置的Master的主机名 
Node的IP    刚刚设置的Node的主机名 


setp 3:
Master安装：(Master的机器进行初始化master节点)
```
[root@k8s-master1 k8s_1.10.0_file]# ./install.sh
1) initial_host       4) load_images        7) install_dashboard
2) install_dep        5) install_k8s        8) install_heapster
3) install_docker     6) k8s_mster_initial
Enter a Number:[6]
[root@k8s-master1 k8s_1.10.0_file]#source  /root/.bash_profile
```

Node节点：(node节点加入集群)：
```
[root@k8s-node1 k8s_1.10.0_file]#kubeadm join 10.10.8.160:6443 --token yonsi5.jtk59pq8s5txqpat \
--discovery-token-ca-cert-hash sha256:ebf22d0e20fd56e5648a5922b39e9792d6022b101761314ec4bce3d1cb9f95a3
```
注意2：Master上完成6的模块后会将集群口令保存在本地文件cluster_token.txt中,该口令是用于数据节点
加入集群中认证使用，dashboard口令会保存在本地文件dashboard_token.txt中，Node节点上完成1-5模块
部署后执行Master节点上的提示信息进行加入集群。

#### 相关的API端口和UI端口：
————————————————————————————————————————————————————
   http://masterip:8080/api/v1/namespaces/kube-system/services/kubernetes-dashboard/

2、influxdb的api:
   http://masterip:30086/query?q="sql "

3、influxdb的UI界面：
   http://masterip:30083

4、cAdvisor的UI界面：
   http://nodeip:4194

5、dashboard：
   https://masterip:32666

#### kubernetes API开启非安全端口8080：
————————————————————————————————————————————————————
在集群安装完成后编辑如下文件进行修改api配置：
```
vi  /etc/kubernetes/manifests/kube-apiserver.yaml 修改如下：
- --insecure-port=8080
- --insecure-bind-address=0.0.0.0

$ systemctl daemon-reload 
$ systemctl restart kubelet
```
