---
title: 深入理解Word2Vec
date: 2018-07-19 16:14:33
tags:
  - Word2Vec
categories:
  - 机器学习
---

# 前言

Word2Vec模型是Tomas Mikolov等人在大概2013年左右发表的论文中提出的一种用于获取词向量的工具，对于了解自然语言处理NLP的读者来说，Word2Vec可以说是家喻户晓的工具。

# 数学基础

Word2Vec的两个模型 CBOW和Skip-Gram  实际上大有用途，从不同角度来描述了周围词与当前词的关系。在理解Word2Vec中过程中，我们首先需要了解一下模型中用到的数学知道，包括sigmoid函数、贝叶斯公式、Huffman编码。

## sigmoid函数

sigmoid函数神经网络常用到的激活函数之一，更是逻辑回归的最关键的函数，
其表达式为：

$$\sigma(x) =\frac { 1 }{ 1- { e }^{ -x } },x\in (-\infty, +\infty)\tag{公式1.1}$$

其导数为：
$$\sigma\prime(x) =\sigma(x) [1-\sigma(x)]\tag{公式1.2}$$
还有一个性质，应该是sigmoid函数被广泛应用到机器学习的领域的原因，在公式推导时，这个导数性质可以大大的简化运算过程，使用得推导出的结果明了易懂。

## 逻辑回归

既然上面讲到了sigmoid函数，下面就关联的串讲一下与之密切相关的逻辑回归模型。在日常生活中我们经常会遇到二分类问题，比如收到的邮件是不是垃圾邮件，网站上推荐的商品会不会被用户点击，其次的广告点击是不是欺诈点击等等。
假设$\{X_i,y_i\}$为二分类问题的样本数据，其中$X_i\in\Bbb{R}^n$，$y\in{0,1}$，当$y=1$时称该样本是正样本，反之$y=0$则为负样本。
对于任意的样本$x=(x_1,x_2,\ldots ,x_n)^\top$，可以将二分类问题的hypothesis函数表示如下：
$$h_\theta(x) =\sigma(\theta_0+\theta_1x_1+\theta_2x_2+\ldots+\theta_nx_n)\tag{公式1.3}$$
其中$\theta=(\theta_1,\theta_2,\ldots,\theta_n)^\top$是所要求解的参数，同样的，为了下标一致后将公式1.3简化，我们引入$x_0=1$，并使用公式1.2进行改写，于是公式1.3可改写成：
$$h_\theta(x) =\sigma(\theta^{\top}x)=\frac { 1 }{ 1- { e }^{ -\theta^{\top}x} }\tag{公式1.4}$$



## 贝叶斯

$$\begin{equation}
e=mc^2
\end{equation}\label{eq1}$$

## Huffman编码
