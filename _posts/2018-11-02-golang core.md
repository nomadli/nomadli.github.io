---
layout:         post
title:          GO 内核
subtitle:       GO 源码剖析
date:           2018-11-02 15:26:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
catalog:        true
tags:
        - golang
---

* content
{:toc}

# 分析常用命令
- go build -gcflags "-N -l" xx.go 禁止优化及内联
- go tool objdump xxx  转汇编
- gdb 单步调试汇编

# golang 内核
- asm_amd64.s 可执行程序入口地址
- runtime.schedinit 初始调度器(虚拟线程),GOMAXPROCS最大256
- G.sched存储栈地址、程序计数器、寄存器等
- runtime.newproc 创建goroutine
- runtime.mstart 启动主goroutine

# M 线程
- void newm(void (*fn)(void), P *p) 创建线程, p1=runtime.mstart, p2=某个调度器
- acquirep 获取调度器
- releasep 释放调度器
- schedule 开始调度

# P 调度器
- Sched结构保存调度器
    - pidle 保存空闲的调度器
- runqget 获取当前调度器待执行协程
- findrunnable 获取其它调度器的协程
- wakep 通知别的线程调度器执行等待的协程
- execute 执行协程

# G 协程
- sched 保存程序栈、代码计数器、寄存器等
- runtime.park 放弃cpu,状态waiting
- runtime.ready 异步调用完成,回到状态runnable
- runtime.gosched 放弃cpu,状态runnable
- entersyscall 进入系统调用,调度器会被从当前线程中暂停,并将调度器中的其它协程移到别的调度器
- exitsyscall 系统调用结束,回到状态runnable
- runtime.mcall保存协程cpu状态
- runtime.gogocall恢复cpu状态