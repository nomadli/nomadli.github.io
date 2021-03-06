---
layout:         post
title:          深度学习
subtitle:       深度学习
date:           2017-09-22 12:15:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
catalog:        true
tags:
        - other
---

* content
{:toc} 

## 机器学习过程
1.  人提供算法集
2.  人提供训练数据
3.  程序从算法集中找出最佳函数(算法+参数)  

## 神经网络深度学习过程
01. 算法集被神经元代替，每个神经元会被训练，多层神经元组成算法
02. 人决定神经网络的链接方式、选择激活函数
03. 机器学习权重、阈值
04. 权重、阈值、激活函数、网络链接形成复杂函数并输出结果 
05. 因此网络链接方式相当于函数集 


## 神经元
01. 神经元是一个简单函数，输入向量输出单一数值
02. 激活函数为预先定义的非线性函数，输入输出为单一数值
03. 输入分量*权重分量->求和->加阈值->激活函数->输出

## 激活函数
- 整流线性单元 src <= 0 ? 0 : src 可以使网络很深
- S型函数 网络不能很深，否则无法训练出好的结果

## 网络结构
- 完全连接前馈式网络 每层神经元等于输入参数个数，并输入到下一层
- 卷积神经网络
- 循环神经网络 recurrent neural network
- 记忆网络 memory network
- 神经图灵机 neural Turing machine
- 动态记忆网络 dynamic memory network

## 学习方式
01. 梯度下降法 随机参数->是否更优->递归 Adam、Drpout等参数算法
02. 反向传递法
03. 监督学习
04. 强化学习    