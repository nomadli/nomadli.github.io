---
layout:         post
title:          iptables
subtitle:       iptables
date:           2017-09-22 12:15:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
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


## firewalld
- 动态防火墙
- /etc/firewalld/优先加载
- /usr/lib/firewalld/
- 9个区域
    - trusted(信任区域)     允许所有的传入流量
    - public(公共区域)      允许ssh等预定义服务的传入流量,默认区域包含所有网卡
    - external(外部区域)    允许ssh等预定义服务的传入流量,做伪路由
    - home(家庭区域)        允许ssh(更多)等预定义服务的传入流量
    - internal(内部区域)    默认值与homel区域相同
    - work(工作区域)        允许ssh等预定义服务的传入流量
    - nm-share
    - dmz(隔离|非军事区域)   允许ssh等预定义服务的传入流量
    - block(限制区域)       拒绝所有传入流量
    - drop(丢弃区域)        丢弃所有传入流量,并且不产生包含ICMP的错误响应
- 根据源IP:PORT IP PORT 的顺序筛选区域判断规则,无对应规则使用public
- firewall-cmd  命令行规则配置、firewall-config 图形配置界面、直接编辑xml
```shell
firewall-cmd
    -h, --help                              #帮助
    -V, --version                           #版本信息
    -q, --quiet                             #不输出状态信息
    --state                                 #当前状态
    --reload                                #重新加载配置,保持已有链接
    --complete-reload                       #重新加载配置,断开所有链接
    --get-log-denied                        #当前拒绝日志级别
    --set-log-denied=all|unicast|broadcast|multicast|off
    --check-config                          #检测当前保存配置是否有误
    --panic-on                              #启用拒绝所有包模式
    --panic-off                             #禁用拒绝所有包模式
    --query-panic                           #是否启用拒绝所有包模式
    --permanent                             #指定针对保存的配置、或修改保存的配置
    --runtime-to-permanent                  #将临时规则保存,成为永久规则
    --timeout=<timeval>                     #'s' or 'm' or 'h'规则启用时长

    --get-default-zone                      #显示当前默认区域 
    --set-default-zone=zone                 #设置默认区域
    --get-active-zones                      #当前正在使用的区域及对应网卡
    [--permanent] --get-zones               #显示所有预定义的区域
    [--permanent] --get-zone-of-interface=l #显示指定网卡绑定的区域
    [--permanent] --get-zone-of-source=ip/24|mac|ipset:ipset
    [--permanent] --list-all-zones          #显示所有区域及其规则
    --permanent --new-zone=zone             #创建一个新的区域
    --permanent --new-zone-from-file=path [--name=zone]
    --permanent --delete-zone=zone          #删除一个区域
    --permanent --load-zone-defaults=zone   #加载区域默认配置
    --info-zone=zone                        #显示指定区域的规则
    --permanent --path-zone=zone            #显示指定区域保存路径
    --zone=zone                             #指定某个区域作为后面子命令的目标

    --get-policies                          #显示预定义策略
    --get-active-policies                   #显示当前使用的策略规则
    --list-all-policies                     #显示所有策略的规则及配置
    --new-policy=<policy>                   #添加一个策略
    --permanent --new-policy-from-file=path [--name=policy]
    --delete-policy=<policy>                #删除一个策略
    --load-policy-defaults=<policy>         #加载指定策略的默认规则
    --info-policy=<policy>                  #显示某策略的规则及配置
    --path-policy=<policy>                  #显示某策略保存路径
    --policy=<policy>                       #指定某个策略作为后面子命令的目标

    --get-ipset-types                       #显示支持的ipset类型
    --new-ipset=<ipset> --type=<ipset type> [--option=<key>[=<value>]]..
    --permanent --new-ipset-from-file=path [--name=<ipset>]
    --permanent --delete-ipset=<ipset>      #删除指定的ipset
    --permanent --load-ipset-defaults=<ipset>
    --info-ipset=<ipset>                    #显示指定ipset的配置
    --permanent --path-ipset=<ipset>        #显示指定ipset的保存路径
    --get-ipsets                            #显示预定义ipset
    --permanent --ipset=<ipset> --set-description="xx"
    --permanent --ipset=<ipset> --get-description
    --permanent --ipset=<ipset> --set-short="sort description"
    --permanent --ipset=<ipset> --get-short #显示指定ipset的简短说明
    [--permanent] --ipset=<ipset> --add-entry=<entry>
    [--permanent] --ipset=<ipset> --remove-entry=<entry>
    [--permanent] --ipset=<ipset> --query-entry=<entry>
    [--permanent] --ipset=<ipset> --get-entries
    [--permanent] --ipset=<ipset> --add-entries-from-file=<entry>
    [--permanent] --ipset=<ipset> --remove-entries-from-file=<entry>

    [--permanent] --get-icmptypes           #显示所有预定义的ICMP类型
    --permanent --new-icmptype=<icmptype>   #添加icmp类型
    --permanent --new-icmptype-from-file=<filename> [--name=<icmptype>]
    --permanent --delete-icmptype=<icmptype>#删除icmp类型
    --permanent --load-icmptype-defaults=<icmptype>
    --info-icmptype=<icmptype>              #显示icmptype的配置信息
    --permanent --path-icmptype=<icmptype>  #显示icmptype配置保存路径
    --permanent --icmptype=<icmptype> --set-description="xxx"
    --permanent --icmptype=<icmptype> --get-description
    --permanent --icmptype=<icmptype> --set-short="sort description"
    --permanent --icmptype=<icmptype> --get-short
    --permanent --icmptype=<icmptype> --add-destination=ipv4/ipv6:ip/24
    --permanent --icmptype=<icmptype> --remove-destination=ipv4/ipv6:ip/24
    --permanent --icmptype=<icmptype> --query-destination=ipv4/ipv6:ip/24
    --permanent --icmptype=<icmptype> --get-destinations

    [--permanent] --get-services            #显示所有预定义的服务
    --permanent --new-service=<service>     #添加新服务
    --permanent --new-service-from-file=path [--name=<service>]
    --permanent --delete-service=<service>  #删除服务
    --permanent --load-service-defaults=<service>
    --info-service=<service>                #显示服务配置
    --permanent --path-service=<service>    #显示服务保存地址
    --permanent --service=<service> --set-description="xxx"
    --permanent --service=<service> --get-description
    --permanent --service=<service> --set-short="short description"
    --permanent --service=<service> --get-short
    --permanent --service=<service> --add-port=<port>[-port]/<protocol>
    --permanent --service=<service> --remove-port=<port>[-port]/<protocol>
    --permanent --service=<service> --query-port=<port>[-port]/<protocol>
    --permanent --service=<service> --get-ports
    --permanent --service=<service> --add-protocol=<protocol>
    --permanent --service=<service> --remove-protocol=<protocol>
    --permanent --service=<service> --query-protocol=<protocol>
    --permanent --service=<service> --get-protocols
    --permanent --service=<service> --add-source-port=<p>[-p]/<protocol>
    --permanent --service=<service> --remove-source-port=<p>[-p]/<protocol>
    --permanent --service=<service> --query-source-port=<p>[-p]/<protocol>
    --permanent --service=<service> --get-source-ports
    --permanent --service=<service> --add-helper=<helper>
    --permanent --service=<service> --remove-helper=<helper>
    --permanent --service=<service> --query-helper=<helper>
    --permanent --service=<service> --get-service-helpers
    --permanent --service=<service> --set-destination=<ipv>:<address>[/<mask>]
    --permanent --service=<service> --remove-destination=<ipv>
    --permanent --service=<service> --query-destination=<ipv>:<addrs>[/<mask>]
    --permanent --service=<service> --get-destinations
    --permanent --service=<service> --add-include=<service>
    --permanent --service=<service> --remove-include=<service>
    --permanent --service=<service> --query-include=<service>
    --permanent --service=<service> --get-includes

    --get-helpers                           #显示预定义helper列表
    --permanent --new-helper=<helper> --module=<module> [--family=<family>]
    --permanent --new-helper-from-file=<filename> [--name=<helper>]
    --permanent --delete-helper=<helper>    #删除helper
    --permanent --load-helper-defaults=<helper>
    --info-helper=<helper>                  #显示helper信息
    --permanent --path-helper=<helper>      #显示helper保存路径
    --permanent --helper=<helper> --set-description=<description>
    --permanent --helper=<helper> --get-description
    --permanent --helper=<helper> --set-short=<description>
    --permanent --helper=<helper> --get-short
    --permanent --helper=<helper> --add-port=<portid>[-<portid>]/<protocol>
    --permanent  --helper=<helper> --remove-port=<port>[-port>]/<protocol>
    --permanent  --helper=<helper> --query-port=<portid>[-<portid>]/<protocol>
    --permanent --helper=<helper> --get-ports
    --permanent --helper=<helper> --set-module=<module>
    --permanent --helper=<helper> --get-module
    --permanent  --helper=<helper> --set-family={ipv4|ipv6|}
    --permanent --helper=<helper> --get-family

    #[P] permanent
    #[Z] zone=
    #[O] policy=
    #[T] timeout=
    --<P|Z|O> --list-all                    #显示指定类型的所有对象
    --permanent --<Z|O> --set-description=<description>
    --permanent --<Z|O> --get-description   #显示指定类型的描述
    --permanent --<Z|O> --get-target        #获取指定类型的目标
    --permanent --<Z|O> --set-target=<target>
    --<Z|O> --set-short=<description>       #设置指定类型的短描述
    --permanent --<Z|O> --get-short         #显示指定类型的短描述
    --<P|Z> --list-services                 #显示之地类型中的服务列表
    --<P|Z|O|T> --add-service=<service>     #添加服务到指定类型中
    --<P|Z|O> --remove-service=<service>    #从指定类型中删除服务
    --<P|Z|O> --query-service=<service>     #查询指定类型中是否有指定服务
    --<P|Z|O> --list-ports                  #显示指定类型中设置的端口
    --<P|Z|O|T> --add-port=<port>[-port]/<protocol>
    --<P|Z|O> --remove-port=<portid>[-<portid>]/<protocol>
    --<P|Z|O> --query-port=<portid>[-<portid>]/<protocol>
    --<P|Z|O> --list-protocols              #显示指定类型中设置的协议列表
    --<P|Z|O|T> --add-protocol=<protocol>   #添加协议到指定类型中
    --<P|Z|O> --remove-protocol=<protocol>  #从指定类型中删除指定协议
    --<P|Z|O> --query-protocol=<protocol>   #查询指定类型中是否有指定协议
    --<P|Z|O> --list-source-ports           #显示指定类型中配置的端口
    --<P|Z|O|T> --add-source-port=<portid>[-<portid>]/<protocol>
    --<P|Z|O> --remove-source-port=<portid>[-<portid>]/<protocol>
    --<P|Z|O> --query-source-port=<portid>[-<portid>]/<protocol>
    --<P|Z|O> --list-icmp-blocks            #显示指定类型是否有icmp类型
    --<P|Z|O|T> --add-icmp-block=<icmptype> #在指定类型中添加icmp类型
    --<P|Z|O> --remove-icmp-block=<icmptype>#删除指定类型中icmp类型
    --<P|Z|O> --query-icmp-block=<icmptype> #查询指定类型中是否有icmp类型
    --<P|Z|O> --list-forward-ports          #显示指定类型的转发端口
    --<P|Z|O|T> --add-forward-port=port=<sport>[-sport]:proto=<protocol>[:toport=dport[-dport]][:toaddr=<address>[/<mask>]]
    --<P|Z|O> --remove-forward-port=port=<portid>[-<portid>]:proto=<protocol>[:toport=<portid>[-<portid>]][:toaddr=<address>[/<mask>]]
    --<P|Z|O> --query-forward-port=port=<portid>[-<portid>]:proto=<protocol>[:toport=<portid>[-<portid>]][:toaddr=<address>[/<mask>]]
    --<P|Z|O|T> --add-masquerade            #在指定类型上添加nat转换
    --<P|Z|O> --remove-masquerade           #删除指定类型上的nat转换
    --<P|Z|O> --query-masquerade            #查询指定类型上是否有nat转换
    --<P|Z|O> --list-rich-rules             #显示指定类型上复杂规则
    --<P|Z|O|T> --add-rich-rule=<rule>      #在指定类型上添加复杂规则
    --<P|Z|O> --remove-rich-rule=<rule>     #在指定类型上删除复杂规则

    --<P|Z|O> --query-rich-rule=<rule>      #查询指定类型上是否有指定复杂规则
    --<P|Z> --add-icmp-block-inversion      #在指定类型启用icmp阻止规则反转
    --<P|Z> --remove-icmp-block-inversion   #失效指定类型icmp阻止规则反转
    --<P|Z> --query-icmp-block-inversion    #查询指定类型是否启用icmp阻止规则反转
    --<P|Z|T> --add-forward                 #启用指定类型的包转发
    --<P|Z> --remove-forward                #禁用指定类型的包转发
    --<P|Z> --query-forward                 #查询指定类型是否启用包转发

    --<P|O> --get-priority                  #显示指定策略优先级
    --<P|O> --set-priority=<priority>       #设置指定策略优先级
    --<P|O> --list-ingress-zones            #显示指定策略in流量涉及的区域
    --<P|O> --add-ingress-zone=<zone>       #为指定策略添加in流量区域
    --<P|O> --remove-ingress-zone=<zone>    #移除指定策略in流量区域
    --<P|O> --query-ingress-zone=<zone>     #查询指定策略in流量是否有指定区域
    --<P|O> --list-egress-zones             #显示指定策略out流量涉及的区域
    --<P|O> --add-egress-zone=<zone>        #为指定策略添加out流量区域
    --<P|O> --remove-egress-zone=<zone>     #移除指定策略out流量区域
    --<P|O> --query-egress-zone=<zone>      #查询指定策略out流量是否有指定区域
    
    --<P|Z> --list-sources                  #显示指定区域源列表
    --<P|Z> --add-source=<source>[/<mask>]|<MAC>|ipset:<ipset>
    --<P|Z> --change-source=<source>[/<mask>]|<MAC>|ipset:<ipset>
    --<P|Z> --query-source=<source>[/<mask>]|<MAC>|ipset:<ipset>
    --<P|Z> --remove-source=<source>[/<mask>]|<MAC>|ipset:<ipset>
    --<P|Z> --list-interfaces               #显示指定区域网卡列表
    --<P|Z> --add-interface=<interface>     #为指定区域添加网卡
    --<P|Z> --change-interface=<interface>  #修改指定区域网卡
    --<P|Z> --query-interface=<interface>   #查询指定区域是否包含指定网卡
    --<P|Z> --remove-interface=<interface>  #删除指定区域的指定网卡

    #类似iptable的规则
    [--permanent] --direct --get-all-chains #获取网络栈列表
    [--permanent] --direct --get-chains {ipv4|ipv6|eb} <table>
    [--permanent] --direct --add-chain {ipv4|ipv6|eb} <table> <chain>
    [--permanent] --direct --remove-chain {ipv4|ipv6|eb} <table> <chain>
    [--permanent] --direct --query-chain {ipv4|ipv6|eb} <table> <chain>
    [--permanent] --direct --get-all-rules  #获取所有规则
    [--permanent] --direct --get-rules {ipv4|ipv6|eb} <table> <chain>
    [--permanent] --direct --add-rule {ipv4|ipv6|eb} <table> <chain> <priority> <arg>...
    [--permanent] --direct --remove-rule {ipv4|ipv6|eb} <table> <chain> <priority> <arg>...
    [--permanent] --direct --remove-rules {ipv4|ipv6|eb} <table> <chain>
    [--permanent] --direct --query-rule {ipv4|ipv6|eb} <table> <chain> <priority> <arg>...
    [--permanent] --direct --passthrough {ipv4|ipv6|eb} <arg>...
    [--permanent] --direct --get-all-passthroughs
    [--permanent] --direct --get-passthroughs {ipv4|ipv6|eb} <arg>...
    [--permanent] --direct --add-passthrough {ipv4|ipv6|eb} <arg>...
    [--permanent] --direct --remove-passthrough {ipv4|ipv6|eb} <arg>...
    [--permanent] --direct --query-passthrough {ipv4|ipv6|eb} <arg>...

    --lockdown-on                           #开启lockdown
    --lockdown-off                          #禁用lockdown
    --query-lockdown                        #查询lockdown状态
    [--permanent] --list-lockdown-whitelist-commands
    [--permanent] --add-lockdown-whitelist-command=<command>
    [--permanent] --remove-lockdown-whitelist-command=<command>
    [--permanent] --query-lockdown-whitelist-command=<command>
    [--permanent] --list-lockdown-whitelist-contexts
    [--permanent] --add-lockdown-whitelist-context=<context>
    [--permanent] --remove-lockdown-whitelist-context=<context>
    [--permanent] --query-lockdown-whitelist-context=<context>
    [--permanent] --list-lockdown-whitelist-uids
    [--permanent] --add-lockdown-whitelist-uid=<uid>
    [--permanent] --remove-lockdown-whitelist-uid=<uid>
    [--permanent] --query-lockdown-whitelist-uid=<uid>
    [--permanent] --list-lockdown-whitelist-users
    [--permanent] --add-lockdown-whitelist-user=<user>
    [--permanent] --remove-lockdown-whitelist-user=<user>
    [--permanent] --query-lockdown-whitelist-user=<user>

firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.142.166" port protocol="tcp" port="5432" accept"
```
