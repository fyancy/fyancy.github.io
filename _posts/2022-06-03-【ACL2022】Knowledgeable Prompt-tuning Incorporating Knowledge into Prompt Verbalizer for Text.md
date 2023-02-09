原文链接: https://arxiv.org/abs/2108.02035
本文参考了舍友的一篇知乎链接：https://zhuanlan.zhihu.com/p/398009000
希望大家也多多支持~
## intro
这是一篇清华刘知远老师组在arxiv上放出来了Prompt-tuning相关的新工作，本文是promp应用于文本分类的一篇工作，应用一种基于外部知识库的 prompt tuning（knowledgeable prompt tuning，简称kpt）,
KPT包含以下三个步骤：
1. 标签词的扩展
2. 扩展标签词的去噪
3. 语言表达器的使用
在few shot 与zeroshot任务中与不错的表现，具体方法如下：

![image](https://img-blog.csdnimg.cn/img_convert/bdac382c7d80122c181e3a9c8284e1e4.png)
从总体思路来说，作者使用外部kb为每个标签生成一组扩展标签单词（每个类生成100多个相关的标签词），并根据Prompt-tuning的核心思想：
  1. 在输入x中插入文本片段（前缀），构建出mask 1 token的模板（Pattern）
2. 在输出y，将label 用一个语言表达器（verbalizer）映射至lablel的扩展词上
3. 问题转化成输入为模板，输出为label related word，且分类问题转化为掩码语言建模问题
模板Xp的样例如下
![image](https://img-blog.csdnimg.cn/img_convert/a178d9bea5f976dc29e1ab59e7fa8e4f.png)
利用MLM模型预测【mask】位置单词，通过预训练模型M计算出 扩展标签词集 中的每个标签词v分别放入mask token位置时的概率
![image](https://img-blog.csdnimg.cn/img_convert/86564176fb5cc1fd2e399a8a03a3c136.png)
最后通过g函数将空间从标签词概率映射到标签概率：
![image](https://img-blog.csdnimg.cn/img_convert/3510ef691d6f992ecf86313770df7f02.png)
这样也就完成了分类，基于上述主思想，作者提出了一种上下文的校准方法消除扩展词v中的噪声。并探讨了利用扩展v的普通平均和加权平均方法。

## Method
具体讲，KPT分为构建，细化，利用三部分：

#### 1. 构建
重点是如何在主题分类和情绪分类中引入外部知识构建扩展标签词。对于主题分类，利用concept net与word net等方法引入相关词，通过边缘表示相关性筛选。情绪这种二元分类，作者引入前人构建的情感字典获得扩展词，最终构建扩展标签词词汇表，示例如下：
![image](https://img-blog.csdnimg.cn/img_convert/54eaf2e4d5ddb7afbd20e3cf10bc05db.png)
#### 2.细化，
作者将细化分为两个场景,zero shot&few shot.
#### zero-shot场景
面临以下三个问题
1. 知识库中得到的扩展词，并不在PLM的单词空间中（out-of-vocabulary）
2. PLM中的稀有词，概率预测往往不准确
3. 标签词的先验分布具有巨大的偏差

**对于第1个问题：**

本文简单的将词拆分成逐token的多个部分，并用PLM逐token预测的平均概率，作为整个词的概率。

**对于第2个问题：**

对于一些稀有词，PLM预测的概率不准确（其实是不稳定），故最好在标签扩展单词表中删去这些稀有词。本文使用MLM去预测句子上下文中这个单词的概率，即如下面这个概率的期望：

![\[公式\]](https://img-blog.csdnimg.cn/80e40ac7ab8e450897b983332b8e1959.png)
xp是模板，即在这个模板下，通过MLM模型预测【mask】为标签扩展词的条件概率的期望。然而这样的概率分布很难直接估计，本文作者假定了一个小尺寸的未标注的support集C作为上下文，并假设c中的样本均符合均匀分布，则上下文的分布为：

![!\[\[公式\]\](https://img-blog.csdnimg.cn/d863d5e18f824bfea0da4d5fbd109ef6.png](https://img-blog.csdnimg.cn/5cb7575ce0244e8ba6b02aa8635e049b.png)
最后我们删去那些概率低于设定阈值的扩展词。

**对于第3个问题：**

无论输入句子的标签如何，但有一些标签词天然地更不可能被预测到，这是由于标签词的先验分布具有很大差异。本文的解决方案，仍然是利用标签词的上下文先验分布来校准预测的分布。我理解是对预测概率（分子）与上下文分布（分母）做了一个对齐操作。
![\[公式\]](https://img-blog.csdnimg.cn/8c6479dd34f246ceb77124355bfbe67a.png)


#### 对于few-shot场景
在few-shot中，因为有少量的标注数据，所以去噪更容易。对于每个标签词，我们为其分配一个可学习的权重参数，然后再将其归一化，得到：

![\[公式\]](https://img-blog.csdnimg.cn/31667397b7f34c12ae086764ac96ced9.png)
在few-shot情况下，我们不需要进行校准，因为训练过程中这个参数会被训练到所需的范围。

## 3.细化
细化的是语言表达器(verbalizer)的使用，同样分为两个场景
在zero-shot情况下
我们简单地认为扩展词中每个词对于预测标签的贡献相同，因此我们对其进行简单平均，并用预测分数的均值作为该标签的预测分数，最后取出预测分数最大的类别，作为最后的结果。
![\[公式\]](https://img-blog.csdnimg.cn/ea5afdcd7e6244d0bb77ef59d5a244be.png)
在few-shot情况下
我们既然已经得到了一个权重参数，我们将其视作扩展词中每个词对于预测标签的贡献度，因此我们将其进行加权平均。
![\[公式\]](https://img-blog.csdnimg.cn/79a30c6dbc7f48dbb418c862fa5550d8.png)
其中
![\[公式\]](https://img-blog.csdnimg.cn/203508e22cd44eaf8b6cea8c99de6a29.png)

## Experiment
本文使用的预训练语言模型是RoBERTa（large）。每个数据集都手工设置了4个不同的模板，如对于，IMDB数据集
![在这里插入图片描述](https://img-blog.csdnimg.cn/038a0f9e420743219ccf8ea9bab522c4.png)
然后对比实验包括：
Prompt-tuning (PT)
Prompt-tuning + Contextualized Calibration(PT + CC)
Fine-tuning (FT)
其中第二个是传统的Prompt-tuning加上本文使用的上下文校准。

第三个则是简单的微调。

实验结果如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/9129a096203348569b04ec40860bf4ac.png)
zero-shot实验结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/28683499c1b143fab90561563db8858c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBASm9rZXJfRERfTkxQ,size_19,color_FFFFFF,t_70,g_se,x_16)
few-shot实验结果

KPT方法的其中一个显著优点是，由于引入了外部知识，因此生成的标签扩展词，是多粒度、多角度的。下图展示了一个示例：
![在这里插入图片描述](https://img-blog.csdnimg.cn/17245ea747204aa7895584a2deea11ed.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBASm9rZXJfRERfTkxQ,size_9,color_FFFFFF,t_70,g_se,x_16)
可以看到，对于政治主题（左），KPT方法生成了“diplomatic”（外交）, “republic”（共和）,“parliament”（议会）等多个主题的扩展词，证明了这一观点


**个人总结：** 其实依旧是基于之前工作的小创新，思路是数据增强+去噪，不过数据增强在于verbalizer对于label space至word space的映射，引入外部的扩展标签词集，辅助分类，去噪并不新颖，就是在细化两个场景，zeroshot滤掉扩展标签词集的低频词，并上下文校验。fewshot则是引入可学习权值，减小噪声影响。

---
