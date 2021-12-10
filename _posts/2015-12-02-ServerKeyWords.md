---
layout:         post
title:          借鉴的开源链接
subtitle:       可借鉴的链接
date:           2015-12-02 16:58:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
catalog:        true
tags:
        - server
---

* content
{:toc}

## 虚拟打印机
1. [Ghostscript](http://git.ghostscript.com/?p=ghostpdl.git;a=summary)
2. [PDF相关的代码](http://git.ghostscript.com/)
3. [PDFCreator](https://github.com/pdfforge/PDFCreator.git)
4. [pdfwriterformac](https://git.code.sf.net/p/pdfwriterformac/git)

## HPC 高性能计算
- RDMA(Remote Direct Memory Access) 直接内存交换
    - InfiniBand 标准RDMA实现，需要网卡、交换机支持
    - RoCE       通过TCP实现，需要网卡支持
    - iWARP      通过二层网络实现, 需要网卡支持
- 并行文件系统
    - [lustre](git.whamcloud.com/fs/lustre-release.git)
    - [pvfs](http://www.pvfs.org)
- [openhpc](https://github.com/openhpc/ohpc) hpc工具集合

## CTF(Common Trace Format) 跟踪日志数据设计,用于高速日志输出

## UIO
- 驱动程序在用户态空间里实现,不支持DMA、中断等

## IOMMU(Input/Output Memory Management Unit)
- 硬件独立于CPU的MMU
- 作用连接 DMA-capable I/O 和 主存, 将设备访问的虚拟内存转换为实际物理内存
- 为每个直通的设备分配独立的页表，因此不同的直通设备彼此之间相互隔离
- /dev/vfio 设备文件
- container是内核对象,表示一个IOMMU设备,container是IOMMU操作的最小对象

## VFIO
- 把设备I/O、中断、DMA等暴露到用户空间
- 直通的最小单元不再是某个单独的设备了,而是分布在同一个group的所有设备
- 将IOMMU的container模拟出多个iommu_group, 用于虚拟设备.如一块物理网卡虚拟多块网卡,共享container