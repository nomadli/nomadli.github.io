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

## [路线图](https://github.com/AMAI-GmbH/AI-Expert-Roadmap)
- https://i.am.ai/roadmap/#disclaimer
- 基础->数据科学家->机器学习->深度学习
- 基础->数据工程师->大数据工程师
- 基础
    - 矩阵、线性代数基础
    - 数据库基础

## 人工智能分类
- 神经网络(Neural Networks)
    - TensorFlow
    - PyTorch
- 决策树(Tree Ensembles)
    - XGboost
    - scikit-learn
- 广义线性模型(Generalized Linear Models)
    - 线性回归
    - 逻辑回归
    - scikit-learn
- 支持向量机(SVM  Support Vector Machines)
    - LIBSVM
    - scikit-learn
- 前后预处理(Pipelines (pre and post processing))
    - scikit-learn

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


## Apple CoreML
![Apple CoreML](../img/apple_coreml.png)
- [入口](https://developer.apple.com/cn/machine-learning)
    - https://developer.apple.com/documentation/coreml
    - https://developer.apple.com/documentation/vision
    - https://developer.apple.com/documentation/naturallanguage
    - https://developer.apple.com/documentation/speech
    - https://developer.apple.com/documentation/soundanalysis
    - https://developer.apple.com/documentation/metalperformanceshadersgraph
- 支持的类型
    - 面部识别的视觉API
    - 自然语言处理API
    - 深度神经网络
    - 循环神经网络
    - 卷积神经网络
    - 机器学习
    - 支持向量机(vector machines)
    - 树集成(trees)
    - 线性模型(General linear models)
- [模型转换工具](https://github.com/apple/coremltools)
- MPS(Metal Performance Shaders) 利用显卡加速数据处理
- Accelerate cpu矢量计算框架 -> vImage
- 图像处理 MPS + vImage->Core Image
- CIFilter 系统自带的理由MPS的图像处理类
- xcrun coremlc compile coreml.mlpackage out_path || $(xcode-select -p)/usr/bin/coremlc ...
- xcrun coremlc generate coreml.mlpackage out_path --language Swift


## state of the art(SOTA) 艺术类的模型
- LaMa 基于傅立叶卷积的分辨率鲁棒的大掩模修复
    - Resolution-Robust Large Mask Inpainting with Fourier Convolutions
    - 基于前馈ResNet类型的修复网络,使用快速傅里叶卷积(结合了对抗损失和高感受野感知损失的多损失组合)和大掩膜生成程序
    - 模型的网络框架是GAN,主体网络结构是ResNet,其中加入了 FFC
- LDM
- ZITS
- MAT
- FcF
- Manga
- anything4 语义生成图
- realistic Vision 1.4 根据语义生成图
- OpenCV2
- Stable Diffusion 1.5
- Stable Diffusion 2.0 语义编辑图
- paint_by_example 语义编辑图
- instruct_pix2pix 语义编辑图

## 开源
- [TVM 将任意模型转为其它模型的开源框架](https://tvm.apache.org)
- [Bringing-Old-Photos-Back-to-Life 微软专业去除老照片划痕、增强脸部](https://github.com/microsoft/Bringing-Old-Photos-Back-to-Life)
- [stable-diffusion-ui 漫画生成](https://github.com/AUTOMATIC1111/stable-diffusion-webui)
- [OpenBLAS](https://github.com/xianyi/OpenBLAS) 线性代数加速库 基于BLAS
- [LAPACK](http://github.com/Reference-LAPACK) 线性代数加速库BLAS