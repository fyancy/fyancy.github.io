
原文链接：https://arxiv.org/abs/2108.13048
数据集：https://drive.google.com/drive/folders/1slqI6pUiab470vCxQBZemQZN-a_ssv1Q
## intro
本文提出了ASR-GLUE benchmark，包含6个不同的NLU任务的新集合，用于评估3种不同背景噪声水平和6个不同母语者的ASR误差下模型的性能。并在噪声强度、误差类型和扬声器变量等方面系统地研究了ASR误差对NLU任务的影响。之后采用了两种方法：基于校正的方法和基于数据增强的方法来提高NLU系统的鲁棒性，但是仍远逊于人类识别能力。
## method
ASR-GLUE流程如下：
![image](https://img-blog.csdnimg.cn/img_convert/bdf905c67e5f86f9b2b068dac0409c8d.png)
首先筛选高质量的文本数据集，保留原始训练集，在测试集中随机选择一个样本子集进行每个任务的人类录音，并提供不同水平的环境噪声，并将音频信号发送到ASR系统中，得到最终的ASR的评估结果。
### ASR系统分析

#### 任务

本文抽取的5个典型的NLU任务为：情感分类(SST-2)、语义文本相似性(STS-B)、转述(QQP2)、 QA NLI(QNLI)、识别文本隐含性(RTE)。并与科学NLI任务(SciTail)合并，总共完成6个任务。
![image](https://img-blog.csdnimg.cn/img_convert/f648709fdef2b032c249c389dae512b2.png)
左：BERT在不同噪声水平下在不同任务上的性能。阴影区域代表人的表现。
右：SST-2任务中不同模型架构的准确性结果。这里的“人类”表示人类在各种噪声设置下的表现。“清洁”表示对干净文本数据的测试。“低、中、高”分别代表低、中、高噪声的测试。
结论：发现各个模型对于ASR的错误敏感度很大，且噪声对准确率影响剧烈

#### ASR错误类型

作者将ASR的错误分为以下四种：
![image](https://img-blog.csdnimg.cn/img_convert/d77d7aa8189b293096bdf7554992e0ce.png)
1. 相似（Similar sounds）：当ASR系统时，有时也会出现错误地将一个单词识别为另一个发音相似的单词。
2. 连接（Liaison）：单词之间产生了连读，或者识别后两个词融在了一起。
3. 插入（Insertion）：ASR系统识别产生单词冗余
4. 删除（Deletion）：ASR系统识别后产生单词遗漏
![image](https://img-blog.csdnimg.cn/img_convert/afad144b80342087aecaf6b8ea9fd488.png)
左：SST-2数据集中不同噪声设置下每种错误类型的百分比。
右：BERT在四个子集上的准确性。每个子集只包含具有一个特定错误类型的测试示例。例如，红色块表示BERT在包含相似声音错误（Similar sounds）的测试样本上的准确性。阴影区域表示由特定错误类型导致的性能下降
#### ASR speakers
![image](https://img-blog.csdnimg.cn/img_convert/5c028e1176eb75273c7bc4074d162eec.png)
测试中6个speaker的准确率差异性，WER越高，意味着测试样本中的ASR误差越大，导致错误分类越多。
### ASR鲁棒性
![image](https://img-blog.csdnimg.cn/img_convert/91261fbfaa21e7f082ef0e13f882bf3a.png)
上图提出的两种增加ASR鲁棒性的策略：ASR纠错和数据增强。
1. ASR纠错：在ASR输出后面加一个纠错模型，本文采用GETToR和BART来将ASR系统的输出转换为干净的文本。在训练中，该模型以ASR假设作为输入，以相应的干净转录文本作为输出。通过这种方法，模型学会了纠正假设中的错误，并将其恢复到一个干净的句子中。

2. ASR数据增强模型：
分为两种，音频级与文本级
**1.音频级增强。** 在音频级增强中，我们采用TTS系统将其中的文本形式的训练数据转换为音频文件。然后我们在音频中添加随机的环境噪声，并采用ASR系统将音频文件转换为ASR假设。在训练过程中，我们使用这些增强的数据作为额外的训练数据，以及原始的训练集来训练NLU模型。
**2.文本级增强。** 由于TTS和ASR系统的成本较高，我们进一步尝试通过文本生成模型或一些手动预定义的规则，将ASR错误注入到训练语料库中，生成错误的文本进行训练

### 实验
![image](https://img-blog.csdnimg.cn/img_convert/bfa7a1a0cd40e1b96de9f1a19a1df51f.png)
由上图可知，音频级增强整体效果最好，但在高噪声情况下效果很差。ASR输出纠错的方法整体较差，作者的解释是ASR系统已经集成了一个强大的n-gram语言模型，以保证系统输出的质量。因此，一个额外的语言模型是冗余的，不能进行进一步的改进。

个人分析： 没有接触过ASR系统，但是感觉这篇对ASR的错误分类不像raddle那么细致系统，且它的第一种ASR纠错其实本质就是添加模型对asr输出文本纠错，且它的效果不好，感觉可以提升。同样文本级增强也是，也是添加噪声语料，对模型进行鲁棒性训练，复旦的平台也可以添加的更全面。

----



