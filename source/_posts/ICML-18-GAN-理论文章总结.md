---
title: ICML'18 GAN 理论文章总结
date: 2018-07-16 23:17:07
tags: [GAN, ICML]
---

这个部分有五篇文章，其中：

1. 两篇文是通过改变GAN的结构以解决GAN训练困难和模式消失（Mode collapse）的问题。
2. 一篇文章从新的数学角度推导GAN的更新过程，该更新过程更一般化，原有的GAN参数更新过程可视为其某种条件下的特例。文中也简要说明了该更新过程是 stable 的。
3. 一篇文章探究了GAN中生成器的 Jacobian 矩阵的奇异值分布和 GAN 性能的关系。这篇文章很有趣，它根据生成器的 Jacobian 矩阵定义了一个**condition number**，然后在训练过程中发现该值与常用的GAN评估方法 **Inception Score** 和 **Frechet Inception Distance** 的评估值十分相关，最后文中提出一种方法通过控制 **condition number** 来改进 GAN 的训练过程。
4. 一篇文章提出一种新的方法计算WGAN中 Wasserstein distance，同时做了许多相关的理论推导。 这篇文章理论知识很多，我看起来很费劲，也很困惑。文章中虽然做了很多的工作，但是相比于WGAN没有太大的创新。

<!-- more --> 

{% pdf /pdf/memo-icml18-gan.pdf %}