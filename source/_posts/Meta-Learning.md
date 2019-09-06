---
title: Meta Learning
date: 2018-07-29 20:38:31
tags: [Meta Learning, AI, Machine Learning]
---

​	Meta Learning 最早源于上世纪八九十年代 [6], 最近成为研究的热点，这是一个很好的 可以用来解决 Learn to learn 问题的框架。 17 年 NIPS 有一个 Workshop on Meta Learning 。与迁移学习相比， Meta Learning 可以视为一个更泛化的概念。 

​	传统的机器学习方法为解决某一个特定的任务总是需要大量的训练数据，有一个很直 观的原因是因为传统的机器学习方法在训练一个模型时，总是从零开始学习。但是人类 的学习过程并不是这样，显然人的学习是一个连续的过程，当一个人想要解决某一个问题 时，他会使用之前跟这个任务相关的知识。以图像分类任务为例，传统的机器学习方法， 例如普通的 CNN 模型，或者 AlexNet， VGG， ResNet 这些模型都需要大量的训练数据。 为了解决数据依赖的问题，现在的一个研究热点就是“one shot learning” (few shot learning)[4]。许多解决这个问题的方法 [9, 8] 就是基于 Meta Learning。 

<!-- more -->

{% pdf /pdf/meta-learning.pdf %}

