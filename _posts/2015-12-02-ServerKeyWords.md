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

## 需求文档
- 项目背景 业务背景、市场情况、行业情况
- 产品目标 实现指标、解决什么、价值
- 用户分析 痛点、特点
- 核心场景 who where when 目标 动作
- 业务流程 流程图、泳道图、时序图、业务架构图
- 具体要求
- 数据埋点
- 风险评估
- 迭代计划

## 数学键盘
- [数学字体](https://github.com/CyanoHao/OpenType-MATH-TTF)
- [清华字体镜像](https://mirrors.sjtug.sjtu.edu.cn/ctan/fonts)
- [macos数学键盘](https://github.com/kostub/iosMath)
- [C#数学键盘](https://github.com/kashifimran/math-editor)
- [微软在线数学](https://math.microsoft.com/zh)

## 语法工具
- flex
- yacc

## 安装代理根证书
```
HTTP/1.1 200 OK
Content-Type: application/x-x509-ca-cert
Content-Disposition: filename=charles-proxy-ssl-proxying-certificate.pem
Content-Length: 1917
Proxy-Connection: close

-----BEGIN CERTIFICATE-----
MIIFTDCCBDSgAwIBAgIGAXZV03igMA0GCSqGSIb3DQEBCwUAMIGpMTowOAYDVQQDDDFDaGFybGVz
IFByb3h5IENBICgxMiBEZWMgMjAyMCwgbm9tYWRsaS1tYWMubG9jYWwpMSUwIwYDVQQLDBxodHRw
czovL2NoYXJsZXNwcm94eS5jb20vc3NsMREwDwYDVQQKDAhYSzcyIEx0ZDERMA8GA1UEBwwIQXVj
a2xhbmQxETAPBgNVBAgMCEF1Y2tsYW5kMQswCQYDVQQGEwJOWjAgFw0wMDAxMDEwMDAwMDBaGA8y
MDUwMDIwODA3MjEzMVowgakxOjA4BgNVBAMMMUNoYXJsZXMgUHJveHkgQ0EgKDEyIERlYyAyMDIw
LCBub21hZGxpLW1hYy5sb2NhbCkxJTAjBgNVBAsMHGh0dHBzOi8vY2hhcmxlc3Byb3h5LmNvbS9z
c2wxETAPBgNVBAoMCFhLNzIgTHRkMREwDwYDVQQHDAhBdWNrbGFuZDERMA8GA1UECAwIQXVja2xh
bmQxCzAJBgNVBAYTAk5aMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEApN5H38rCutIY
dmZePtq50N/vtIfulcS/Mx0wKJVB01sujrFKelIKn8fXhlPy9YvoQd+kZnEq3RNfrMngo+CKZ4bQ
IzA+mMpyL76z+2NoQiI1O7bT5iW+HNEExGeerK5ZfSfXiYLa7dNWhktNpRY1+ISXvsHUFrNR8JMG
sVpGNxReuIbz5lDscWuwO/7fjSQBDtHWgw+edYCn6EkxcDuszGSaIIG65ZqkLVnU7rnDv/Ltxaqe
Toqq8X7HhuTB+0mVmBYHS+oU/OUkqj8rsGQI48ifFWAYu1tKcIm03kUMHKTQGS3+AL6UabLJp/Gn
2sqo57ipSyeucx7e4EETwmz6twIDAQABo4IBdDCCAXAwDwYDVR0TAQH/BAUwAwEB/zCCASwGCWCG
SAGG+EIBDQSCAR0TggEZVGhpcyBSb290IGNlcnRpZmljYXRlIHdhcyBnZW5lcmF0ZWQgYnkgQ2hh
cmxlcyBQcm94eSBmb3IgU1NMIFByb3h5aW5nLiBJZiB0aGlzIGNlcnRpZmljYXRlIGlzIHBhcnQg
b2YgYSBjZXJ0aWZpY2F0ZSBjaGFpbiwgdGhpcyBtZWFucyB0aGF0IHlvdSdyZSBicm93c2luZyB0
aHJvdWdoIENoYXJsZXMgUHJveHkgd2l0aCBTU0wgUHJveHlpbmcgZW5hYmxlZCBmb3IgdGhpcyB3
ZWJzaXRlLiBQbGVhc2Ugc2VlIGh0dHA6Ly9jaGFybGVzcHJveHkuY29tL3NzbCBmb3IgbW9yZSBp
bmZvcm1hdGlvbi4wDgYDVR0PAQH/BAQDAgIEMB0GA1UdDgQWBBQ4vrYuvJb3rEr/INCsRWKBqBmS
ejANBgkqhkiG9w0BAQsFAAOCAQEAU4UbEAzF/GGVnK13AdcOSdyczC1G1ru4oiBZNXK8QYlLcxFs
rOyWhqTKDjetd2IwQs28sq8tRrHb0fr23pOT+l3nOUG9iawobRElrZkrOASf+MTImIr5dKOLCNY8
RS7G/kWRGhS591bGK1NHOhIBKqaiIOrGuVDTk0pS8L7N0PRUcpmzrvhTbdzufl1NqsLex+XI++z6
f9NT5BN9HkgKytAgfcufzuUlm8OQZ+5ghao2rSy9vweJ+3VCmYewncfvYCq0DisO+CgnoMKw9ED6
h2VKXn+k5iforQ0dTOgu9bjj6KUf8uRnNFRMzEVs4db2U2c+hGtSRnJGpZkUNDaSmg==
-----END CERTIFICATE-----
```
