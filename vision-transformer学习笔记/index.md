# Vision Transformer学习笔记


自《[Attention is all you need](https://arxiv.org/abs/1706.03762)》之后，transformer先是在NLP领域大放异彩，继而近几年又在CV领域引领科研潮流，ViT、Swin Transformer几乎成了CV学者们耳熟能详的名词。本文是对Vision Transformer系列的学习总结，以ViT的框架为基础，梳理相关工作。

<!--more-->

## 1. 引言

每一个研究热门的兴起，都值得人们思考其背后的意义。个人认为Transformer之于NLP（_Natural Language Processing_）以及CV（_Computer Vision_）领域，意义是极其重大的，除了其自身优异的架构性能，更多地，是因为其提供了一种可能，即以一种相对简单的方法打破两个领域之间的壁垒，也就是将NLP的先进经验不需要做太多修改便应用在CV领域。我们都知道近些年NLP领域的发展极其迅速，像是BERT等工作，**无监督+大数据**使其模型极其强大，固然需要大量的资源，但也足以覆盖NLP大多数问题了。而CV领域，目前的确有很多工作是在朝着无监督的方向发展，但是目前还谈不上比肩NLP的程度，私以为二者面对的任务不同，导致在模型设计上可能需要有很大的差异，要**因材施教**，所以要么设计出一种针对于CV任务的模型方案，要么就想办法复用NLP的先进经验，想办法将NLP的模型结构应用到CV领域。听起来貌似后者实现更容易一些，但是领域之间的鸿沟说大不大，说小也不小。说到这里，本文的主角Transformer就是后者方案的典型尝试，可以说Vision Transformer的工作直接打破了AlexNet以来CNN对CV领域的统治地位，虽然随着研究的不断迭代，CNN与Transformer谁更好用仍未可知，但并不妨碍我们探究Transformer的原理以及工作演变。

## 2. 以ViT为中心的研究风暴

### 2.1 CV应用Transformer的难点

关于Transformer的详细结构暂不在这里赘述，而纵观前几年的论文发表情况，Transformer先是在NLP领域中十分火热，并且总体来看，其性能上限随着模型参数量以及数据量的增长，仍保持增长状态，还未饱和。而近几年，则是有大量工作尝试在CV中复用Transformer的红利。按时间节点分析，2018年到2020年间，很多工作相继被提出，大多是结合CNN以及self attention，尝试在CV现有的框架中**引入**Transformer。它们或是直接替换CNN为self attention，或是将Transformer的输入从原始图片换成中间过程中的特征图，归根结底都是为了解决在CV与NLP之间的domain gap，概述有三：

1. **输入数据的差异**
	1. NLP任务的输入是单词或语句，一般而言是一维的序列，且序列长度适中
	2. CV任务一般输入是二维的图片，且图片的尺寸从早期的224×224(pixel)，逐渐发展到了近1000×1000(pixel)。
	3. 所以在数据上分析，如果将二维图片展开成一维向量，那么CV任务输入的数据尺寸将会远超过NLP任务。
2. **计算量的差异**
	1. 对于Transformer计算self attention来说，因为需要计算每个word之间的相关性，所以对应**平方级别**的时间复杂度。
	2. NLP任务，长度适中的一维序列对应的计算量适中
	3. CV任务则往往受限于与输入序列长度成平方级别的复杂度，导致应用Transformer较为困难，需要改变以往的模型范式。
3. **Transformer与CNN的差异**
	1. CNN具有一些*inductive biases*，也叫归纳偏置，分别是locality以及translation equivariance。locality，局部性，是指图像相邻区域，特征相近。translation equivariance，平移等变性，是指CNN提取图片的特征针对平移操作是等变的。
	2. Transformer可以关注global的信息，self attention可以计算输入序列中两两之间的相关性，所以相比于CNN，Transformer可以更好地捕捉长距离的信息。

因此，针对上述问题，在2020年，Google研发团队提出了ViT这篇工作，而自2020年到至今为止，大量的相关工作相继被提出，大多是以ViT为baseline，尝试优化其性能，后文也会梳理这些改动的工作。这里要提一下最近火热的Swin Transformer，其在CV各大任务上都完成了对CNN的反超，诸如检测、分割等下游任务，现在大多会应用Swin作为网络的主干，而Swin的框架也是改自ViT的，因此ViT的影响力可见一斑。

### 2.2 ViT概述

明确了上述问题之后，我们来看一下ViT的解决方案，首先介绍一下ViT。

[paper](https://arxiv.org/pdf/2010.11929.pdf)  [code](https://github.com/google-research/vision_transformer)

#### ViT的Motivation是什么？结构如何设计？

ViT受到了NLP领域中Transformer模型的可扩展性的启发，尝试将NLP中的Transformer直接应用在CV中，而不经过过多的修改，这样就可以实现在CV任务中训练一个包含几百亿乃至几千亿参数的大模型。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220222190859.png)

首先，为了解决计算量问题，ViT尝试对输入网络的图像**划分patch**，也就是将HxW的图像分成MxN个小块，然后将这些patches转换为一维的序列，这样有效地降低了输入Transformer的序列长度（1000x1000相比于14x14的差异）。处理好输入之后，ViT尝试直接应用NLP领域通用的Transformer结构，因为ViT针对分类任务，所以这里只用了Encoder，并且也尝试使用了class token来与每个patches的特征做信息交流，以提供用来分类的特征。
***本质来讲，ViT的主旨就是想证明，NLP中的Transformer结构，在CV任务中，通过大规模的数据集进行训练，也可以达到比肩CNN的效果。***

#### ViT的结构范式与先前的Transformer应用在CV中的工作有何不同？

其实对图像分块的操作并不难想到，之前也有相关工作，但是很重要的一点是具体实现上，这也是为什么同样相似的工作，在google手上就可以达到这样出类拔萃效果的原因。更多的训练资源一定程度上影响了模型的性能，而这往往是大多数研究学者们不具备的。

而另一点不同，其实也是出发点的差异，ViT论文中通篇都在强调Transformer的可扩展性，强调NLP领域目前的成功，根本原因就是想通过ViT做一次尝试，大规模数据+直接使用NLP中的模型，能否取得一定的效果？答案是确定的。


### 2.3 以ViT为中心，梳理相关工作




## 相关文献

A Survey on Vision Transformer

A Survey of Visual Transformers
