---
layout:         post
title:          java native
subtitle:       java native 笔记
date:           2019-03-13 17:54:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
catalog:        true
tags:
        - android
---

* content
{:toc}

## 开发步骤
- public class A {public static native String hello(String name);}
- javac src/com/study/jnilearn/A.java -d ./bin
- javah -jni -classpath ./bin [-d ./jni | -o ./jni/A.h] com.nomdli.A
- 实现.h头文件中的函数
- 编译成动态库(*.dll *.so .jnilib)
- static{System.loadLibrary("A");} 或 static{System.load("/x/x/A.so");}
