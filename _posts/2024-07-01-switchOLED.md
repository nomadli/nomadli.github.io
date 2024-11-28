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

## 名词
- NVIDIA Tegra X1   cpu
- Horizon OS        Nintendo Switch 操作系统
- OFW               Original Firmware 原版固件
- Spacecraft-NX     硬件破解固件 https://github.com/Spacecraft-NX/firmware 0.2后支持OLED
- hwfly-nx          基于Spacecraft-NX https://github.com/hwfly-nx
- Hekate            自定义bootloader https://github.com/CTCaer/hekate
- Fusee             转用于Atmosphere的bootloader
- CFW               Custom Firmware 自制系统固件 [Atmosphere](https://github.com/Atmosphere-NX/Atmosphere)
- Homebrew          自定义软件管理
- NRO               Homebrew 程序包
- NAND              内存存储 emmc
- NSP               Nintendo Submission Package eShop可安装的程序包
- NSZ               压缩后的NSP
- XCI               NX Card Image dump 卡带中的文件
- XCZ               压缩后的XCI
- prodinfo          机器识别信息,cpu安全区备份

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
    - CFW (SYSNAND) 原始系统加载破解
    - CFW (EMUMMC) SD卡虚拟系统加载破解
    - bootloader -> hekate_ipl.ini 启动菜单配置
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

## [switch系统](https://switchbrew.org/wiki/xxx)
- boot 内置系统模块, 读取引导分区0:0x00C000的BCT(bad block table)
- boot调用PM内置模块, 硬编码启动boot2 --> 命令 pm: shell NotifyBootCompleted
- boot2 是第一个非内置的系统模块
- close power -> volume down + volume up -> push power -> up power -> show -> up volume 
- close power -> volume down -> push power -> up volume down -> volume up -> up power -> show -> up volume up

## 游戏校验
- 主机检查是否可以联网
    - 系统定时访问ctest.cdn.nintendo.net
    - 检查http头X-Organization:Nintendo
    - tls使用每个设备唯一的私钥与服务器通信, 私钥被CPU加密模块保护
- 获取设备授权令牌 - 是否被ban
    - Device AUTHorization 服务器 dauth-lp1.ndas.srv.nintendo.net
    - http get /challenge  key_generation=主密钥修订版号
    - 返回 {"challenge": string, "data": base-64 string} 
    - data为对称加密AES密钥, 由CPU加密模块加入密钥槽
    - http get /device_auth_token?challenge=%s&client_id=%016x&key_generation=%d&system_version=%s&mac=(前面参数的AES-128 CMAC)
    - 返回token或者错误被禁止
- 授权登陆Nintendo的账户
    - api.accounts.nintendo.com bog-standard oauth authorization登录协议
- 取得当前游戏的授权令牌
    - Application AUTHorization aauth-lp1.ndas.srv.nintendo.net
    - /application_auth_token 利用第二步的令牌:device_auth_token
        - gamecard: application_id=%016llx&application_version=%08x&device_auth_token=%.*s&media_type=GAMECARD&cert=%.*s
        - digital: application_id=%016llx&application_version=%08x&device_auth_token=%.*s&media_type=DIGITAL&cert=%.*s&cert_key=%.*s
    - 获取游戏授权令牌或错误
        - gamecard 证书 banned cert is 0x1F727C — 2124-4025
