---
title:  “NLP and Word Embeddings”
mathjax: true
layout: post
date:   2019-09-05 08:00:12 +0800
categories: deeplearning.ai
---

自然语言处理(Natural Language Processing)是序列模型第二周的课课程。我们将会把RNN，GRU和LSTM用于自然语言处理。其中一个关键概念就是词嵌入(Word Embeddings)，这是词汇表示的一种方式，可以让算法自动理解一些类似的词。


# 词汇表征

比较基本的词汇表征方法是用one-hot向量来表示一个词，比如man在词典里是第5391个，则向量的第5391分量为1，其他位置是0。这种表征方式有两个缺点：一是向量维度太大，二是每个词都是孤立的。词嵌入的方式，是指特征化的表示每个词，比如用一个300维的特征向量。通过可视化算法t-SNE，可以发现相似的词会聚集在一块。

# 嵌入矩阵

当我们用算法学习词嵌入的时候，实际是学习一个嵌入矩阵。假设词汇表含有10,000个单词，词汇表里有a，aaron，orange，zulu。则嵌入矩阵$$E$$是一个$$300 \times 10,000$$的矩阵。假设orange的单词编号是6257，用符号$$O_{6527}$$来表示这个one-hot向量($$10,000 \times 1$$)。$$E * O_{6527}$$得到一个300维的列向量$$e_{6527}$$，也即是单词orange的嵌入向量。下图的例子是作业中football的嵌入向量：

![image01]({{site.baseurl}}/image/20190905/lookup.png)

神经网络训练嵌入矩阵是早期的成功方法。随机初始化矩阵$$E$$，然后使用梯度下降法来学习这个$$300 \times 10,000$$矩阵中的各个参数，以及隐藏单元和Softmax层的参数$$W,b$$。可以设一个窗口超参数，比如4表示预测给定4个单词后的下一个单词，于是对于神经网络的输入就是一个1200维的特征向量，然后通过softmax层分类器从10,000个单词中选出预测值。

![image02]({{site.baseurl}}/image/20190905/embedding_matrix.png)

# Word2Vec

前篇有篇博客已经单独总结过一次Word2Vec，CBOW模型从原始语句推测目标词，而Skip-Gram模型从目标词推测出原始语句。CBOW对小型数据库比较合适，而Skip-Gram在大型语料中表现更好。直接通过神经网络训练，会遇到一个计算速度问题，每次计算softmax的概率时，需要对词汇表中10,000个词做指数以及求和计算。解决方案是分级softmax分类器(Hierarchical Softmax)和负采样(Negative Sampling)。

1. 分级softmax分类器

    我们不能直接计算确定到底属于10,000类中的哪一类，如果有一个分类器，告诉我们目标词在词汇表的前5000个还是在后5000个词中；然后第二个分类器告诉我们这个词在前2500个词还是后2500个词中，诸如此类，直到最终准确找到目标词。这样一个树形结构的分类器，意味着每个节点都是一个二分类器，如逻辑回归分类器。

2. 负采样

    Word2Vec算法需要给定一对单词，即上下文词-目标词(context-target)。在例子"I want a glass of orange juice to go along with my cereal.“中，orange和juice是一个正样本。生成一个正样本的方式是，先抽取一个上下文词，然后在一定词距内比如说正负10个词距内选取一个目标词，标签为1。生成一个负样本的方式是，采用相同的上下文词，再在字典中随机选一个词，标签为0。负样本需要重复K次，对于小数据集，K从5到20比较好；如果数据集很多，K可以是2到5。

    现在每次迭代我们只需要训练其中的K+1条数据(K个负样本和1个正样本)，把问题转变成为10,000个二分类问题，而不使用一个巨大的10,000维度的softmax。

    ![image03]({{site.baseurl}}/image/20190905/negative_sampling.png)

**之前也看了Word2Vec的文章，主要集中在公式的推导上面，看完一脸懵逼，负采样我基本没有看懂。大牛们总算能解释清楚来龙去脉，这里要解决的是softmax计算量巨大的问题，然后很快就明白其原理了。**

## GloVe词向量

另一个在NLP社区有着一定势头的算法是GloVe算法。GloVe算法对上下文词和目标词的关系明确化，假定$$X_{ij}$$是单词$$i$$在单词$$j$$上下文中出现的次数，目标是优化:

$$
\begin{aligned}
& min \sum_{i=1}^{10,000} \sum_{j=1}^{10,000} f(X_{ij}) (\theta_i^T e_j + b_i + b_j' - logX_{ij})^2 \\
& f(X_{ij}) = 0 \ when \ X_{ij} = 0 
\end{aligned}
$$

## 词嵌入除偏

词嵌入包含性别，种族，性取向等方面的偏见，比如Programmer就更多的和Man联系在一起。除偏主要有三个步骤:

1. 计算偏见趋势例如性别趋势，$$e_{he} - e_{she}$$和$$e_{male} - e_{female}$$，计算平均值得到一个性别无偏向量$$e_{non-bias}$$作为中轴线。

2. 中和(Neutralize)，对性别中立的词向中轴线上投影。

    ![image04]({{site.baseurl}}/image/20190905/neutralize.png)

3. 均衡(Equalize)，对于某些词比如actor和actress，它们到中轴线的距离不等，因此需要调整保证相等的距离。

    ![image05]({{site.baseurl}}/image/20190905/equalize.png)

IBM开源了一个机器学习除偏的项目[AIF360](https://github.com/IBM/AIF360)。