---
layout:         post
title:          switch OLDE
subtitle:       switch OLDE
date:           2024-07-01 16:34:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
catalog:        true
tags:
        - other
---

* content
{:toc}

## 硬破启动
- 提示NO SD 表示读取了SD卡playload.bin未找到
- 从Stock(sysnand)内置系统启动删除wifi并设置飞行模式
- SD format mbr exfat
- 可选 DeepSea 整合包
- hekate boot文件命名为 playload.bin 放 / 目录
- 重启进入 hekate
- switch 内置emmc 分为BOOT0、BOOT1、GPP(sys、...、user)
- Tools->Backup emmc 可以忽略user
- emuMMCC->Create emuMMC->SD File
    - Stock(sysnand) 原始系统不加载破解
    - CFW (SYSNAND) 原始系统加载破解∂
    - CFW (EMUMMC) SD卡虚拟系统加载破解
- 关机拔SD卡插电脑
    - 硬件序列号修改
        - cp atmosphere/config_templates/exosphere.ini /
        - blank_prodinfo_emummc 改为 1
    - 屏蔽任天堂服务器
        - atmosphere/config/system_settings.ini enable_dns_mitm = u8!0x1 删除注释符
        - touch atmosphere/hosts/emummc.txt
        - 127.0.0.1 *nintendo.* 127.0.0.1 *nintendo-europe.com  127.0.0.1 *nintendoswitch.* 95.216.149.205 *conntest.nintendowifi.net 95.216.149.205 *ctest.cdn.nintendo.net
- 破解补丁
    - ITotalJustice/patches 解压到 /  
    - bootloader/hekate_ipl.ini [CFW (EMUMMC)]节添加kip1patch=nosigchk
- 金手指
    - gbatemp.net 解压 atmosphere/contents/
