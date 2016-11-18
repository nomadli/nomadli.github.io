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
- pod install --verbose --no-repo-update
- pod update --verbose --no-repo-update
- pod search name 查找项目是否在库中
- pod setup 更新项目列表
- pod repo remove master 清理
- pod spec create '仓库名' 创建podspec配置
- pod lib create '仓库名' 创建仓库模板
- pod lib lint '仓库名'.podspec 验证仓库 --allow-warnings --fail-fast --use-libraries --sources --verbose --subspec="name"
- pod trunk register email name --description='***'注册公共
- pod trunk me 个人信息
- pod trunk add-owner '仓库名' name email
- pod trunk push '仓库名'.podspec 
- pod repo add '仓库名' '仓库地址' 

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
- pod '****', :git => 'https://github.com/**'
- pod '****', :svn => 'https://svn.github.com/**'
- pod '****', :path => '~/xxx/xxx'
- pod '****', :branch => 'xxx'
- pod '****', :tag => 'xxx'
- pod '****', :commit => 'xxx'

## 禁止警告
- inhibit_all_warnings! 全部警告
- pod '****' :inhibit_warnings => true

## 指定target  
- link_with 'target1','target2'.... target1、target2、使用相同的依赖库  
- target:'target1' do .... end 不同的target使用不同的依赖库  

## 编译为动态库
- use_frameworks! 编译为动态库 platform 必须在8.0以上

## 指定地址源
- source http://github.com/...../Specs.git

## svn 命令
- svnadmin create name
- svn co file:///path
- svn rm -m "remove tree or file" http://*
- svn import -m "add tree or file" name http://* 
