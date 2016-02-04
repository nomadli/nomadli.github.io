---
layout: post
title:  "IOS 越狱插件开发"
date:   2016-01-04 11:36:00
categories: IOS
excerpt: IOS 越狱插件开发 Theos Tweak。
---

* content
{:toc}

## 地址
1. [theos](https://github.com/DHowett/theos) 类IDE
2. [lidi](git://git.saurik.com/ldid.git) 签名
3. dkpg MacPorts源安装 打包生成deb
4. [libsubstrate.dylib](http://apt.saurik.com/debs/mobilesubstrate_0.9.5001_iphoneos-arm.deb) 越狱hook库

## 环境搭建
1. command line tools 安装
2. MacPorts 安装
3. theos 安装在/opt/theos
4. export THEOS=/opt/theos
5. ldid 安装在$THEOS/bin/ldid
6. sudo port install dpkg
7. libsubstrate.dylib 安装 8 - 11 步骤
8. ar p *.deb data.tar.lzma > substrate.tar.lzma
9. tar -xvf substrate.tar --strip-components 3 ./Library/Frameworks/CydiaSubstrate.framework/
10. ln -s ../CydiaSubstrate.framework/Headers/CydiaSubstrate.h include/substrate.h
11. ln -s ../CydiaSubstrate.framework/CydiaSubstrate lib/libsubstrate.dylib
12. ios 头文件 在$THEOS/include

## 使用步骤
1. $THEOS/bin/nic.pl 工程选项 2 - 6 项
2. application    		普通app xcode就可以
3. library        		dylib或者Bundle
4. preference_bundle  	Preference插件,在Settings.app里
5. tool           		无界面工具命令行
6. tweak          		hook插件
7. MobileSubstrate Bundle filter 选择tweak后这个限制hook哪个app，默认com.apple.springboard
8. 在Tweak.xm文件中进行hook
9. %hook 后跟要hook的类 %end表示结束
10. %orig 表示调用原函数 %orig(arg1,arg2,....)
11. %log()输出日志到syslog
12. %group 将hook分组 与%end成对 默认%group_ungrouped组
13. %init()初始化指定组，不指定初始化默认组
14. %ctor{} 初始化组或MSHookFunction
15. %new 添加新函数
16. %c 等同于objc_getClass()
17. 在markfile中 工程名_FRAMEWORKS = UIKit 包含要用的库
18. make、package、install