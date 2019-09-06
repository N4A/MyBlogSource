---
title: Word2Vec
date: 2017-10-21 15:42:58
tags: [World2vec, NPLM, 词向量]
---

### Introduction

最近看了一些Word2Vec的一些相关文章，主要分为两类。一是有关于Word2Vec的发展，主要是以下4篇文章起到奠基性的作用

我看了一些Word2Vec的一些相关文章，主要分为两类。一是有关于Word2Vec的发展，主要是以下4篇文章起到奠基性的作用

1. A Neural Probabilistic Language Model.2003 (NNLM)

2. Recurrent neural network based language model.2010 (RNNLM)

3. Distributed Representations of Words and Phrases and their Compositionality.2013 (Skip Gram Model and CBOW)

4. Efficient Estimation of Word Representations in Vector Space.2013 (Skip Gram Model and CBOW)


二是关于Word2Vec应用的一些文章，一些有趣的idea像是如下的文章

1. Linguistic Regularities in Continuous Space Word Representations.2013
2. Exploiting Similarities among Languages for Machine Translation.2013
3. Distributed Representations of Sentences and Documents.2014

<!-- more --> 

### Development

#### NPLM: A Neural Probabilistic Language Model.2003 

这篇文章主要解决在此之前的自然语言模型是统计语言模型和基于统计语言模型n-gram模型的维度灾难问题。

![statistical language model ](/img/SLM.PNG)

统计语言模型的基本想法就是对于一句话，在给定前几个词的情况下，统计出现下一个词的概率。这样一句话的出现概率就是第一个词出现的概率$P(W_1)$乘上在第一个词给定的情况下出现第二个词的概率$P(W_2|W_1)$, 依此类推，一句话的概率就是上图第一行的联合条件概率乘积。

N-gram模型就是假设一个词出现的概率只考察前后该词前后n个词，以此来降低复杂度。

这些模型的问题就是复杂度非常高，例如：

![curse of dimension](/img/CoD.PNG)

上图的**free parameters**就是指前面语言模型的各个概率P. 统计语言模型就是要统计所有文本将所有概率P确定下来。

文章的作者要解决这一问题，采用distributed vectors来表示每一个词，一句话的上下文语境用训练好的vector的值和网络参数来表达。如下：

![feature vector](/img/feature vector.PNG)

作者给出的网络结构如下：

![NNLM](/img/NNLM.PNG)

如图，首先有一个全局的矩阵C,通过C将一个词w转换为向量C(w)。然后用这些向量作为一个三层神经网络的输入，最终训练出参数和所有词向量的矩阵C。

作者首次使用神经网络来表示自然语言模型，大大降低了计算复杂度（复杂度在后面统一比较）。作者实际上已经使用了词向量但是没有发现这些向量蕴含的语义和语法内容，只是用它来作为自然语言模型的工具。

#### RNNLM:Recurrent neural network based language model.2010 

NNLM有一个问题就是对于一个词，训练的时候它的上下文只取该词前面的n个词，如果n过大，训练复杂度增大；n过小，覆盖的上下文不够。本文作者就利用RNN的特性来解决这一问题。

![RNN LM](/img/RNNLM.PNG)

如上图，RNNLM中将隐藏层视为context层，每一次迭代中，输入由新的输入和前一次的context层共同组成。这样上下文的宽度n不用很大，也可以利用到距离当前词足够远的词带来的影响。

但是本文作者同样是没有注意到词向量蕴含的语义和语法内容

#### Skip Gram Model and CBOW

“Distributed Representations of Words and Phrases and their Compositionality”和“Efficient Estimation of Word Representations in Vector Space”两篇文章提出了Skip Gram Model and CBOW两个模型。

CBOW模型：

![CBOW](/img/CBOW.PNG)

CBOW模型相比于前两个模型，去掉了隐藏层。输出不是线性的softmax，而是基于huffman树的softmax，这样首先使得输出层复杂度由N降到log(N)，其次很好的应用了huffman树的聚类效果。

Skip Gram Model与CBOW类似，只是CBOW是由当前词的上下文来推导出当前词汇，而Skip Gram Model则是反过来。

这两个模型首先是降低了计算复杂度，如下图比较：

![complexity](/img/complexity.PNG)

按照文章的说法

> This makes the training extremely efficient: an optimized single-machine implementation can train on more than 100 billion words in one day 

**此外最重要的是，作者首次察觉到词向量蕴含了丰富的语法和语义的含义。比如**

>  **vec(“Madrid”) - vec(“Spain”) + vec(“France”) is closer to vec(“Paris”) . **

**文章中用了大量篇幅和实现表现词向量的良好特性**

### Application

然后还有些应用词向量的文章。

####  Exploiting Similarities among Languages for Machine Translation

文章的想法非常简单。在一种语言中一个词的词向量为$X_i$，另一种语言中其对应的词的词向量为$Z_i$,作者做了一些实验发现这写词在各自的语言空间中的相对位置关系有一定的相似性，如下图：

![translation](/img/translation2.PNG)

所以作者认为，只要训练一个转移矩阵W，使得$X_i$经过变换$WX_i$转换到与$Z_i$接近的位置，再转换成对应的词，这样就完成了翻译，训练目标如下图：

![translation](/img/translation.PNG)

作者想法很新颖，但是模型结构非常简单，训练的过程中只考虑到词，没有融入整句话的语义概念。翻译的效果比不上之前的翻译模型，只是开阔了一种新的思路。

####  Distributed Representations of Sentences and Documents

这一篇文章进一步扩展词向量，提出一种方法来训练句子和文章的向量。

![pv dm](/img/pvdm.PNG)

如图，作者的想法也非常简单，在原有的CBOW模型上，作者在输入层额外添加了一个paragraph vector。这个向量是由段落决定的，文章的每一段对应一个向量，在训练过程中，同一段落内部使用同一个向量。这一个向量就代表着该段落的context and semantic.

这是一个非常简单但是巧妙的想法，效果也相当不错。作者做了一个实验来来评估模型，实验是分析一段评论是positive or negative，数据集是Stanford Sentiment Treebank Dataset。实验结果如下，错误率要低于之前的模型。

![pvdm experiment](/img/pvdm_exam.PNG)

这一篇训练sentence vector还可以应用到前一篇翻译的工作中，可以弥补其没有融入整句话的语义概念的不足之处。
