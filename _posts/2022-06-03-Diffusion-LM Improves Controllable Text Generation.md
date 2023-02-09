---

layout:     post
title:      "In-Context Learning（上下文学习）相关分享"
subtitle:   大模型相关
date:       2023-02-09 18:00:00
author:     "Kabi"
tags:

  - 大模型
  - In Context learning
---
详见
https://zhuanlan.zhihu.com/p/603650082/edit

## 1. 前言

随着大模型（GPT3，Instruction GPT，ChatGPT）的横空出世，如何更高效地提示大模型也成了学术界与工业界的关注，因此In-context learning的方法在NLP领域十分火热。
从时间线上看，它的演变历程大约是从Prompt learning（2021年初） 到 Demonstration learning （2021年底） 再到 In-cotnext learning（2022年初），但从方法原理上，他们却有很多相似之处。
本文对一篇有代表性的in-context learning论文：Rethinking the Role of Demonstrations: What Makes In-Context Learning Work? 进行阅读，之后我也会做其他ICL论文的阅读笔记。

## 2. Diffusion Models for Continuous Domains

在这篇综述论文https://arxiv.org/pdf/2301.00234.pdf 给出了详细的定义：
​


In Context Learning（ICL）的关键思想是从类比中学习。上图给出了一个描述语言模型如何使用ICL进行决策的例子。首先，ICL需要一些示例来形成一个演示上下文。这些示例通常是用自然语言模板编写的。然后ICL将查询的问题（即你需要预测标签的input）和一个上下文演示（一些相关的cases）连接在一起，形成带有提示的输入，并将其输入到语言模型中进行预测。
值得注意的是，与需要使用反向梯度更新模型参数的训练阶段的监督学习不同，ICL不需要参数更新，并直接对预先训练好的语言模型进行预测（这是与prompt，传统demonstration learning不同的地方，ICL不需要在下游P-tuning或Fine-tuning）。我们希望该模型学习隐藏在演示中的模式，并据此做出正确的预测。
详细介绍见论文

## 3. Diffusion-LM: Continuous Diffusion Language Modeling

作者对标准的扩散模型进行了部分修改。

### 3.1 End-to-end Training

为了将连续的扩散模型运用到离散的文本，定义embedding函数$EMB(w_{i})$将每一个词语映射为向量。

![c2859cc4729d40efb319ae101bb1c4e5](https://user-images.githubusercontent.com/47687248/171830401-c7c307aa-ee4c-4a46-a804-6a88820919d6.png)

在上图中，在forward process中，添加马尔可夫变换使得将离散的词语$w$映射为$x_{0}$, $q_{\phi}(x_{0}|w)=N(EMB(w),\sigma_{0}I)$; 在reverse process中，添加了可训练的rouding step，$p_{\theta}(w|x_{0})=\Pi_{i=1}^{n}p_{\theta}(w_{i}|x_{i})$, 其中$p_{\theta}(w_{i}|x_{i})$是softmax分布，训练目标如下所示：

![19295bddcc3c4fb7b2e6bcf88a504297](https://user-images.githubusercontent.com/47687248/171830479-f82a4e78-a366-4184-b50d-cb27269b586c.png)

### 3.2  Reducing Rounding Errors

作者在这里设计了方法来减少Rounding Errors，详见论文。

## 4. Decoding and Controllable Generation with Diffusion-LM

### 4.1 Controllable Text Generation

![bee8e0276e7444f8844360104bcd68e5](https://user-images.githubusercontent.com/47687248/171830744-7f92c6bf-c27f-4657-a0ef-a6cb9c37939a.png)

在训练好Diffusion-LM后，作者设计了plug-and-play的机制来控制Diffusion-LM。相比于直接控制的离散的text，作者在由Diffusion-LM生成的连续的隐变量$x_{0:T}$上进行控制。

![3e3fad5b41d3404a8af86780d1dc910d](https://user-images.githubusercontent.com/47687248/171830911-7d7c320c-97c7-42cd-8e6d-542532b7553a.png)

同时为了生成流利的文本，设计了其他的训练目标。

### 4.2 同时为了生成流利的文本，设计了其他的训练目标。

作者在解码的过程中使用了 Minimum Bayes Risk Decoding

## 5. 实验结果

可以看到相比PPLM，FUDGE模型，Diffusion-LM表现优异。详细实验分析见论文。

![d2bb23b04cd44a8eb458c163125dc2fb](https://user-images.githubusercontent.com/47687248/171831225-d6b36950-24c5-47de-a747-ec12fd388ec7.png)

如下表Diffusion-LM在属性组合控制上表现优异

![aac543d7b07049b9b4eceb381cefe576](https://user-images.githubusercontent.com/47687248/171831260-6d5597cf-0c96-4713-b000-0b93f48c0a28.png)

## 6. 总结

论文提出Diffusion-LM，在复杂且细粒度的控制上达到了优异的效果。
但Diffusion-LMs仍然存在缺点：
1. 较高的perplexity
2. 解码的速度相对较慢
3. 训练较难收敛
