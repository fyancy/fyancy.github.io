# 【ACL 2021】《 Defending Pre-trained Language Models from Adversarial Word Substitution Without Performance Sacrifice》阅读笔记

<font color=#999AAA >英文标题：Defending Pre-trained Language Models
from Adversarial Word Substitution Without Performance Sacrifice
Representations
<font color=#999AAA >中文翻译：在对抗性的替换词下保护预训练语言模型而不牺牲性能
<font color=#999AAA >原文链接:  [https://arxiv.org/pdf/2006.03659.pdf](https://arxiv.org/pdf/2006.03659.pdf)
<font color=#999AAA >代码:  [https://github.com/LilyNLP/ADFAR](https://github.com/LilyNLP/ADFAR)


@[TOC](文章目录)

</font>

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">









#  Abstract
预先训练的上下文语言模型(PrLMs)在下游自然语言理解任务中取得了强大的性能提高。然而，PrLMs仍然很容易被对抗性的替换词所愚弄，这是最具挑战性的文本对抗性攻击方法之一。本文提出了一个可以保护性能的框架，具有频率感知随机化的异常检测(ADFAR)。详细地，我们设计了一个辅助的异常检测分类器AD，并采用了一个多任务的学习程序，通过它，PrLMs能够识别对抗性的输入样本。然后，为了防御对抗性的替换词，对那些已识别的对抗性的输入样本应用了一个具有频率识别能力的随机化过程。实验结果表明，ADFAR在各种推理任务中的性能明显优于那些新提出的防御方法。值得注意的是，ADFAR并不会影响PrLMs的整体性能。

# 一、Introduction

预先训练好的语言模型(PrLMs)被广泛采用为各种NLP系统的重要组成部分。然而，作为基于DNN的模型，PrLMs仍然很容易被文本对抗性样本所愚弄。PrLMs的这种脆弱性不断引起潜在的安全问题，因此研究防御技术来帮助PrLMs对抗文本对抗性样本是迫切必要的。
前人提出了不同类型的**文本攻击方法，从字符级单词拼写错，单词级替代，短语级插入和删除，到句子级意译**。由于自然语言的离散性，通过拼写更正和语法纠错，可以很容易地发现和恢复的攻击方法。然而，基于对抗性替换词的攻击方法可以产生高质量和有效的对抗性样本，这仍然很难被现有的方法检测到。因此，对抗性词替代对PrLMs的鲁棒性提出了更大、更深刻的挑战。因此，本文致力于克服对抗性的词替换所带来的挑战。

为此，作者提出了一个保护性能的框架，具有频率感知随机化的异常检测(ADFAR)，以帮助PrLMs在不牺牲性能的情况下防御对抗性单词替换。在推理中引入随机化可以有效地防御对抗性攻击。此外(Mozes等人，2020)表明，通常的对抗样本是用较少出现的同义词代替单词，而PrLMs对频繁出现的词更健壮。因此，我们提出了一个具有频率感知能力的随机化过程来帮助PrLMs抵御对抗性的单词替换。
# 二. Related work ＆ Method
**AWS**
对抗性词替代(AWS)是攻击PrLMs等高级神经模型的最有效的方法之一。在AWS中，攻击者故意用其同义词替换某些词，以误导对目标模型的预测。同时，高质量的对抗性样本应保持语法正确性和语义一致性。为了制作高效和高质量的对抗性样本，攻击者应该首先确定要被干扰的脆弱token，然后选择合适的同义词来替换它们。当前AWS模型采用启发式算法定位句子中脆弱的标记。为了说明，对于给定的样本和目标模型，攻击者迭代地mask token并检查模型的输出。对最终输出日志具有显著影响的token被视为脆弱的。

**防御aws**
对于一般攻击方法，对抗性训练被广泛采用以减轻对抗性效应，但是表明该方法仍然容易受到AWS的攻击。这是因为AWS模型利用动态算法来攻击目标模型，而对抗性训练只涉及一个静态训练集。由于AWS启发式攻击方法在攻击模型时迭代地替换每个单词，直到它成功地改变了模型的输出，因此使用静态策略来防御这种动态过程通常是不同的。相反，动态策略，如随机化，可以更好地解决这个问题。人们还可以观察到，用它们更频繁的替代品替换单词可以更好地减轻对抗性的效果，并保持原始的性能。因此，设计了一种具有频率感知能力的随机化策略来干扰AWS策略。

# 三.Method
## 1.频率感知随机化（FAR）
![在这里插入图片描述](https://img-blog.csdnimg.cn/8314c930260644bfbc516b408882587e.png)
上图与下面的算法给出了频率感知随机化的方法，包括三个步骤。
1. 选择频率较低的稀有单词 和一些随机单词作为替代候选词（这些词有可能为对抗替换的词汇）。
2. 我们选择意义最接近和频率最高的同义词来为每个候选词形成一个同义词集。
3. 每个候选词在它自己的同义词集中被一个随机的同义词所取代。为了量化两个词之间的语义相似性，我们通过两个单词的余弦相似性通来评估。为确定一个单词的频率，我们使用了一个由频率单词存储库提供的频率字典评估。
![在这里插入图片描述](https://img-blog.csdnimg.cn/41d8f473158a4b2bb3054753693e694b.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvbmdndWFudGluZw==,size_16,color_FFFFFF,t_70)
该算法与与上述三个步骤基本一致，其中r是预定义的替代比率
## 2.异常检测（AD）
将FAR过程应用于每个输入，仍然会降低正常样本的预测精度。为了克服这一问题，我们在PrLMs中添加了一个辅助异常检测头，并采用了多任务学习程序，使PrLMs能够对输入文本进行分类，同时区分敌对样本，而不引入额外的模型。在推断中，频率感知随机化只适用于被检测为对抗性的样本。这样非对抗性样本，很大程度上避免了精度的降低不会受到影响。
周等人。（2019）还详细阐述了使用扰动识别来阻止攻击的想法。然而，他们的方法在token级别检测异常，需要两个资源消耗的PrLM进行检测和校正，而我们的方法在句子级检测异常，不需要额外的模型。
## 3 ADFAR总体框架
![在这里插入图片描述](https://img-blog.csdnimg.cn/2b63dfb0eadc45058bb751d39533c53a.png)
### 3.1 训练数据设置
如图2所示，我们结合了对抗性训练和数据增强的想法来构建我们的随机增强的对抗性训练数据。首先，我们使用了一个AWS模型。基于原始训练集生成对抗性样本。根据常见的对抗训练设置，我们将对抗样本与原始样本相结合，形成一个对抗训练集（蓝色）。

其次，为了让PrLMs更好地处理的随机化样本，我们将频率感知随机化（FAR）方法应用于对抗训练集，生成随机化对抗样本。最后，将对抗训练集和随机对抗训练集相结合，形成随机增强对抗训练集。
### 3.2 辅助异常检测器（AAD）
除了原始文本分类器之外，我们还在PrLMs上添加了一个辅助异常检测器（AD）来区分对抗性样本。对于输入句子，PrLMs通过自我注意每个token的上下文信息，并生成一系列上下文嵌入{h0，……hm）。对于文本分类任务，使用h0∈RH作为聚合序列表示。原始文本类字符利用h0通过一个逻辑回归来预测X被标记为类yˆc的概率：
![在这里插入图片描述](https://img-blog.csdnimg.cn/95805543c77f43a0ab528f13851f1b52.png)
如图2所示，原始文本分类器是在随机增强对抗训练集上进行训练的，而异常检测器只在对抗训练集上进行训练
![在这里插入图片描述](https://img-blog.csdnimg.cn/68aced9dce6c404099b531712c84d9e7.png)
### 3.3 训练目标
![在这里插入图片描述](https://img-blog.csdnimg.cn/88e5dba24cb54bcca7d45d6354630713.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvbmdndWFudGluZw==,size_16,color_FFFFFF,t_70)
首先，异常检测器预测了一个输入样本是否是敌对的。如果将输入样本确定为非对抗性样本，则文本分类器（标签A）的输出直接用作其最终预测。
如果输入样本被确定为对抗性样本，则将频率感知随机化过程应用于原始输入样本。然后，将随机样本再次发送到PrLM，并使用文本分类器（标签B）的第二个输出作为其最终预测

作者采用一个多任务学习框架，训练PrLM对输入文本进行分类，同时区分敌对样本。我们设计了两个以最小化交叉熵损失的形式实现的并行训练目标：文本分类的lossc和异常检测的lossd。总损失函数定义为其之和：
![在这里插入图片描述](https://img-blog.csdnimg.cn/f3d8e712d7244f108126f59107e18d12.png)
# 4.Experiment
实验利用bert作为PrLM基线,以TextFooler作为词替换的攻击模型。下表ADFAR和其他防御框架的性能。由于随机化可能会导致结果存在方差，我们报告基于平均5次运行的结果。
### 4.1
![在这里插入图片描述](https://img-blog.csdnimg.cn/382205b993da421290f79c39f1922fc6.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvbmdndWFudGluZw==,size_16,color_FFFFFF,t_70)
orig acc是正常样本和Adv共同训练的预测精度。advacc是模型在面对AWS时的攻击后精度。结果是基于平均5次运行。*

实验结果表明，ADFAR可以有效地帮助PrLM对抗AWS。与DISP和safer的方法相比，ADFAR在对抗性样本中取得了最好的性能。

### 4.2
同时，ADFAR一般并不损害非对抗性样本的性能。在Mr和IMDB等任务上，ADFAR甚至可以提高基线PrLM。如表3所示，与以往的方法相比，ADFAR实现了显著更高的推理速度。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2f3e281b0acd4bdc96f393d96c8f52c3.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvbmdndWFudGluZw==,size_16,color_FFFFFF,t_70)
*参数和推理时间的统计信息。推理时间表示使用一个NVIDIARTX3090的Mr数据集中一个样本的平均推理时间*
### 4.3
因此看看ADFAR在面对其他AWS模型生成的对抗样本时是否表现良好是很重要的。我们利用PWWS(Ren等人，2019)和genetic(Alzantot等人，2018)来进一步研究ADFAR的性能。
![在这里插入图片描述](https://img-blog.csdnimg.cn/7844d02519e24827bbaa40f52f0ba8ea.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvbmdndWFudGluZw==,size_16,color_FFFFFF,t_70)
### 4.4
下表显示了ADFAR利用RoBERTaBASE(Liu等人，2019)和ELECTRA(Clark等人，2020)作为PrLMs的性能。为了提高PrLM的鲁棒性和性能，RoBERTa扩展了更大的语料库和使用更有效的参数，而etra应用GAN风格的架构进行预训练。
![在这里插入图片描述](https://img-blog.csdnimg.cn/ee8c81ff405d49f58954cbe7f3e55bb8.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvbmdndWFudGluZw==,size_16,color_FFFFFF,t_70)

实证结果表明，ADFAR可以进一步提高RoBERTa和ELECTRA的鲁棒性，同时保持了电子的原始性能。
### 4.5
![在这里插入图片描述](https://img-blog.csdnimg.cn/b0e7e60bb2d44a4eafd63873ed9a7313.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvbmdndWFudGluZw==,size_16,color_FFFFFF,t_70)
ADFAR和DISP对异常检测的性能。上表显示了ADFAR和DISP对异常检测的性能。实证结果表明，ADFAR可以更准确地预测，因为其F1得分明显高于DISP。此外，ADFAR有一个更简单的框架，因为其异常检测器与分类器共享相同的PrLM，而DISP需要一个额外的PrLM。结果还表明，目前的AWS攻击策略很容易受到我们的异常检测器的检测，这证明了我们AD的功能。

### 4.6
![在这里插入图片描述](https://img-blog.csdnimg.cn/a208deefa0d447ea87d7594873893230.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvbmdndWFudGluZw==,size_16,color_FFFFFF,t_70)
*替代比 r和频率感知策略在推理过程中替代候选选择中的影响。*
![在这里插入图片描述](https://img-blog.csdnimg.cn/f68bfe3143234586bed380ddefe0b851.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvbmdndWFudGluZw==,size_16,color_FFFFFF,t_70)
*同义词集ns的大小和频率感知策略的影响。*


# 5. Conclusion

本文的主要提出了一种辅助模型免受对抗文本影响的框架ADFAR，针对的对抗文本噪声主要是“词替换”。该框架可以拆分成两部分理解，FAR代表频率感知随机化方法，通过将被词替换的文本的词随机抽取一些替换回高频词，用对抗性的方法来打败对抗，可理解为一种数据增强的方法。而AD为则是一个异常检测器，通过逻辑回归区分对抗性样本，如若低于阈值，则该样本存在对抗性，需被FAR处理后分类。其他则可进行直接分类。