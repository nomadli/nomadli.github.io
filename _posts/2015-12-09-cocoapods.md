---
layout: post
title:  "CocoaPods"
date:   2015-12-09 15:13:00
categories: IOS
excerpt: CocoaPods使用。
---

* content
{:toc}

## 安装与升级  
- sudo gem install cocoapods  

## 使用  
- touch Podfile、edit Podfile  
- pod install  
- pod update

## 指定平台和SDK版本
- platform :ios, 'x.x.x' ios平台Deployment Target是x.x.x

## 添加依赖库  
- pod '****' 使用最新版本
- pod '****', 'x.x.x'     只使用x.x.x版本
- pod '****', '> x.x.x'	   使用高于x.x.x版本
- pod '****', '>= x.x.x'  使用高于或等于x.x.x版本
- pod '****', '< x.x.x'   使用小于x.x.x版本
- pod '****', '<= x.x.x'  使用小于等于x.x.x版本
- pod '****', '~> x.x.x'  使用大于等于x.x.x但小于x.x的版本

## 指定target  
- link_with 'target1','target2'.... target1、target2、使用相同的依赖库  
- target:'target1' do .... end 不同的target使用不同的依赖库  

## 库列表
- pod search name 查找项目是否在库中
- pod setup 更新项目列表









