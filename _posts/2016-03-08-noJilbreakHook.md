---
layout: post
title:  "IOS 非越狱插件开发"
date:   2016-01-04 11:36:00
categories: IOS
excerpt: IOS 非越狱插件开发。
---

* content
{:toc}

## file otool
1. 解压ipa
2. file 可执行文件 查看包含多少cpu代码
3. otool -l 可执行文件 | grep crypt 查看是否加密

## dumpdecrypted
1. 解密ipa，在运行时dump代码 [github](https://github.com/stefanesser/dumpdecrypted)
2. 越狱机器 ssh链接 运行app 执行 pa ax 得到app 地址
3. 查看app infp.plist 获取bundle id
4. 取app得沙盒目录 [[NSClassFromString(@"LSApplicationProxy") applicationProxyForIdentifier:bundleID] dataContainerURL];
5. 把dumpdecrypted.dylib拷贝到app沙盒tmp目录
6. 执行DYLD_INSERT_LIBRARIES=dumpdecrypted.dylib app目录/可执行文件
7. 只会把当前架构的代码脱壳 lipo app -thin armv7 -output app_armv7 抽取当前架构代码，不再是富文件
8. 或者直接在pp助手等越狱应用中下载，就不用自己解密了

## MachOView
查看Mach-O文件结构工具，Load Commands段标记加载哪些动态库。

## iosopendev
1. 各种模板 http://iosopendev.com
2. brew install ldid dpkg
3. export THEOS=/opt/theos
4.  git clone git://git.saurik.com/ldid.git
5. cd ldid
6. git submodule update --init
7. ./make.sh
8. cp -f ./ldid $THEOS/bin/ldid

## yololib
将自定义动态库添加进Load Commands字段
https://github.com/KJCracks/yololib
./yololib AppBinary self.dylib

## Hook
1. class-dump 了解app 找到需要hook的函数
2. ida 反汇编查看某函数具体逻辑
3. app不会调用自定义动态库中的函数，只会触发动态库初始化，因此添加初始化函数 __attribute__((constructor)) static void entry（）{}
4. https://github.com/rpetrich/CaptainHook 实现了动态库中hook的宏，
5. CHDeclareClass宏 声明想hook的类
6. 在entry中用CHLoadClass 或 CHLoadLateClass加载3中申明的class
7. CHMethod hook 类的方法 格式为：参数个数、返回类型、类名、selector、selector 类型、参数...
8. CHSuper 调用原来的函数
9. CHClassHook注册hook函数 格式：参数个数、返回类型、类名、selector

## 签名
1. 保证app文件夹下有embedded.mobileprovision文件，证书
2. 编辑application-identifier 把TeamID修改为自己的
3. codesign -f -s "iPhone Developer: dzym79@qq.com(xxxxxxxx)" xxx.app/self.dylib
4. codesign -f -s "iPhone Developer: dzym79@qq.com(xxxxxxxx)" --entitlements Entitlements.plist xxx.app
5. xcrun -sdk iphoneos PackageApplication -v xxx.app -o xxx.ipa

## 我破解魔力小孩数学实验
1. xcode7

