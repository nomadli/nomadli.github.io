---
layout: post
title:  "IOS 非越狱插件开发"
date:   2016-01-04 11:36:00
categories: IOS
excerpt: IOS 非越狱插件开发。
---

* content
{:toc}

## MachOView
查看Mach-O文件结构工具，Load Commands段标记加载哪些动态库。

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