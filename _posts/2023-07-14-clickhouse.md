---
layout:         post
title:          clickhouse
subtitle:       elecclickhousetron
date:           2023-06-27 14:25:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
catalog:        true
tags:
        - other
---

* content
{:toc}

## 表引擎
- TinyLog 单机只读, 按列存储为单独的压缩文件, 不支持多写会损坏,不支持索引
- Memory 读写无锁,无索引,无压缩
- Merge 读引擎,并发读取其它本地表
- MergeTree 单机、巨量数据插入、按规则合并而非UPDATE、按主键排序、可主键分区、支持副本、支持数据采样建表
- ReplicatedMergeTree 比MergeTree多了定时删除相同主键的重复项
- SummingMergeTree 比MergeTree多了定时合并相同主键的重复项,数值相加,非数值使用旧值
- Distribute 分布式不存储数据，在集群中进行分布式并行查询

## 表分区
- 分区是按某列的条件分目录存储
- 如按时间分区
- 加速查询 wherer

## 分片
- 集群中的一个实例节点
- 将一份数据分片加速查询

## 文件系统
- table名/分区(partition)名/xxxx.xx
    - checksums.txt:校验文件,二进制保存其它文件size及size哈希值,校验文件的完整性和正确性
    - columns.txt:  明文存储列字段信息
    - count.txt:    明文存储计数,记录当前数据分区目录下数据的总行数
    - primary.idx:  主索引二进制存储稀疏索引
    - [Column].bin: 压缩存储数据,默认LZ4压缩
    - [Column].mrk: 二进制存储列字段标记,保存bin数据的偏移量
    - [Column].mrk2:自定义大小的索引间隔,标记文件会以.mrk2命名
    - partition.dat、minmax_[Column].id:使用分区会额外生产partition.dat与minmax索引文件，它们均用二进制格式存储。partition.dat用于保存当前分区下分区表达式最终生成的值；而minmax索引用于记录当前分区下分区字段对应原始数据的最小和最大值。在这些分区索引的作用下，进行数据查询能够快速跳过不必要的数据分区目录，从而减少最终需要扫描的数据范围。
    - skp_idx_[Column].idx、skp_idx_[Column].mrk: 声明了二级索引就会额外生成相应的二级索引与标记文件，它们同样也使用二进制存储
