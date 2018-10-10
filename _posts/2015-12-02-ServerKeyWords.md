---
layout:         post
title:          服务器相关技术
subtitle:       服务器相关技术
date:           2015-12-02 16:58:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
catalog:        true
tags:
        - server
---

* content
{:toc}

## 动态库搜索路径
1. 编译时指定的动态库搜索路径
2. LD_LIBRARY_PATH 指定的动态库路径
3. /etc/ld.so.conf 指定的动态库路径
4. 默认路径/lib
5. 默认路径/usr/lib

#lib
001. boost
002. log4cxx
003. libapr
004. libaprutil
005. libcrypto
006. libcurl
007. libexpat
008. libgloox
009. libiconv
010. libsmtp
011. libsnmp
012. libssl
013. libsybdb
014. libmysqlclient
015. libcoredumper
016. libopenssl
017. libevent
018. redis
019. zookeeper
020. rdkafka
021. zlib

## 分布式开源框架
- Hadoop Apache开发的分布式系统基础架构，实现了一个分布式文件系统HDFS(Hadoop Distributed File System)和MapReduce海量数据计算,适合做搜索引擎。   [![github][1]](https://github.com/apache/hadoop)
- HBase Apache为Hadoop开发的一个分布式的、面向列的非结构化数据存储的数据库，源于Google论文Bigtable。最大特点是基于列而不是基于行的模式。   [![github][1]](https://github.com/apache/hbase)
- Storm Twitter开发的分布式的、容错的实时计算系统。   [![github][1]](https://github.com/apache/storm)
- ElasticSearch 基于Lucene的分布式搜索服务器。   [![github][1]](https://github.com/elastic/elasticsearch)

## linux系统监控
- netdata  [![github][1]](https://github.com/firehol/netdata)

## 云相关
001. KVM
002. XEN
003. hypervisor
101. CloudStack
102. OpenStack
103. AWS
104. VMware vCloud OpenStack vSphere vCenter
201. Docker
202. Kubernets
203. CoreOS
204. Mesos

## 开源监控网站
1. http://openradar.appspot.com

[1]: /img/github.png
