# Vision Transformer学习笔记


自《[Attention is all you need](https://arxiv.org/abs/1706.03762)》之后，transformer先是在NLP领域大放异彩，继而近几年又在CV领域引领科研潮流，ViT、Swin Transformer几乎成了CV学者们耳熟能详的名词。本文是对Vision Transformer系列的学习总结，以ViT的框架为基础，梳理相关工作。

<!--more-->

## 1. 引言

每一个研究热门的兴起，都值得人们思考其背后的意义。个人认为Transformer之于NLP（_Natural Language Processing_）以及CV（_Computer Vision_）领域，意义是极其重大的，除了其自身优异的架构性能，还因为其提供了一种可能，即以一种相对简单的方法打破两个领域之间的壁垒，也就是将NLP的先进经验不需要做太多修改便应用在CV领域。我们都知道近些年NLP领域的发展极其迅速，像是BERT等工作，**无监督+大数据**使其模型极其强大，固然需要大量的资源，但也足以覆盖NLP大多数问题了。而CV领域，目前的确有很多工作是在朝着无监督的方向发展，但是目前还谈不上比肩NLP的程度，私以为二者面对的任务不同，导致在模型设计上可能需要有很大的差异，要**因材施教**，所以要么设计出一种针对于CV任务的模型方案，要么就想办法复用NLP的先进经验，想办法将NLP的模型结构应用到CV领域。听起来貌似后者实现更容易一些，但是领域之间的鸿沟说大不大，说小也不小。说到这里，本文的主角Transformer就是后者方案的典型尝试，可以说Vision Transformer的工作直接打破了AlexNet以来CNN对CV领域的统治地位，虽然随着研究的不断迭代，CNN与Transformer谁更好用仍未可知，但并不妨碍我们探究Transformer的原理以及工作演变。

## 2. 以ViT为中心的研究风暴

### 2.1 CV应用Transformer的难点

纵观前几年的论文发表情况，Transformer在NLP领域中十分火热，并且总体来看，其性能上限随着模型参数量以及数据量的增长，仍保持增长状态，还未饱和。而近几年，则是有大量工作尝试在CV中复用Transformer的红利。按时间节点分析，2018年到2020年间，很多工作相继被提出，大多是结合CNN以及self attention，尝试在CV现有的框架中**引入**Transformer。它们或是直接替换CNN为self attention，或是将Transformer的输入从原始图片换成中间过程中的特征图，归根结底都是为了解决在CV与NLP之间的domain gap，概述有二：

1. **输入数据的差异**
	1. NLP任务的输入是单词或语句，一般而言是一维的序列，且序列长度适中
	2. CV任务一般输入是二维的图片，且图片的尺寸从早期的224×224(pixel)，逐渐发展到了近1000×1000(pixel)。
	3. 所以在数据上分析，如果将二维图片展开成一维向量，那么CV任务输入的数据尺寸将会远超过NLP任务。
2. **计算量的差异**
	1. 对于Transformer计算self attention来说，因为需要计算每个word之间的相关性，所以对应平方级别的时间复杂度。
	2. NLP任务，长度适中的一维序列对应的计算量适中
	3. CV任务则往往受限于与输入序列长度成平方级别的复杂度，导致应用Transformer较为困难，需要改变以往的模型范式。

### 2.2 ViT概述

明确了上述问题之后，我们来看一下ViT的解决方案，首先介绍一下ViT，



[paper](https://arxiv.org/pdf/2010.11929.pdf)  [code](https://github.com/google-research/vision_transformer)

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220222190859.png)





## 相关文献

A Survey on Vision Transformer

A Survey of Visual Transformers
