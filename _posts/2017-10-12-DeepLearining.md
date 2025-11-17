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

## LLVM
- 预训练
    - 数据采集
        - 列举主流URL
        - 过滤有害URL
        - 获取URL内容
        - 语音过滤(english、chinese、...)
        - Gopher过滤(无意义、低信息、暴力、偏见、垃圾文本...)
        - minHash 去重(重复或近似)减少数据冗余,避免过拟合、偏向高频内容、节省计算资源
        - C4(Colossal Clean Crawled Corpus)过滤,数据清洗筛选提取高质量多样化语料,去噪声重复低   效内容
        - Custom Filters 自定义过滤, 筛选特定领域信息
        - PII Removal(Personally Identifiable Information)删除个人信息
    - Tokenization 分词、令牌化;将文本数据表示为token的一维序列
        - 分词过细导致序列过长计算量大
        - 分词过粗导致词汇表爆炸内存占用高
        - BPE(Byte-Pair Encoding)算法平衡词汇表与序列长度
        - WordPiece
        - SentencePiece
    - 词汇表
        - 将同时出现的Token重新合并成一个新的token;重复此步骤
        - 压缩token映射表,重新标号token
    - 数据分片
        - 解决内存与存储的限制
        - 并行加速训练
        - 单分片训练出错只需重新开始该分片
    - 模型架构选择 Transformer自注意力机制
        - 包含自注意力+多头注意力+前馈神经网络+注意力层+FFN等
        - 变体 Decoder-only 适合生成任务单向注意力 Encoder-decoder适合翻译序列到序列任务
        - 层数(L) 12-100+
        - 隐藏层维度(d_model) 768-12288
        - 注意力头数(h) 12-128
    - 训练任务设计
        - 自监督学习;无需人工标注通过文本自身生成监督信号
        - 因果语音建模(CLM)
        - 掩码语音建模(MLM)
        - 混合目标如UniLM
    - 训练执行
        - 分布式训练、并行策略、通信优化
        - 数据加载与预处理
        - 向前传播
        - 反向传播
        - 梯度同步
        - 参数更新
        - 每个分片训练结果采用测试集计算Lost函数将拟合偏离反馈到神经网络参数调整再进行下次训练
        - 训练优化 混合FP16/FP8 放大Loss Scaling防止梯度下溢 激活监测点 内核融合
        - 硬件优化 显存管理、计算加速FlashAttention、Sparsity
    - 预训练结果 Base Model 其回答不可读或有害或只是重复问题;需后训练
- 后训练
    - 将模型从通用知识库转变为可控、安全、可用工具
    - 监督微调(Supervised Fine-Tuning SFT) 数据集是由人工创建的高质量对话
    - 奖励模型(Reward Modeling)训练一个评估生成内容质量的模型,为后续强化学习提供反馈信号
    - 领域适应(Domain Adaptation)在领域相关数据上继续预训练、领域高质量数据后训练
- 问题
    - 幻觉
    - 长记忆
    - 数据计算能力
    - 模型直接使用工具的能力;目前都是在结果输出后使用工具

## RL 强化学习
    - GRPO(group Relative Policy Optimization)分组奖励估计
    - 冷启动采用高质量长思维链的人工生成数据初始化然后开始优化推理能力训练和偏好训练和SFT

## Dify
```yaml
kind: app
version: 0.3.0
app:                                #应用配置的容器节点
  description: '应用描述(可选)'
  icon: 🤖                          #应用图标(可选)
  icon_background: '#FFEAD5'     #图标背景色(可选)
  mode: agent-chat                  #应用模式(必填) 核心交互方式 advanced-chat workflow chatflow
  name: xcode                       #应用名称(必填)
  use_icon_as_answer_icon: false    #是否将应用图标用作回答图标(可选) 默认false

dependencies:                       #依赖项
- current_identifier: null          #当前依赖的唯一标识(可选) 区分不同版本的依赖
  type: marketplace                 #依赖类型(必填) marketplace: 市场插件 custom: 自定义依赖
  value:
    marketplace_plugin_unique_identifier: langgenius/bedrock:0.0.24@d15b21a5f8833d2489cd336798188b929a9f86e1c05f2f4770c05fc3e1e991bf  #依赖的具体值(必填) 自定义为api url
- current_identifier: null
  type: marketplace
  value:
    marketplace_plugin_unique_identifier: langgenius/json_process:0.0.2@dde6d7b676ccdcea89206d29232181a840170c19277d3d978e27cd1e3c92c707
- current_identifier: null
  type: marketplace
  value:
    marketplace_plugin_unique_identifier: junjiem/mcp_sse:0.2.1@53cc613667fcf91dd7208dd5f6d2c8df3c7ff0af8b79e8f3c0a430f1b39bda4c
- current_identifier: null
  type: marketplace
  value:
    marketplace_plugin_unique_identifier: langgenius/regex:0.0.3@257eaab07b70ab1f77a881b870eefee93fc8fd0dd13350077410264f31695039
- current_identifier: null
  type: marketplace
  value:
    marketplace_plugin_unique_identifier: yevanchen/markitdown:0.0.4@776b3e2e930e2ffd28a75bb20fecbe7a020849cf754f86e604acacf1258877f6
model_config:
  agent_mode:
    enabled: true
    max_iteration: 5            #最大推理轮次
    prompt: "coding assistant"  #Agnet角色
    strategy: react             #function_call采用函数调用策略 react动态规划工具调用路径
    tools:                      #可调用工具列表
    - enabled: true
      isDeleted: false
      notAuthor: false
      provider_id: code
      provider_name: code
      provider_type: builtin
      tool_label: 代码解释器
      tool_name: simple_code
      tool_parameters:
        code: ''
        language: ''
    - enabled: true
      isDeleted: false
      notAuthor: false
      provider_id: junjiem/mcp_sse/mcp_sse
      provider_name: junjiem/mcp_sse/mcp_sse
      provider_type: builtin
      tool_label: 获取 MCP 工具列表
      tool_name: mcp_sse_list_tools
      tool_parameters:
        prompts_as_tools: ''
        resources_as_tools: ''
        servers_config: ''
    - enabled: true
      isDeleted: false
      notAuthor: false
      provider_id: junjiem/mcp_sse/mcp_sse
      provider_name: junjiem/mcp_sse/mcp_sse
      provider_type: builtin
      tool_label: 调用 MCP 工具
      tool_name: mcp_sse_call_tool
      tool_parameters:
        arguments: ''
        prompts_as_tools: ''
        resources_as_tools: ''
        servers_config: ''
        tool_name: ''
    - enabled: true
      isDeleted: false
      notAuthor: false
      provider_id: langgenius/json_process/json_process
      provider_name: langgenius/json_process/json_process
      provider_type: builtin
      tool_label: JSON 解析
      tool_name: parse
      tool_parameters:
        content: ''
        ensure_ascii: ''
        json_filter: ''
    - enabled: true
      isDeleted: false
      notAuthor: false
      provider_id: langgenius/json_process/json_process
      provider_name: langgenius/json_process/json_process
      provider_type: builtin
      tool_label: JSON 删除
      tool_name: json_delete
      tool_parameters:
        content: ''
        ensure_ascii: ''
        query: ''
    - enabled: true
      isDeleted: false
      notAuthor: false
      provider_id: langgenius/json_process/json_process
      provider_name: langgenius/json_process/json_process
      provider_type: builtin
      tool_label: JSON 替换
      tool_name: json_replace
      tool_parameters:
        content: ''
        ensure_ascii: ''
        query: ''
        replace_model: ''
        replace_pattern: ''
        replace_value: ''
        value_decode: ''
    - enabled: true
      isDeleted: false
      notAuthor: false
      provider_id: langgenius/json_process/json_process
      provider_name: langgenius/json_process/json_process
      provider_type: builtin
      tool_label: JSON 插入
      tool_name: json_insert
      tool_parameters:
        content: ''
        create_path: ''
        ensure_ascii: ''
        new_value: ''
        query: ''
        value_decode: ''
    - enabled: true
      isDeleted: false
      notAuthor: false
      provider_id: yevanchen/markitdown/markitdown
      provider_name: yevanchen/markitdown/markitdown
      provider_type: builtin
      tool_label: markitdown
      tool_name: markitdown
      tool_parameters:
        files: ''
    - enabled: true
      isDeleted: false
      notAuthor: false
      provider_id: langgenius/regex/regex
      provider_name: langgenius/regex/regex
      provider_type: builtin
      tool_label: 正则表达式内容提取
      tool_name: regex_extract
      tool_parameters:
        content: ''
        expression: ''
  annotation_reply:     #标注回复配置, score_threshold 相似度阈值 embedding_model 向量化模型 storage_mode 存储模式
    enabled: false
  chat_prompt_config: {}        #对话提示词模板 role
  completion_prompt_config: {}  #定义生成式任务的提示模板和生成规则
  dataset_configs:              #配置数据集用于RAG
    datasets:
      datasets: []
    reranking_enable: false
    retrieval_model: multiple
    top_k: 4
  dataset_query_variable: ''    #数据集查询的动态参数
  external_data_tools: []       #定义外部数据工具集成
  file_upload:
    allowed_file_extensions:
    - .JPG
    - .JPEG
    - .PNG
    - .GIF
    - .WEBP
    - .SVG
    - .MP4
    - .MOV
    - .MPEG
    - .WEBM
    allowed_file_types:
    - image
    allowed_file_upload_methods:
    - remote_url
    - local_file
    enabled: true
    image:
      detail: high
      enabled: true
      number_limits: 6
      transfer_methods:
      - remote_url
      - local_file
    number_limits: 6
  model:
    completion_params:
      cross-region: true                # 启用跨区域推理(提升响应速度)
      latest_two_messages_cache_checkpoint: false #缓存最近两次用户与系统的对话消息
      max_tokens: 32768                 # 支持长文本生成(如生成完整报告)
      model_name: Claude 4.0 Opus
      reasoning_budget: 128000          # 限制LLM推理过程的资源消耗
      reasoning_type: false             # 推理策略 none step_by_step critical_thinking multi_hop self_consistency
      response_format: JSON             # Text​ JSON​ Markdown​ HTML​ JSON Schema
      stop: []                          # 生成内容遇到什么字符时停止
      system_cache_checkpoint: false    # 缓存系统级的通用数据 如知识库索引、工具配置、用户偏好
      temperature: 0.5                  # 降低随机性,输出更稳定, 越小幻觉越少
      top_k: 500
      top_p: 0.99                       # 优先选择高概率词, 提升准确性
    mode: chat
    name: anthropic claude
    provider: langgenius/bedrock/bedrock
  more_like_this:                       #否启用类似内容推荐功能
    enabled: false
  opening_statement: ''                 #定义对话开始时的开场白或初始提示
  pre_prompt: ''                        #注入的固定提示 定义角色或全局约束
  prompt_type: structured               #定义提示模板类型,simple(基础) structured(结构化) custom(自定义）)
  retriever_resource:                   #外部数据检索资源 知识库、数据库
    enabled: true
  sensitive_word_avoidance:             #敏感词过滤
    configs: []
    enabled: false
    type: ''
  speech_to_text:                       #配置语音输入功能
    enabled: false
  suggested_questions: []               #预定义建议问题列表，引导用户输入
  suggested_questions_after_answer:     #在生成回答后追加建议问题
    enabled: false
  text_to_speech:                       #配置语音输出功能
    enabled: false
    language: ''
    voice: ''
  user_input_form: []           #自定义用户输入表单
```