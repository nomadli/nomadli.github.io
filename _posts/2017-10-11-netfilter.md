---
layout:         post
title:          iptables
subtitle:       iptables
date:           2017-09-22 12:15:00
author:         nomadli
header-img:     img/post-bg-ios9-web.jpg
catalog:        true
tags:
        - other
---

* content
{:toc} 

## 表(table) 排名分先后
1.  raw 数据跟踪 
2.  mangle 包修改
3.  nat  网络地址转换
4.  filter 包过滤  

## 命令(command)
- -A 在指定链的末尾添加一条新的规则
- -D 删除指定链中的某一条规则，可以按规则序号和内容删除
- -I 在指定链中插入一条新的规则，默认在第一行添加
- -R 修改、替换指定链中的某一条规则，可以按规则序号和内容替换
- -L 列出指定链中所有的规则进行查看
- -E 重命名用户定义的链，不改变链本身
- -F 清空（flush）
- -N 新建一条用户自己定义的规则链
- -X 删除指定表中用户自定义的规则链
- -P 设置指定链的默认策略
- -Z 将所有表的所有链的字节和数据包计数器清零
- -n 使用数字形式显示输出结果
- -v 查看规则表详细信息的信息
- -V 查看版本
- -h 获取帮助

## 入站链(chain) 排名分先后
01. PREROUTING(nat mangle raw)内核判断是否为本机数据
02. INPUT(filter mangle) 用户空间输入过滤

## 转发链(chain) 排名分先后
01. PREROUTING(nat mangle raw)内核判断是否为本机数据
02. FORWARD(filter mangle) 内核判断
03. POSTROUTING(nat raw mangle)输出内核过滤

## 出站链(chain) 排名分先后
01. OUTPUT(filter nat mangle raw) 用户空间输出过滤
02. POSTROUTING(nat raw mangle)输出内核过滤

## 条件(condition)
- -p TCP | UDP | ICMP | all | /etc/protocols 中任意name 
- -s networ name | hostname | 192.0../24 | IP 源地址
- -d networ name | hostname | 192.0../24 | IP 目的地址
- -i eth0 | eth+ 入口设备
- -o eth0 | eth+ 出口设备
- --sport 80 | 80:8080 源端口或范围
- --dport 80 | 80:8080 目的端口或范围

## 结果(target)
01. ACCEPT 允许数据包通过
02. DROP 丢弃数据包不响应
03. REJECT 拒绝数据包并发送拒绝响应 --reject-with tcp-reset
04. LOG 记录日志并检查下一条规则 --log-prefix "..."
05. REDIRECT 重新导向另一端口并继续检查规则 --to--ports
06. MASQUERADE 修改源IP为当前IP可指定端口并继续 --to--ports
07. SNAT 修改源IP或端口,并继续 --to-souce 10.1..-10.2..:2-9
08. DNAT 修改目标IP后端口,并继续 --to-destination 1.1..-1.2..:2-9
09. MIRROR 将源和目的地址互换并发回。
10. QUEUE 放入队列等待用户防火墙程序处理
11. RETURN 结束当前子过滤程序并返回上一级过滤继续执行
12. MARK 做标记继续下面的规则 --set-mark 2 

## iptables 语法
01. iptables [-t filter] command [chian] [condtion] [-j target]

## 举例
- service iptables save 保存规则到/etc/sysconfig/iptables
- iptables -D INPUT 1 删除IPUT链的filter表的第一条规则
- iptables -I INPUT -p icmp -j REJECT 拒绝进入防火墙的所有ICMP协议包
- iptables -A FORWARD -p ! icmp -j ACCEPT 允许转发除ICMP协议外所有数据包
- iptables -nL --line-number -t nat 显示nat表所有管道的规则及行号