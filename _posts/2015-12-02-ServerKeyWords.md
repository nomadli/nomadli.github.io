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

## HPC
- RDMA(Remote Direct Memory Access) 直接内存交换
    - InfiniBand 标准RDMA实现，需要网卡、交换机支持
    - RoCE       通过TCP实现，需要网卡支持
    - iWARP      通过二层网络实现, 需要网卡支持
- 并行文件系统
    - [lustre](git.whamcloud.com/fs/lustre-release.git)
    - [pvfs](http://www.pvfs.org)
- [openhpc](https://github.com/openhpc/ohpc) hpc工具集合
