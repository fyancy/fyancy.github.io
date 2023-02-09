# 【ACL 2020】《 Coach: A Coarse-to-Fine Approach for Cross-domain Slot Filling》阅读笔记

<font color=#999AAA >英文标题：Coach: A Coarse-to-Fine Approach for Cross-domain Slot Filling
<font color=#999AAA >中文翻译：Coach: 由粗到精的跨域槽填充
<font color=#999AAA >原文链接:  [https://www.aclweb.org/anthology/2020.acl-main.3.pdf](https://www.aclweb.org/anthology/2020.acl-main.3.pdf).



@[TOC](文章目录)

</font>

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">









#  Abstract
作为面向任务的对话系统中的一项基本任务，槽填充需要在特定领域通过广泛的训练数据来进行。然而这样的数据往往十分稀缺。因此，跨域槽填充方法的出现解决了数据稀缺问题。
在本文中，作者提出了一种粗到细的跨域槽填充方法(Coach)。该模型首先通过检测token是否为槽实体来学习槽实体的一般模式（共有特征）。之后它才会进一步预测槽实体属于某个特定域。此外，作者还提出了一种模板正则化的方法，通过对基于模板的话语表示进行正则化，来提高槽填充的自适应鲁棒性。实验结果表明，作者的模型在槽填充方面显著优于最先进的方法。此外，我们的模型也可以应用于跨域命名的实体识别任务，并具有比其他现有基线更好的自适应性能。


# 一、Introduction
槽填充模型的目的，是在特定领域中识别用户话语与任务相关的槽类别，是面向任务的对话系统中不可缺少的组成部分。有监督方法在槽填充任务中取得了巨大的成就，但是需要大量的有标记样本。然而得到这样的训练样本昂贵且耗时。为解决数据稀缺问题，本文积极研究跨领域插槽填充方法，利用在源领域学习的知识，并将模型适应到槽类型少标记甚至无标记的目标域中。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210515205530879.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvbmdndWFudGluZw==,size_16,color_FFFFFF,t_70)
跨域槽填充是为了处理了unseen的槽类型。如上图所示，在2017年Bapna提出一种zero-shot的方法进行跨域自适应。它首先生成单词级的表示，然后将每个槽类型进行BIO标注，通过对填槽后每个槽与标注间的特征信息进行提取（eg.两个域则综合考虑两个域的BIO标注的连接特征信息），之后进行token的最终预测。

缺点：由于不同域中的槽实体的固有差异，该框架很难捕获目标域中的整个槽实体。且会出现多重预测问题，例如tune这个单词在播放列表与音乐类型两个域有可能均被标注成B。

作者的方法则提出了一种新的跨域槽填充框架：coach（一种从粗到细的方法），该模型共享源域的所有槽类的参数。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210515211730398.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvbmdndWFudGluZw==,size_16,color_FFFFFF,t_70)它首先通过预测每个token是否为实体来粗略地学习槽实体（slot entity）的标注模式。然后，它结合了每个槽实体的特征，形成插槽类型描述矩阵，并根据每个插槽与该矩阵各个槽的相似性来精细地预测目标域特定的槽类型。通过这种方式，这就是由粗到细的过程。该框架能够避免多重预测问题。
此外，我们引入了一种模板正则化方法，将话语中的槽实体标记分离到不同的槽标签中，并生成正确和不正确的模板来正则化话语表示。通过这样做，模型学会将在语义相似的话语表示（即在相同或相似的模板中）聚类到相似的向量空间中，这进一步提高了自适应的鲁棒性。

# 二. Related work
创新点
  1.将句法分析中由粗到细的思想迁移到跨域槽填充任务，并分为两个步骤处理unseen的槽。
  2.在分类任务中，样本为零或很少的低资源问题一直是一项有趣而具有挑战性的任务，本文跨域适应解决了低资源目标领域的数据稀缺问题。
  3.实验会有目标域的槽从未出现过（其他研究源域中均出现过目标域槽），前人的大多数研究跨领域并没有集中在预测目标领域中unseen的标签类型，因为源领域和目标领域在所考虑的任务中都有相同的标签类型。
# 三. Method
## step1
如图2所示，我们的Coach框架中的插槽填充过程包括两个步骤。在第一步中，我们利用BiLSTM-CRF结构，通过让我们的模型预测每个token是否为槽实体，来学习槽实体的一般模式(即对每个token进行BIO三分类）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210515220416867.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvbmdndWFudGluZw==,size_16,color_FFFFFF,t_70)
具体公式如下：
首先设每个utterance为n个tokens
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210515221746508.png)
图2左半部分，tokens分别通过嵌入层和双向LSTM层得到每个token的隐层表示，输入CRF层得到BIO三分类：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021051522101189.png =600x120)
<font color=#999AAA >E（w）表示句子中各个话语的embedding，每个token的embedding过双向lstm生成话语表示（utterance representation）的隐含状态，隐含状态过条件随机场进行bio标注
## step2
在第二步进一步预测了每个槽实体的特定类型。为了生成槽实体的表示，我们利用另一个编码器BiLSTM来编码槽实体token的隐藏状态，并为每个槽实体生成表示。
根据槽实体进行编码并与槽类型描述的表示矩阵进行相似度计算，公式如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210515221313802.png =600x120)
<font color=#999AAA >rk代表第k个槽实体的表示，，[hi，hi1，…，hj]表示第k个槽实体的BiLSTM隐藏状态。Mdesc∈Rns×ds是槽描述的表示矩阵(ns是可能的槽类型的数量，ds是槽描述的维度)，sk是这个第k个槽实体的特定槽类型预测。*

其中槽类型描述的表示由每个token的embedding相加：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210515222116746.png)
## Template Regularization
在许多情况下，在目标域中也可以找到目标域中类似或相同的插槽类型。然而，由于源域和目标域之间的方差，模型要识别目标域中的插槽类型仍然具有挑战性。为了提高自适应能力，我们引入了一种模板正则化方法。如图2所示，我们首先用不同的槽替换话语中的槽实体标记标签，以生成正确和不正确的话语模板。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210516094358200.png)

然后，作者使用BiLSTM+attention来生成话语和模板的表示（utterance and template representations）：
![](https://img-blog.csdnimg.cn/20210516092951337.png)
<font color=#999AAA >其中ht是第t步中的BiLSTM隐藏状态，wa是注意层中的权值向量，R是输入话语或模板的表示。

之后最小化是正确与错误模板的正则化损失函数，因为Lw取负值，意味着在训练阶段，作者最小化Ru和Rr之间的距离，并最大化Ru和Rw之间的距离，即最小化与正确模板的向量距离，疏远与错误模板的距离，优化性能
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210516093639831.png)
<font color=#999AAA >其中Ru是用户话语的表示，Rr和Rw是是非模板的表示，我们设置β为1，MSE表示均方误差。

为了生成一个错误的模板，我们用另一个随机的槽实体替换正确的槽实体，并为每个话语生成两个错误的模板。通过这样做，模型学会了将相同或类似的模板中的表示聚类到一个类似的向量空间中。因此，属于相同槽类型的token的隐藏状态往往相似，这提高了这些槽类型在目标域中的鲁棒性。

# 四.Experiment
## 第一个实验（存在目标域标签不可见）
作者在SNIPS(评估我们的框架，包含跨7个域（意图）的39个插槽类型，以及每个域的2000个训练样本。每次试验选择一个域作为目标域，其他六个域作为源域。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210516094215576.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvbmdndWFudGluZw==,size_16,color_FFFFFF,t_70)
<font color=#999AAA >F1分数基于标准BIO结构。每行中的分数表示目标域的性能，TR表示+模板正则化模块。

分析：coach比目前最先进的RZT方法有更好的性能，在zeroshot情况下平均高出rzt f1分数3%，此外，基于模板表示的正则化，帮助话语表示聚类到相似的向量空间，进一步提高了鲁棒性。在20 shots与50 shots的情况下高出了rzt百分8-9的f1分数。
## 第二个实验（目标域标签均可见）
作者还研究了另一个在目标领域中没有不可见的标签的自适应情况。我们利用(NER)数据集作为源域。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021051609502775.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvbmdndWFudGluZw==,size_16,color_FFFFFF,t_70)
在此任务中，由于NER中的文本相对更开放，即域间差异性较大，这使得很难捕获每种标签类型的模板。但依旧优于基线。
## 槽类别seen与unseen的对比
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210516095344707.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvbmdndWFudGluZw==,size_16,color_FFFFFF,t_70)
分析：可以看出作者的方法在seen与unseen的槽类型上都有显著的提升，因为它显示的学习槽实体的一般模式。但即便是seen的槽实体，任务难度也很大，原因是目标域与源域跨度也很大，基线无法准确识别。由此说明正则化有效提高了填槽的性能，说明了聚类方法的鲁棒性


## 消融实验

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210516094836946.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvbmdndWFudGluZw==,size_16,color_FFFFFF,t_70)
作者对编码token实体的方法进行了消融研究，作者采用两种方案代替BiLSTM，一种是使用变压器编码器(trs)（Vaswani等人，2017），另一种是简单地求和槽实体内标识的隐藏状态。我们可以看到在不同的方法之间没有显著的性能差异，我们观察到使用BiLSTM来编码token实体通常会取得更好的结果。

# 五. 总结
本文 引入了一种新的跨域槽填充框架来处理unseen的槽类型问题。模型在所有插槽类型中共享其参数，并学习预测输入token是否为插槽实体。然后，它将根据插槽类型的描述来检测这些插槽实体内标识的具体插槽类型。此外，还提出了模板正则化方法来进一步提高自适应的鲁棒性。实验表明，作者的模型显著优于现有的跨域插槽填充方法，并且在目标域中没有不可见标签类型的跨域NER任务中也取得了更好的性能。

另附个人博客：https://dongguanting.github.io/