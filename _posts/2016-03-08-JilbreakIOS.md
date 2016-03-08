---
layout: post
title:  "IOS 越狱"
date:   2016-01-04 11:36:00
categories: IOS
excerpt: IOS 越狱。
---

* content
{:toc}

## 目标
1. 内核、amfid、libmiss.dylib：三者配合实现了代码签名
2. 内核：对内存页属性的保护
3. 获取 root 权限：重新 mount 磁盘分区需要 root 权限

## USB接口权限
1. AFC apple file connect 文件管理
2. MobileBackup				系统备份
3. image Mounter				dmg文件挂载
4. installation Proxy		安装服务

## 越狱过程
1. 创建目录、目录连接等指向