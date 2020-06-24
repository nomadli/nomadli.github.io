---
layout:         post
title:          ClusterLabs集群模式
subtitle:       ClusterLabs集群模式
date:           2020-06-24 10:07:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
catalog:        true
tags:
        - other
---

* content
{:toc}

## ClusterLabs 链接
- [官网](https://clusterlabs.org)
- [github](https://github.com/ClusterLabs)
- [libqb](https://github.com/ClusterLabs/libqb)
- [corosync](https://github.com/corosync/corosync)
- [pacemaker](https://github.com/ClusterLabs/pacemaker)

## 部署
- /etc/yum.repos.d/xxx.repo
```txt
    [centos-7-base]
    name=CentOS-$releasever - Base
    mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
    #baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
    enabled=1
```
- [all] yum install pacemaker pcs resource-agents
- [all] systemctl start pcsd.service
- [all] systemctl enable pcsd.service
- [all] echo password | passwd --stdin username
- [one] pcs cluster auth ip1 ip2 ... -u username -p password --force
- [one] pcs cluster setup --force --name cluster_name<15 ip1 ip2 ...
- [one] pcs cluster start --all
- [one] pcs property set stonith-enabled=false 不使用远程电源管理
- [one] pcs property set no-quorum-policy=ignore 如果只有两个节点,仲裁是没有必要的
- [one] pcs resource defaults migration-threshold=1 当节点失败时,移除节点,通常不这样,而是等待恢复自动加入
- [one] pcs resource create sname Dummy op monitor interval=120s 启用名命名为s_name的服务120秒检测一次
- [one] pcs status 查看服务状态
- [one] crm_mon -1 查看服务状态
- [one] crm_resource --resource s_name --force-stop 将本机节点强制设置为故障