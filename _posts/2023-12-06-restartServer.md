---
layout:         post
title:          restart server
subtitle:       restart server
date:           2023-11-06 18:51:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
catalog:        true
tags:
        - server
---

* content
{:toc}

## TCP
![](../img/tcp2.webp)
![](../img/tcp.png)

- 由于有半链接和全链接内核队列, 因此关掉listen重新listen会丢掉这部分用户链接, 因此需要新进程继承listen
```go
package main

import (
    "context"
    "flag"
    "fmt"
    "net"
    "net/http"
    "os"
    "os/exec"
    "os/signal"
    "syscall"
)

var (
    upgrade bool
    ln      net.Listener
    server  *http.Server
)

func hello(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "hello world from pid:%d, ppid: %d\n",
        os.Getpid(), os.Getppid())
}

func main() {
    flag.BoolVar(&upgrade, "upgrade", false, "user can't use this")
    flag.Parse()
    http.HandleFunc("/", hello)
    server = &http.Server{Addr: ":8999"}
    var err error
    if upgrade { // 如果是平滑重启，会在fork时添加-upgrade的命令行参数
        // 继承的fd时从3开始的，（0,1,2分别备stdin,stdout,stderr占据了）
        fd := os.NewFile(3, "")
        // 平滑重启时，直接通过fd=3来继承套接字, 通过fd构造net.Listener时
        //，会将原理fd dup一份，因而下面要手动close以释放资源
        ln, err = net.FileListener(fd)
        if err != nil {
            fmt.Printf("fileListener fail, error: %s\n", err)
            os.Exit(1)
        }
        fd.Close() // 释放fd 3
    } else { // else分支对应服务首次启动，需要主动listen
        ln, err = net.Listen("tcp", server.Addr)
        if err != nil {
            fmt.Printf("listen %s fail, error: %s\n", server.Addr, err)
            os.Exit(1)
        }
    }
    go func() {
        err := server.Serve(ln)
        if err != nil && err != http.ErrServerClosed {
            fmt.Printf("serve error: %s\n", err)
        }
    }()
    setupSignal()
    fmt.Println("over")
}

func setupSignal() {
    ch := make(chan os.Signal, 1)
    signal.Notify(ch, syscall.SIGUSR2, syscall.SIGINT, syscall.SIGTERM)
    sig := <-ch
    switch sig {
    case syscall.SIGUSR2: // 通过给服务发送USR2信号触发平滑重启
        fmt.Println("SIGUSR2 received")
        err := forkProcess()
        if err != nil {
            fmt.Printf("fork process error: %s\n", err)
        }

        // 调用go标准库里的优雅重启方法，方法中会停止accept新连接，
        // 处理完历史连接后就退出
        err = server.Shutdown(context.Background())
        if err != nil {
            fmt.Printf("shutdown after forking process error: %s\n", err)
        }

    case syscall.SIGINT, syscall.SIGTERM: // 这两个信号只触发优雅退出
        signal.Stop(ch)
        close(ch)
        err := server.Shutdown(context.Background())
        if err != nil {
            fmt.Printf("shutdown error: %s\n", err)
        }
    }
}

func forkProcess() error {
    flags := []string{"-upgrade"} // 添加命令行参数，告知子进程继承fd而不要重新监听
    fmt.Printf("forkProcess - arg: %v", os.Args[0])
    // 将fork+exec两个系统后调用封装到了一起，os.Args[0]就是服务的binary
    // 所在路径，如果升级服务，平滑重启前需要覆盖服务的binary！！！
    cmd := exec.Command(os.Args[0], flags...)
    cmd.Stderr = os.Stderr
    cmd.Stdout = os.Stdout
    l, _ := ln.(*net.TCPListener)
    lfd, err := l.File()
    if err != nil {
        return err
    }
    // ExtraFiles填入继承的fd，GO标准库会保证继承的
    // fd时从3开始（0,1,2分别备stdin,stdout,stderr占据了）
    cmd.ExtraFiles = []*os.File{lfd}
    return cmd.Start()
}
```
```C
#include <unistd.h> 
#include <sys/types.h> 
#include <sys/stat.h> 
#include <stdlib.h>
#include <stdio.h>
#include <sys/stat.h> 
#include <fcntl.h>
#include <string.h>

int main()
{
 pid_t pid;
 int i, fd;
 char *buf = "This is a Daemon\n";

 pid = fork();
 if (pid < 0) {
  exit(1);
 } 
 
 if (pid > 0) {
  exit(0); 
 }
 setsid();
 chdir("/");  
 umask(0);
 
 while(1) {
  if ((fd = open("/tmp/daemon.log", 
    O_CREAT|O_WRONLY|O_APPEND, 0600)) < 0) {
   exit(1);
  }
  write(fd, buf, strlen(buf) + 1);
  close(fd);
  sleep(10);
 }
 
 exit(0);
}
```

## int prctl(int option,unsigned long arg2...5) option:
- PR_GET_PDEATHSIG      返回处理器信号
- PR_SET_PDEATHSIG      arg2作为处理器信号pdeath被输入,如果父进程退出,进程接受这个信号
- PR_GET_DUMPABLE       返回处理器标志dumpable
- PR_SET_DUMPABLE       arg2作为处理器标志dumpable被输入
- PR_GET_NAME           返回调用进程的进程名字给参数arg2  kernel >= 2.6.9
- PR_SET_NAME           把参数arg2作为调用进程的经常名字 kernel >= 2.6.11
- PR_GET_TIMING         返回进程计时模式
- PR_SET_TIMING         判定和修改进程计时模式,用于启用传统进程计时模式的
- PR_TIMING_STATISTICAL 或用于启用基于时间戳的进程计时模式的PR_TIMING_TIMESTAMP
- CAP_CHOWN：           _POSIX_CHOWN_RESTRICTED定义的系统,改变系统文件所有者和组所有的权限
- CAP_DAC_OVERRIED      如果_POSIX_ACL定义绕过所有的DAC访问,包括ACL执行访问
- CAP_DAC_READ_SEARCH   如果_POSIX_ACL定义绕过所有DAC的读限制,并在所有的文件和目录里搜索
- CAP_LINUX_IMMUTABLE   排除CAP_DAC_OVERRIED和CAP_DAC_READ_SEARCH绕过的访问控制
- CAP_FOWNER            绕过所有文件所有允许限制, 不绕过CAP_FSETID,MAC,DAC
- CAP_FSETID            绕过文件的S_ISUID和S_ISGID位的时候，chown来设置S_ISUID和S_ISGID
- CAP_FS_MASK           用来回应suser()或是fsuser()
- CAP_KILL              一个有有效用户ID的进程发送信号时必须匹配有效用户ID的功能会越过
- CAP_SETGID            允许setgid()和setgroups()
- CAP_SETUID            允许set*uid()
- CAP_SETPCAP           把所有的许可给所有的pid。或是把所有的许可删除
- CAP_LINUX_IMMUTABLE   允许更改S_IMMUTABLE和S_APPEND文件属性
- CAP_NET_BIND_SERVICE  允许绑定1024下的TCP/UDP套接字
- CAP_NET_BROADCAST     允许广播，监听多点传送
- CAP_NET_ADMIN         允许配置接口,管理IP防火墙IP伪装和帐户,配置socket调试选项,修改路由表
                        配置socket上的进程的组属性,绑定所有地址的透明代理,允许配置TOS
                        配置混杂模式,清除驱动状态,多点传送,读或写系统记录
- CAP_NET_RAW           允许用RAW套接字 允许用PACKET套接字
- CAP_IPC_LOCK          允许琐定共享内存段  允许mlock和mlockall
- CAP_IPC_OWNER         越过IPC所有权检查
- CAP_SYS_MODULE        插入或删除内核模块
- CAP_SYS_RAWIO         允许ioperm/iopl /dev/prot /dev/mem /dev/kmem 块设备访问/dev/*
- CAP_SYS_CHROOT        允许chroot()
- CAP_SYS_PTRACE        允许ptrace()任何进程
- CAP_SYS_PACCT         允许配置进程帐号
- CAP_SYS_ADMIN         允许配置安全钥匙,管理随机设备,设备管理,检查和配置磁盘限额,配置内核日志
                        配置域名,配置主机名,调用bdflush(),mount(),umount(),配置smb连接
                        ioctls,nfsservctl,VM86_REQUEST_IRQ,在alpha上读写pci配置
                        mips上的irix_prctl,刷新所有的m68k缓存,删除semaphores
                        用CAP_CHOWN去代替"chown"IPC消息队列，标志和共享内存
                        锁定或是解锁共享内存段,开关swap,在socket伪装pids
                        设置块设备的缓存刷新,设置软盘驱动器,开关DMA开关,管理md设备
                        管理ide驱动,访问nvram设备,管理apm_bios,串口,bttv电视设备
                        在isdn CAPI的驱动下生成命令,读取pci的非标准配置,DDI调试ioctl
                        发送qic-117命令,启动或禁止SCSI的控制,发送SCSI命令,配置加密口令
- CAP_SYS_BOOT          允许reboot()
- CAP_SYS_NICE          提高或设置其他进程的优先权,用FISO和实时的安排和配置
- CAP_SYS_RESOURCE      越过资源限制,设置资源限制,越过配额限制,越过保留的ext2文件系统
                        允许大于64hz的实时时钟中断,越过最大数目的控制终端,越过最大数目的键
- CAP_SYS_TIME          允许处理系统时钟,允许_stime,允许设置实时时钟
- CAP_SYS_TTY_CONFIG    允许配置终端设备,允许vhangup()终端返回值
- PR_GET_DUMPABLE和PR_GET_KEEPCAPS调用成功时返回0或者1,其他成功时返回0 错误时返回－1 errno
- EINVAL                option的值或参数不正确
- EBADF                 无效的描述符

## 端口
- http://www.iana.org/assignments/port-numbers
- IANA建议其它服务监听49152-65535
- BSD系统服务监听端口范围是1024-4999。
- Linux使用32768-61000范围可在/proc/sys/net/ipv4/ip_local_port_range配置
- 其它端口作为主动发起链接时的随机端口
- 0   :无效端口,通常用于分析操作系统
- 1   :传输控制协议端口服务多路开关选择器
- 2   :管理实用程序
- 3   :压缩进程
- 5   :远程作业登录
- 7   :回显
- 9   :丢弃
- 11  :在线用户
- 13  :时间
- 17  :每日引用
- 18  :消息发送协议
- 19  :字符发生器
- 20  :FTP 文件传输协议(默认数据口)
- 21  :FTP 文件传输协议(控制)
- 22  :SSH 远程登录协议
- 23  :telnet(终端仿真协议),木马 Tiny Telnet Server 开放此端口
- 24  :预留给个人用邮件系统
- 25  :SMTP 服务器所开放的端口，用于发送邮件
- 27  :NSW 用户系统 FE
- 29  :MSG ICP
- 31  :MSG 验证,木马 Master Paradise、HackersParadise 开放此端口
- 33  :显示支持协议
- 35  :预留给个人打印机服务
- 37  :时间
- 38  :路由访问协议
- 39  :资源定位协议
- 41  :图形
- 42  :主机名服务
- 43  :who is 服务
- 44  :MPM(消息处理模块)标志协议
- 45  :消息处理模块
- 46  :消息处理模块(默认发送口)
- 47  :NI FTP
- 48  :数码音频后台服务
- 49  :TACACS 登录主机协议
- 50  :远程邮件检查协议
- 51  :IMP(接口信息处理机)逻辑地址维护
- 52  :施乐网络服务系统时间协议
- 53  :dns 域名服务器
- 54  :施乐网络服务系统票据交换
- 55  :ISI 图形语言
- 56  :施乐网络服务系统验证
- 57  :预留个人用终端访问
- 58  :施乐网络服务系统邮件
- 59  :预留个人文件服务
- 60  :未定义
- 61  :NI 邮件
- 62  :异步通讯适配器服务
- 63  :whois++
- 64  :通讯接口
- 65  :TACACS 数据库服务
- 66  :Oracle SQL*NET
- 67  :引导程序协议服务端
- 68  :引导程序协议客户端
- 69  :小型文件传输协议
- 70  :信息检索协议
- 71  :远程作业服务
- 72  :远程作业服务
- 73  :远程作业服务
- 74  :远程作业服务
- 75  :预留给个人拨出服务
- 76  :分布式外部对象存储
- 77  :预留给个人远程作业输入服务
- 78  :修正 TCP
- 79  :查询远程主机在线用户等信息
- 80  :http,用于网页浏览,木马 Executor 开放此端口
- 81  :HOST2 名称服务
- 82  :传输实用程序
- 83  :模块化智能终端 ML 设备
- 84  :公用追踪设备
- 85  :模块化智能终端 ML 设备
- 86  :Micro Focus Cobol 编程语言
- 87  :预留给个人终端连接
- 88  :Kerberros 安全认证系统
- 89  :SU/MIT telnet(终端仿真网关)
- 90  :DNSIX 安全属性标记图
- 91  :MIT Dover 假脱机
- 92  :网络打印协议
- 93  :设备控制协议
- 94  :Tivoli 对象调度
- 96  :DIXIE 协议规范
- 97  :快速远程虚拟文件协议
- 98  :TAC 新闻协议
- 99  :后门程序 ncx99 开放此端口
- 100 :未知用途
- 101 :NIC 主机名称服务
- 102 :消息传输代理
- 103 :Genesis 点对点传输网络
- 105 :信箱名称服务
- 106 :3COM-TSMUX 开放端口
- 107 :远程 Telnet 服务
- 108 :SNA 网关访问服务
- 109 :POP2 服务器开放此端口,用于接收邮件
- 110 :POP3 服务器开放此端口,用于接收邮件
- 111 :SUN 公司的 RPC 服务所有端口
- 112 :McIDAS 数据传输协议
- 113 :认证服务，用于鉴别 TCP 连接的用户
- 114 :音频新闻多点服务
- 115 :简单文件传输服务
- 116 :ANSA REX 通知
- 117 :UUCP 路径服务
- 118 :SQL 服务
- 119 :NEWS 新闻组传输协议，承载 USENET 通信
- 121 :木马 BO jammerkillahV 开放端口
- 122 :SMAKY 网络
- 123 :网络时间协议，蠕虫病毒会利用，一般关闭
- 128 :GSS X 许可认证
- 129 :密码生成器协议
- 130 :Cisco 软件开放端口
- 131 :Cisco 软件开放端口
- 132 :Cisco 软件开放端口
- 133 :统计服务
- 134 :INGRES-网络服务
- 135 :DCOM 服务，冲击波病毒利用，不能关闭
- 136 :命名系统
- 137 :NETBIOS 协议应用，为共享开放
- 138 :NETBIOS 协议应用，为共享开放
- 139 :NETBIOS 协议应用，为共享开放
- 140 :EMFIS 数据服务
- 141 :EMFIS 控制服务
- 143 :Interim 邮件访问协议
- 144 :UMA 软件开放端口
- 145 :UAAC 协议
- 149 :AED 512 仿真服务
- 150 :SQL(结构化查询语言)-网络
- 152 :后台文件传输协议
- 156 :SQL(结构化查询语言)服务
- 158 :PC 邮件服务器
- 159 :NSS-路由
- 160 :SGMP-陷阱
- 161 :简单网络管理协议
- 162 :SNMP 陷阱
- 163 :CMIP/TCP 管理
- 164 :CMIP/TCP 代理
- 166 :Sirius 系统
- 169 :发送
- 170 :网络附言
- 177 :x 显示管理控制协议，入侵者通过它访问 X-windows 操作台
- 178 :NextStep Window 服务
- 179 :边界网关协议
- 180 :图表
- 181 :统一
- 184 :OC 服务器
- 185 :远程-KIS
- 186 :KIS 协议
- 187 :应用通信接口
- 189 :队列文件传输
- 190 :网关进入控制协议
- 191 :Prospero 目录服务
- 192 :OSU 网络监视系统
- 193 :Spider 远程控制协议
- 194 :多线交谈协议
- 197 :目录地址服务
- 198 :目录地址服务监视器
- 200 :IBM 系统资源控制器
- 201 :AppleTalk(Mac 机所用的网络协议)路由保证
- 202 :AppleTalk(Mac 机所用的网络协议)Name Binding
- 203 :AppleTalk(Mac 机所用的网络协议)未用端口
- 204 :AppleTalk(Mac 机所用的网络协议)回显
- 205 :AppleTalk(Mac 机所用的网络协议)未用端口
- 206 :AppleTalk(Mac 机所用的网络协议)区信息
- 207 :AppleTalk(Mac 机所用的网络协议)未用端口
- 208 :AppleTalk(Mac 机所用的网络协议)未用端口
- 209 :快速邮件传输协议
- 210 :ANSI(美国国家标准协会)Z39.50
- 211 :Texas Instruments 914C/G 终端
- 213 :IPX(以太网所用的协议)
- 218 :Netix 消息记录协议
- 219 :Unisys ARPs
- 220 :交互邮件访问协议 v3
- 223 :证书分发中心
- 224 :masq 拨号器
- 241 :预留端口 (224-241)
- 245 :链接
- 246 :显示系统协议
- 257 :安全电子交易系统
- 258 :Yak Winsock 个人聊天
- 259 :有效短程遥控
- 260 :开放端口
- 261 :IIOP 基于 TLS/SSL 的命名服务
- 266 :SCSI(小型计算机系统接口)on ST
- 267 :Tobit David 服务层
- 268 :Tobit David 复制
- 281 :个人连结
- 282 :Cable 端口 A/X
- 286 :FXP 通信
- 308 :Novastor 备份
- 313 :Magenta 逻辑
- 318 :PKIX 时间标记
- 333 :Texar 安全端口
- 344 :Prospero 数据存取协议
- 345 :Perf 分析工作台
- 346 :Zebra 服务器
- 347 :Fatmen 服务器
- 348 :Cabletron 管理协议
- 358 :Shrink 可上网家电协议
- 359 :网络安全风险管理协议
- 362 :SRS 发送
- 363 :RSVP 隧道
- 372 :列表处理
- 373 :Legend 公司
- 374 :Legend 公司
- 376 :AmigaEnvoy 网络查询协议
- 377 :NEC 公司
- 378 :NEC 公司
- 379 :TIA/EIA/IS-99 调制解调器客户端
- 380 :TIA/EIA/IS-99 调制解调器服务器
- 381 :hp(惠普)性能数据收集器
- 382 :hp(惠普)性能数据控制节点
- 383 :hp(惠普)性能数据警报管理
- 384 :远程网络服务器系统
- 385 :IBM 应用程序
- 386 :ASA 信息路由器定义文件.
- 387 :Appletalk 更新路由.
- 389 :轻型目录访问协议
- 395 :网络监视控制协议
- 396 :Novell(美国 Novell 公司)Netware(Novell 公司出的网络操作系统)over IP
- 400 :工作站解决方案
- 401 :持续电源
- 402 :Genie 协议
- 406 :交互式邮件支持协议
- 408 :Prospero 资源管理程序
- 409 :Prospero 资源节点管理.
- 410 :DEC(数据设备公司)远程调试协议
- 411 :远程 MT 协议
- 412 :陷阱协定端口
- 413 :存储管理服务协议
- 414 :信息查询
- 415 :B 网络
- 423 :IBM 操作计划和控制开端
- 424 :IBM 操作计划和控制追踪
- 425 :智能计算机辅助设计
- 427 :服务起位置
- 434 :移动 ip 代理
- 435 :移动 ip 管理
- 443 :基于 TLS/SSL 的网页浏览端口，能提供加密和通过安全端口传输的另一种 HTTP
- 444 :简单网络内存分页协议
- 445 :Microsoft-DS，为共享开放，震荡波病毒利用，一般应关闭
- 446 :DDM-远程关系数据库访问
- 447 :DDM-分布式文件管理
- 448 :DDM-使用安全访问远程数据库
- 456 :木马 HACKERS PARADISE 开放此端口
- 458 :apple quick time 软件开放端口
- 459 :ampr-rcmd 命令
- 464 :k 密码服务
- 469 :广播控制协议
- 470 :scx-代理
- 472 :ljk-登陆
- 481 :Ph 服务
- 487 :简单异步文件传输
- 489 :nest-协议
- 491 :go-登陆
- 499 :ISO ILL 协议
- 500 :Internet 密钥交换，Lsass 开放端口，不能关闭
- 509 :陷阱
- 510 :FirstClass 协议
- 512 :远程进程执行
- 513 :远程登陆
- 514 :cmd 命令
- 515 :spooler
- 516 :可视化数据
- 518 :交谈
- 519 :unix 时间
- 520 :扩展文件名称服务器
- 525 :时间服务
- 526 :新日期
- 529 :在线聊天系统服务
- 530 :远程过程调用
- 531 :聊天
- 532 :读新闻
- 533 :紧急广播端口
- 534 :MegaMedia 管理端
- 537 :网络流媒体协议
- 542 :商业
- 543 :Kerberos(软件)v4/v5
- 544 :krcmd 命令
- 546 :DHCPv6 客户端
- 547 :DHCPv6 服务器
- 552 :设备共享
- 554 :Real Time Stream 控制协议
- 555 :木马 PhAse1.0、Stealth Spy、IniKiller 开放此端口
- 556 :远距离文件服务器
- 563 :基于 TLS/SSL 的网络新闻传输协议
- 564 :plan 9 文件服务
- 565 :whoami 查询
- 566 :streettalk
- 567 :banyan-rpc(远程过程调用)
- 568 :DPA 成员资格
- 569 :MSN 成员资格
- 570 :demon(调试监督程序)
- 571 :udemon(调试监督程序)
- 572 :声纳
- 573 :banyan-贵宾
- 574 :FTP 软件代理系统
- 581 :Bundle Discovery 协议
- 582 :SCC 安全
- 583 :Philips 视频会议
- 584 :密钥服务器
- 585 :IMAP4+SSL (Use 993 instead)
- 586 :密码更改
- 587 :申请
- 589 :Eye 连结
- 595 :CAB 协议
- 597 :PTC 名称服务
- 598 :SCO 网络服务器管理 3
- 599 :Aeolon Core 协议
- 600 :Sun IPC(进程间通讯)服务器
- 601 :可靠系统登陆服务
- 604 :通道
- 606 :Cray 统一资源管理
- 608 :发送人-传递/提供 文件传输器
- 609 :npmp-陷阱
- 610 :npmp-本地
- 611 :npmp-gui( 图形用户界面)
- 612 :HMMP 指引
- 613 :HMMP 操作
- 614 :SSL(加密套接字协议层)shell(壳)
- 615 :Internet 配置管理
- 616 :SCO(Unix 系统)系统管理服务器
- 617 :SCO 桌面管理服务器
- 619 :Compaq(康柏公司)EVM
- 620 :SCO 服务器管理
- 623 :ASF 远程管理控制协议
- 624 :Crypto 管理
- 631 :IPP (Internet 打印协议)
- 633 :服务更新(Sterling 软件)
- 637 :局域网服务器
- 641 :repcmd 命令
- 647 :DHCP(动态主机配置协议)Failover
- 648 :注册登记协议(RRP)
- 649 :Cadview-3d 软件协议
- 666 :木马 Attack FTP、Satanz Backdoor 开放此端口
- 808 :ccproxy http/gopher/ftp (over http)协议
- 1001:木马 Silencer，WebEx 开放端口
- 1011:木马 Doly 开放端口
- 1024:动态端口的开始,木马 yai 开放端口
- 1025:inetinfo.exe(互联网信息服务)木马 netspy 开放端口
- 1026:inetinfo.exe(互联网信息服务)
- 1027:应用层网关服务
- 1030:应用层网关服务
- 1031:BBN IAD
- 1033:本地网络信息端口
- 1034:同步通知
- 1036:安全部分传输协议
- 1070:木马 Psyber Stream，Streaming Audio 开放端口
- 1071:网络服务开放端口
- 1074:网络服务开放端口
- 1080:Socks 这一协议以通道方式穿过防火墙，允许防火墙后面的人通过一个 IP 地址访问 INTERNET
- 1110:卡巴斯基反病毒软件开放此端口
- 1125:卡巴斯基反病毒软件开放此端口
- 1203:许可证生效端口
- 1204:登陆请求监听端口
- 1206:Anthony 数据端口
- 1222:SNI R&D 网络端口
- 1233:普遍的附录服务器端口
- 1234:木马 SubSeven2.0、Ultors Trojan 开放此端口
- 1243:木马 SubSeven1.0/1.9 开放此端口
- 1245:木马 Vodoo，GabanBus，NetBus，Vodoo 开放此端口
- 1273:EMC-网关端口
- 1289:JWalk 服务器端口
- 1290:WinJa 服务器端口
- 1333:密码策略(网络服务)(svchost.exe)
- 1334:网络服务(svchost.exe)
- 1335:数字公正协议
- 1336:即时聊天协议(svchost.exe)
- 1349:注册网络协议端口
- 1350:注册网络协议端口
- 1352:tcp lotusnote lotus note
- 1371:富士通配置协议端口
- 1372:富士通配置协议端口
- 1374:EPI 软件系统端口
- 1376:IBM 个人-个人软件端口
- 1377:Cichlid 许可证管理端口
- 1378:Elan 许可证管理端口
- 1380:Telesis 网络许可证管理端口
- 1381:苹果网络许可证管理端口
- 1386:CheckSum 许可证管理端口
- 1387:系统开放端口(rundll32.exe)
- 1388:数据库高速缓存端口
- 1389:文档管理端口
- 1390:存储控制器端口
- 1391:存储器存取服务器端口
- 1392:打印管理端口
- 1393:网络登陆服务器端口
- 1394:网络登陆客户端端口
- 1395:PC 工作站管理软件端口
- 1396:DVL 活跃邮件端口
- 1397:音频活跃邮件端口
- 1398:视频活跃邮件端口
- 1399:Cadkey 许可证管理端口
- 1433:Microsoft 的 SQL 服务开放端口
- 1434:Microsoft 的 SQL 服务监视端口
- 1492:木马 FTP99CMP 开放此端口
- 1509:木马 Psyber Streaming Server 开放此端口
- 1512:Microsoft Windows 网络名称服务
- 1524:许多攻击脚本安装一个后门 SHELL 于这个端口
- 1600:木马 Shivka-Burka 开放此端口
- 1645:远程认证拨号用户服务
- 1701:第 2 层隧道协议
- 1731:NetMeeting 音频调用控制
- 1801:Microsoft 消息队列服务器
- 1807:木马 SpySender 开放此端口
- 1900:可被利用 ddos 攻击，一般关闭
- 1912:金山词霸开放此端口
- 1981:木马 ShockRave 开放此端口
- 1999:木马 BackDoor,yai 开放此端口
- 2000:木马 GirlFriend 1.3、Millenium 1.0 开放此端口
- 2001:木马 Millenium 1.0、Trojan Cow,黑洞 2001 开放此端口
- 2003:GNU 查询
- 2023:木马 Pass Ripper 开放此端口
- 2049:NFS 程序常运行于此端口
- 2115:木马 Bugs 开放此端口
- 2140:木马 Deep Throat 1.0/3.0，The Invasor 开放此端口
- 2500:应用固定端口会话复制的 RPC 客户
- 2504:网络平衡负荷
- 2565:木马 Striker 开放此端口
- 2583:木马 Wincrash 2.0 开放此端口
- 2801:木马 Phineas Phucker 开放此端口
- 2847:诺顿反病毒服务开放此端口
- 3024:木马 WinCrash 开放此端口
- 3128:squid http 代理服务器开放此端口
- 3129:木马 Master Paradise 开放此端口
- 3150:木马 The Invasor,deep throat 开放此端口
- 3210:木马 SchoolBus 开放此端口
- 3306:MySQL 开放此端口
- 3333:木马 Prosiak 开放此端口
- 3389:WINDOWS 2000 终端开放此端口
- 3456:inetinfo.exe(互联网信息服务)开放端口，VAT 默认数据
- 3457:VAT 默认控制
- 3527:Microsoft 消息队列服务器
- 3700:木马 Portal of Doom 开放此端口
- 3996:木马 RemoteAnything 开放此端口
- 4000:腾讯 QQ 客户端开放此端口
- 4060:木马 RemoteAnything 开放此端口
- 4092:木马 WinCrash 开放此端口
- 4133:NUTS Bootp 服务器
- 4134:NIFTY-Serve HMI 协议
- 4141:Workflow 服务器
- 4142:文档服务器
- 4143:文档复制
- 4145:VVR 控制
- 4321:远程 Who Is 查询
- 4333:微型 sql 服务器
- 4349:文件系统端口记录
- 4350:网络设备
- 4351:PLCY 网络服务
- 4453:NSS 警报管理
- 4454:NSS 代理管理
- 4455:PR 聊天用户
- 4456:PR 聊天服务器
- 4457:PR 注册
- 4480:Proxy+ HTTP 代理端口
- 4500:Lsass 开放端口，不能关闭
- 4547:Lanner 许可管理
- 4555:RSIP 端口
- 4590:木马 ICQTrojan 开放此端口
- 4672:远程文件访问服务器
- 4752:简单网络音频服务器
- 4800:Icona 快速消息系统
- 4801:Icona 网络聊天
- 4802:Icona 许可系统服务器
- 4848:App 服务器-Admin HTTP
- 4849:App 服务器-Admin HTTPS
- 4950:木马 IcqTrojan 开放 5000 端口
- 5000:木马 blazer5，Sockets de Troie 开放 5000 端口，一般应关闭
- 5001:木马 Sockets de Troie 开放 5001 端口
- 5006:wsm 服务器
- 5007:wsm 服务器 ssl
- 5022:mice 服务器
- 5050:多媒体会议控制协议
- 5051:ITA 代理
- 5052:ITA 管理
- 5137:MyCTS 服务器端口
- 5150:Ascend 通道管理协议
- 5154:BZFlag 游戏服务器
- 5190:America-Online(美国在线)
- 5191:AmericaOnline1(美国在线)
- 5192:AmericaOnline2(美国在线)
- 5193:AmericaOnline3(美国在线)
- 5222:Jabber 客户端连接
- 5225:HP(惠普公司)服务器
- 5226:HP(惠普公司)
- 5232:SGI 绘图软件端口
- 5250:i 网关
- 5264:3Com 网络端口 1
- 5265:3Com 网络端口 2
- 5269:Jabber 服务器连接
- 5306:Sun MC 组
- 5321:木马 Sockets de Troie 开放 5321 端口
- 5400:木马 Blade Runner 开放此端口
- 5401:木马 Blade Runner 开放此端口
- 5402:木马 Blade Runner 开放此端口
- 5405:网络支持
- 5409:Salient 数据服务器
- 5410:Salient 用户管理
- 5415:NS 服务器
- 5416:SNS 网关
- 5417:SNS 代理
- 5421:网络支持 2
- 5423:虚拟用户
- 5427:SCO-PEER-TTA(Unix 系统)
- 5432:PostgreSQL 数据库
- 5550:木马 xtcp 开放此端口
- 5569:木马 Robo-Hack 开放此端口
- 5599:公司远程安全安装
- 5600:公司安全管理
- 5601:公司安全代理
- 5631:pcANYWHERE(软件)数据
- 5632:pcANYWHERE(软件)数据
- 5673:JACL 消息服务器
- 5675:V5UA 应用端口
- 5676:RA 管理
- 5678:远程复制代理连接
- 5679:直接电缆连接
- 5720:MS-执照
- 5729:Openmail 用户代理层
- 5730:Steltor’s 日历访问
- 5731:netscape(网景)suiteware
- 5732:netscape(网景)suiteware
- 5742:木马 WinCrash1.03 开放此端口
- 5745:fcopy-服务器
- 5746:fcopys-服务器
- 5755:OpenMail(邮件服务器)桌面网关服务器
- 5757:OpenMail(邮件服务器)X.500 目录服务器
- 5766:OpenMail (邮件服务器)NewMail 服务器
- 5767:OpenMail (邮件服务器)请求代理曾(安全)
- 5768:OpenMail(邮件服务器) CMTS 服务器
- 5777:DALI 端口
- 5800:虚拟网络计算
- 5801:虚拟网络计算
- 5802:虚拟网络计算 HTTP 访问, d
- 5803:虚拟网络计算 HTTP 访问, d
- 5900:虚拟网络计算机显示 0
- 5901:虚拟网络计算机显示 1
- 5902:虚拟网络计算机显示 2
- 5903:虚拟网络计算机显示 3
- 6000:X Window 系统
- 6001:X Window 服务器
- 6002:X Window 服务器
- 6003:X Window 服务器
- 6004:X Window 服务器
- 6005:X Window 服务器
- 6006:X Window 服务器
- 6007:X Window 服务器
- 6008:X Window 服务器
- 6009:X Window 服务器
- 6456:SKIP 证书发送
- 6471:LVision 许可管理器
- 6505:BoKS 管理私人端口
- 6506:BoKS 管理公共端口
- 6507:BoKS Dir 服务器,私人端口
- 6508:BoKS Dir 服务器,公共端口
- 6509:MGCS-MFP 端口
- 6510:MCER 端口
- 6566:SANE 控制端口
- 6580:Parsec 主服务器
- 6581:Parsec 对等网络
- 6582:Parsec 游戏服务器
- 6588:AnalogX HTTP 代理端口
- 6631:Mitchell 电信主机
- 6667:Internet 多线交谈
- 6668:Internet 多线交谈
- 6670:木马 Deep Throat 开放此端口
- 6671:木马 Deep Throat 3.0 开放此端口
- 6699:Napster 文件(MP3)共享服务
- 6701:KTI/ICAD 名称服务器
- 6788:SMC 软件-HTTP
- 6789:SMC 软件-HTTPS
- 6841:Netmo 软件默认开放端口
- 6842:Netmo HTTP 服务
- 6883:木马 DeltaSource 开放此端口
- 6939:木马 Indoctrination 开放此端口
- 6969:木马 Gatecrasher、Priority 开放此端口
- 6970:real 音频开放此端口
- 7000:木马 Remote Grab 开放此端口
- 7002:使用者& 组 数据库
- 7003:音量定位数据库
- 7004:AFS/Kerberos 认证服务
- 7005:音量管理服务
- 7006:错误解释服务
- 7007:Basic 监督进程
- 7008:服务器-服务器更新程序
- 7009:远程缓存管理服务
- 7011:Talon 软件发现端口
- 7012:Talon 软件引擎
- 7013:Microtalon 发现
- 7014:Microtalon 通信
- 7015:Talon 网络服务器
- 7020:DP 服务
- 7021:DP 服务管理
- 7100:X 字型服务
- 7121:虚拟原型许可证管理
- 7300:木马 NetMonitor 开放此端口
- 7301:木马 NetMonitor 开放此端口
- 7306:木马 NetMonitor，NetSpy1.0 开放此端口
- 7307:木马 NetMonitor 开放此端口
- 7308:木马 NetMonitor 开放此端口
- 7323:Sygate 服务器端
- 7511:木马聪明基因开放此端口
- 7588:Sun 许可证管理
- 7597:木马 Quaz 开放此端口
- 7626:木马冰河开放此端口
- 7633:PMDF 管理
- 7674:iMQ SSL 通道
- 7675:iMQ 通道
- 7676:木马 Giscier 开放此端口
- 7720:Med 图象入口
- 7743:Sakura 脚本传递协议
- 7789:木马 ICKiller 开放此端口
- 7797:Propel 连接器端口
- 7798:Propel 编码器端口
- 8000:腾讯 QQ 服务器端开放此端口
- 8001:VCOM 通道
- 8007:Apache(类似 iis)jServ 协议 1.x
- 8008:HTTP Alternate
- 8009:Apache(类似 iis)JServ 协议 1.3
- 8010:Wingate 代理开放此端口
- 8011:木马 way2.4 开放此端口
- 8022:OA-系统
- 8080:WWW 代理开放此端口
- 8081:ICECap 控制台
- 8082:BlackIce(防止黑客软件)警报发送到此端口
- 8118:Privoxy HTTP 代理
- 8121:Apollo 数据端口
- 8122:Apollo 软件管理端口
- 8181:Imail
- 8225:木马灰鸽子开放此端口
- 8311:木马初恋情人开放此端口
- 8351:服务器寻找
- 8416:eSpeech Session 协议
- 8417:eSpeech RTP 协议
- 8473:虚拟点对点
- 8668:网络地址转换
- 8786:Message 客户端
- 8787:Message 服务器
- 8954:Cumulus 管理端口
- 9000:CS 监听
- 9001:ETL 服务管理
- 9002:动态 id 验证
- 9021:Pangolin 验证
- 9022:PrivateArk 远程代理
- 9023:安全网络登陆-1
- 9024:安全网络登陆-2
- 9025:安全网络登陆-3
- 9026:安全网络登陆-4
- 9101:Bacula 控制器
- 9102:Bacula 文件后台
- 9103:Bacula 存储邮件后台
- 9111:DragonIDS 控制台
- 9217:FSC 通讯端口
- 9281:软件传送端口 1
- 9282:软件传送端口 2
- 9346:C 技术监听
- 9400:木马 Incommand 1.0 开放此端口
- 9401:木马 Incommand 1.0 开放此端口
- 9402:木马 Incommand 1.0 开放此端口
- 9594:信息系统
- 9595:Ping Discovery 服务
- 9800:WebDav 源端口
- 9801:Sakura 脚本转移协议-2
- 9802:WebDAV Source TLS/SSL
- 9872:木马 Portal of Doom 开放此端口
- 9873:木马 Portal of Doom 开放此端口
- 9874:木马 Portal of Doom 开放此端口
- 9875:木马 Portal of Doom 开放此端口
- 9899:木马 InIkiller 开放此端口
- 9909:域名时间
- 9911:SYPECom 传送协议
- 9989:木马 iNi-Killer 开放此端口
- 9990:OSM Applet 程序服务器
- 9991:OSM 事件服务器
- 10000:网络数据管理协议
- 10001:SCP 构造端口
- 10005:安全远程登陆
- 10008:Octopus 多路器
- 10067:木马 iNi-Killer 开放此端口
- 10113:NetIQ 端点
- 10115:NetIQ 端点
- 10116:NetIQVoIP 鉴定器
- 10167:木马 iNi-Killer 开放此端口
- 11000:木马 SennaSpy 开放此端口
- 11113:金山词霸开放此端口
- 11233:木马 Progenic trojan 开放此端口
- 12076:木马 Telecommando 开放此端口
- 12223:木马 Hack’99 KeyLogger 开放此端口
- 12345:木马 NetBus1.60/1.70、GabanBus 开放此端口
- 12346:木马 NetBus1.60/1.70、GabanBus 开放此端口
- 12361:木马 Whack-a-mole 开放此端口
- 13223:PowWow 客户端，是 Tribal Voice 的聊天程序
- 13224:PowWow 服务器，是 Tribal Voice 的聊天程序
- 16959:木马 Subseven 开放此端口
- 16969:木马 Priority 开放此端口
- 17027:外向连接
- 19191:木马蓝色火焰开放此端口
- 20000:木马 Millennium 开放此端口
- 20001:木马 Millennium 开放此端口
- 20034:木马 NetBus Pro 开放此端口
- 21554:木马 GirlFriend 开放此端口
- 22222:木马 Prosiak 开放此端口
- 23444:木马网络公牛开放此端口
- 23456:木马 Evil FTP、Ugly FTP 开放此端口
- 25793:Vocaltec 地址服务器
- 26262:K3 软件-服务器
- 26263:K3 软件客户端
- 26274:木马 Delta 开放此端口
- 27374:木马 Subseven 2.1 开放此端口
- 30100:木马 NetSphere 开放此端口
- 30129:木马 Masters Paradise 开放此端口
- 30303:木马 Socket23 开放此端口
- 30999:木马 Kuang 开放此端口
- 31337:木马 BO(Back Orifice)开放此端口
- 31338:木马 BO(Back Orifice)，DeepBO 开放此端口
- 31339:木马 NetSpy DK 开放此端口
- 31666:木马 BOWhack 开放此端口
- 31789:Hack-a-tack
- 32770:sun solaris RPC 服务开放此端口
- 33333:木马 Prosiak 开放此端口
- 33434:路由跟踪
- 34324:木马 Tiny Telnet Server、BigGluck、TN 开放此端口
- 36865:KastenX 软件端口
- 38201:Galaxy7 软件数据通道
- 39681:TurboNote 默认端口
- 40412:木马 The Spy 开放此端口
- 40421:木马 Masters Paradise 开放此端口
- 40422:木马 Masters Paradise 开放此端口
- 40423:木马 Masters Paradise 开放此端口
- 40426:木马 Masters Paradise 开放此端口
- 40843:CSCC 防火墙
- 43210:木马 SchoolBus 1.0/2.0 开放此端口
- 43190:IP-PROVISION
- 44321:PCP 服务器(pmcd)
- 44322:PCP 服务器(pmcd)代理
- 44334:微型个人防火墙端口
- 44442:ColdFusion 软件端口
- 44443:ColdFusion 软件端口
- 44445:木马 Happypig 开放此端口
- 45576:E 代时光专业代理开放此端口
- 47262:木马 Delta 开放此端口
- 47624:Direct Play 服务器
- 47806:ALC 协议
- 48003:Nimbus 网关
- 50505木马 Sockets de Troie 开放此端口
- 50766木马 Fore 开放此端口
- 53001木马 Remote Windows Shutdown 开放此端口
- 54320木马 bo2000 开放此端口
- 54321木马 SchoolBus 1.0/2.0 开放此端口
- 61466木马 Telecommando 开放此端口
- 65000木马 Devil 1.03 开放此端口
- 65301PC Anywhere 软件开放端口
