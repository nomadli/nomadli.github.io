---
layout:         post
title:          kubernets
subtitle:       kubernets
date:           2018-05-03 13:13:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
catalog:        true
tags:
        - other
---

![](../img/kubernetes_01.svg)

# 概念
- Node 运行着kubelet(angent)、容器(docker,rkt)的机器,

        k8 --register-node true node主动注册、false手动维护node
        HostName        可用通过 --hostname-override参数覆盖
        ExternalIP      被集群外部的IP
        InternalIP      集群内的路由节点IP
        OutOfDisk       True 资源以及耗尽,不能再新增pod
        Ready           True 健康可用添加pod
                        False 不健康
                        unknown 40秒没报告
        MemoryPressure  True 内存过低
        DiskPressure    True 磁盘容易过低
        Capacity        cpu、内存、最大可用建pod数
        Info            内核版本、k8版本、docker版本等
        
- 对象 是一个功能集如一个服务和nginx两个容器的集合,对象有多种类型:
        
        deployment  对外服务(long-running)
        Job         批处理(batch)
        DaemonSet   节点服务(node-daemon)日志、监控
        PetSet      有状态应用(stateful application)
        
- Pod 一个Node上的某个业务的工作集, 代表共享存储、镜像版本、集群ip，即代表应用的逻辑主机,pod中的容器共享ip及端口。一个pod总是在一个Node上。
- Service代表一组某个业务的pod通过lable来访问, 用于服务发现、负载均衡,四种类型:

            ClusterIP 集群对外暴露一个IP
            NodePort  直接访问每个Node的IP及端口自动路由到ClusterIP
            LoadBalancer 使用外部负载均衡器
            Externalname 类似域名DNS需要1.7版本以上
- Volume 存储作用范围是是pod,
        
        emptyDir                生命周期与pod相同
        hostPath                挂载Node的文件系统到pod
        gcePersistentDisk       挂载GCE上永久磁盘到容器
        awsElasticBlockStore    挂载AWS上EBS盘到容器
        NFS(Network File System)挂载网络磁盘
        iSCSI                   挂载现有iscsi磁盘
        Flocker                 开源容器集群数据卷管理器
        Glusterfs               挂载Glusterfs开源网络文件系统
        Rados Block Device      挂载RBD磁盘系统
        cephfs                  挂载cephfs文件系统
        gitRepo                 将git代码下拉到容器路径
        secret                  将密钥等存储系统挂载到容器
        Persistent Volume Claim 挂载持久存储卷
        downwardAPI             通过环境变量设置pod信息
        projected               将多个Volume映射到一个目录
        FlexVolume              测试阶段
        AzureFileVolume         加载Azure文件卷
        AzureDiskVolume         加载Azure虚拟机磁盘
        vsphereVolume           加载vSphere VMDK
        Quobyte                 加载Quobyt存储
        PortworxVolume          将服务器磁盘聚合为高可用存储
        ScaleIO                 基于软件的可扩展共享块网络存储
        StorageOS               类似HDFS磁盘系统
        Local                   测试阶段
        Using subPath           指定目录作为卷
        Resources
            
# 核心组件
- etcd 集群状态表
- apiserver 与kubernets交互的接口如:认证、授权、控制、发现
- controller manager 维护集群状态如:故障检测、自动扩展、滚动更新、回滚等
- Node控制器

            可用区域 一般为一个机房或云提供商
            --node-eviction-rate 每秒最多删除多少不健康pod
            --unhealthy-zone-threshold 不健康比率大于就会降低删除速率
            --large-cluster-size-threshold Nod数小于则停止删除
            --secondary-node-eviction-rate 降低的删除速率
            
- Replication控制器
- Endpoints控制器
- Route 控制器
- Volume 控制器
- scheduler 资源调度,根据策略调度pod到相应的机器上
- kubelet Node代理,维护容器负责Volume(CVI)、网络(CNI)管理等
- Container runtime 负责镜像管理、容器运行停止等
- kube-proxy 负责为service提供cluster内部服务器发现、负载均衡
- Service 控制器
- Service Account 控制器
- Token 控制器
- cloud controller manager 云控制器
- kube-ui
- Cluster-level Logging
- Lables Selectors 标签选择器

# 插件
- kube-dns 为集群提供DNS服务
- Ingress Controller为服务提供外网入口
- Heapster 提供资源监控
- Dashboard 提供GUI
- Federation 提供可跨可用区的集群
- Fluentd-elasticsearch 集群日志

# 架构
- 类操作系统分层架构
- Kubernetes内核提供API及插件接口
- 应用层 部署、路由、dns解析等
- 管理层 系统度量、自动化、策略管理
- 接口层 kubectl ci工具、sdk等
- 外部插件

# API
- api 通过json来调用
- 对象有三个属性:元数据(metadata)、规范(spec)、状态(status)

        Spec    描述对象所需要的状态
        Status  描述对象的实际状态.
        
- 元数据标识API对象,至少包含:namespace、name、uid,还有lables等
- 复制控制器(Replication Controller,RC) 监控pod数量是否与指定数目相同,少了就启动新的pod,多了就杀死多余的pod。只适用deployment
- 副本集(Replica Set,RS)替代复制控制器,可以适用所有业务
- 集群联邦(Federation) 跨机房跨运营商调度
- 密钥对象(Secret)类似keygen
- 名字空间(Namespace)提供虚拟隔离
- 用户帐户(User Account)人的账户
- 服务帐户(Service Account)pod的账户关联到一个名字空间
- 访问授权RBAC 添加了角色的功能

# kubectl 命令
1.  kubectl run deployment—name --image=xx/xxx:v1 --port=80 创建一个deployment。
2.  kubectl crate -f x/x/x.yaml --record

        apiVersion: apps/v1beta1      必填
        kind: Deployment              必填
        metadata:                     必填
            name: nginx-deployment    必填
            UID:                      自动生成的
            Namespace:nginx-xxx       可选
            "labels": {               可选、可搜索过滤
                "key1":"value1",
                "key2":"value2"
            }
            "annotations":{           可选择类似备注
                "key1":"value1"
            }
        spec:                         必填
            replicas: 3
            template:
                metadata:
                    labels:
                        app: nginx
                spec:
                    containers:
                    - name: nginx
                    image: nginx:1.7.9
                    ports:
                    - containerPort: 80
    
        apiVersion: v1
        kind: Namespace
        metadata:
            name: new-namespace
            
3.  kubectl create namespace name 
4.  kubectl delete namespaces name

        删除一个namespace会删除所有属于该namespace的资源
        default和kube-system命名空间不可删除
        PersistentVolumes是不属于任何namespace的
        PersistentVolumeClaim是属于某个特定namespace的
        Events是否属于namespace取决于产生events的对象
        
5.  kubectl --namespace=<name> ... 零时namespace
6.  kubectl get xx(deployments) 显示资源(如deployments)
7.  kubectl describe xx 详细信息 
8.  kubectl logs 打印pod中容器日志
9.  kubectl exec 在pod容器中执行命令 

# 安装
- 需要所有机器同步时间 yum install -y ntpdate && ntpdate time.windows.com && date
    - centos stream yum install -y chrony
    - vi /etc/chrony.conf  pool time.windows.com iburst
    - systemctl restart chronyd
    - 修改时区 cp -a /usr/share/zoneinfo/Etc/GMT-8 /etc/localtime
    - 修改时区 timedatectl set-timezone Asia/Shanghai
    - 将当前时间写入硬件 timedatectl set-local-rtc 0
- 设置主机名称 hostnamectl set-hostname name
- 内核设置可选
```shell
#vm机安装检测mac地址、/sys/class/dmi/id/product_uuid、host_name 都要不同

#https://github.com/cri-o/cri-o/blob/main/install.md#readme
#OS=CentOS_9_Stream|CentOS_8|CentOS_8_Stream|CentOS_7  VERSION=1.18.3 ...
curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable.repo https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/devel:kubic:libcontainers:stable.repo
curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo
yum clean all
yum makecache
yum install -y cri-o #/var/run/crio/crio.sock
#可选
yum install containernetworking-plugins
#源码安装 启用btrfs文件系统需要安装btrfs-progs-devel, go>=1.18
yum install -y containers-common device-mapper-devel git glib2-devel glibc-devel glibc-static go gpgme-devel libassuan-devel libgpg-error-devel libseccomp-devel libselinux-devel pkgconfig make runc
#centos8以上 源码安装先执行
subscription-manager repos --enable=rhel-8-for-x86_64-baseos-rpms
subscription-manager repos --enable=rhel-8-for-x86_64-appstream-rpms
subscription-manager repos --enable=codeready-builder-for-rhel-8-x86_64-rpms
yum install -y containers-common device-mapper-devel git make glib2-devel glibc-devel glibc-static go runc
go get github.com/cpuguy83/go-md2man
yum install -y libassuan libassuan-devel libgpg-error libseccomp libselinux pkgconf-pkg-config gpgme-devel

git clone git@github.com:cri-o/cri-o
cd cri-o
make
make install
#/etc/containers/registries.conf /etc/crio/crio.conf.d
make install.config
#/etc/crio/crio.conf
[crio.runtime]
conmon_cgroup = "pod"
cgroup_manager = "systemd" #"cgroupfs"
[crio.image]
pause_image="registry.k8s.io/pause:3.6"
#systemd
make install.systemd
systemctl daemon-reload
systemctl enable crio
systemctl start crio

cat > /etc/modules-load.d/k8s.conf << EOF
overlay
br_netfilter
EOF
modprobe overlay
modprobe br_netfilter
#验证
lsmod | grep br_netfilter

cat > /etc/sysctl.d/kubernetes.conf <<EOF 
#开启网桥模式
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1 
net.ipv4.ip_forward=1
vm.swappiness=0 # 禁止使用 swap 空间，kubelet需要

#下面不是必须
net.ipv4.tcp_tw_recycle=0 
vm.overcommit_memory=1 # 不检查物理内存是否够用 
vm.panic_on_oom=0 # 开启 OOM 
fs.inotify.max_user_instances=8192 #每用户可以内存中创建文件node实例的上限
fs.inotify.max_user_watches=1048576 #每用户可监控文件夹上限
fs.inotify.max_queued_events= #每用户可以等待事件上限
fs.file-max=52706963 #同时打开文件数量
fs.nr_open=52706963 #单进程最大打开文件数
net.ipv6.conf.all.disable_ipv6=1 #禁用ipv6
net.netfilter.nf_conntrack_max=2310720 #最大追踪链接数
EOF
#立即生效
sysctl --system
#或立即加载修改的配置
sysctl -p /etc/sysctl.d/kubernetes.conf
#验证
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables ...

#多网卡且默认网卡无法到达k8控制平面需要先设置route表

#kubeadm :用来初始化集群的指令
#kubelet :在集群中的每个节点上用来启动 Pod 和容器等
#kubectl :用来与集群通信的命令行工具
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF
#容器访问文件系统必须禁用selinux
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
yum install -y kubelet kubeadm kubectl ebtables ethtool --disableexcludes=kubernetes
systemctl enable --now kubelet

cat <<EOF | sudo tee kubeadm-config.yaml
kind: ClusterConfiguration
apiVersion: kubeadm.k8s.io/v1beta3
kubernetesVersion: v1.21.0
---
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
cgroupDriver: systemd
EOF
kubeadm init --config kubeadm-config.yaml
```