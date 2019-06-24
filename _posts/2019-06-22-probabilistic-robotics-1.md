---
layout: math
title: 机器人学中的基本概率知识
date: 2019-04-24 22:42:15
categories: [读书笔记]
tags: [概率机器人,slam]
---

递归状态估计
<!--more-->

## 概率 Probability

> 概率机器人技术的核心就是由**传感器数据来估计机器人及环境状态**.状态估计解决的是从不能直接观测但可以推断的传感器数据中估计数量的问题.

> 状态估计旨在从数据中找回状态变量.概率状态估计算法在可能的状态空间上计算置信度分布.

### 概率的基本概念
随机变量: 机器人的传感器测量\控制\机器人的状态及其环境这些都作为随机变量.随机变量可以取多个值,且他们是根据具体的**概率定律**来取值.
令X表示一个随机变量,x表示X的某一特定值.如果X所取的所有值的空间是离散的,记X出现的值为x这一事件的概率为:$p(X=x)$.这就是离散概率.某个离散概率所有事件发生概率之和为1,即:

$$
\sum\limits_xp(X=x)=1
$$

离散概率永远为非负值,即$p(X=x)\geq 0$.为了简化表示,常用缩写$p(x)$代替$p(X=x)$.
连续空间: 连续空间中的随机变量可以取连续值.连续随机变量都具有概率密度函数.
在状态估计的前提中,随机变量都拥有概率密度函数(Probability Density Function).如正态分布的概率密度函数由下面高斯函数给出:

$$p(x)=(2\pi\sigma^2)^{-\frac{1}{2}}\exp\{-\frac{1}{2}\frac{(x-\mu)^2}{\sigma^2}\}$$

这是一维的正太分布,$\mu$是均值,$\sigma$是方差,通常这样的分布可以缩写为:$\mathcal{N}(x;\mu,\sigma^2)$.
高维状态下的正太分布概率函数可以由下式给出:

$$p(\boldsymbol x)=\det(2\pi{\boldsymbol \Sigma})^{-\frac{1}{2}}\exp\{-\frac{1}{2}(\boldsymbol x-\boldsymbol \mu)^T{\boldsymbol \Sigma}^{-1}(\boldsymbol x-\boldsymbol \mu)\}$$

其中$\boldsymbol \mu$是均值矢量,${\boldsymbol \Sigma}$是一个半正定对称矩阵,即协方差矩阵.高维正态分布概率分布函数是一维(标量)正态分布的严格泛化.
和离散随机概率分布总和为1,连续概率密度分布的积分总和也总是等于1:

$$\int p(x){\rm d}x=1$$

**注:概率密度函数上限不局限于1**

两个随机变量X和Y的联合分布(joint distribution)由下式给出:

$$ p(x,y)=p(X=x,Y=y)$$

该式表示X和Y值为x和y的概率.如果X和Y相互独立,有:

$$ p(x,y)=p(x)p(y) $$

**随机变量经常携带其他随机变量的信息**,X在Y值为y这个事件发生的条件下值为x的概率为:
$$p(x|y)=p(X=x|Y=y)$$
这个式子称为条件概率(conditional probability).若$p(y)>0$,则得出:

$$p(x|y)=\frac{p(x,y)}{p(y)}$$

当X和Y**相互独立**,那么X就不会携带Y的信息,反之亦然.
由以上条件概率和概率公理得到一个**全概率定理**:

$$p(x)=\sum\limits_yp(x|y)p(y)  \;\;(离散情况)$$

$$p(x)=\int p(x|y)p(y){\rm d}y  \;\;(连续情况)$$

### 贝叶斯准则
贝叶斯准则(Bayes Rules)将条件概率与其"逆"概率$p(y|x)$联系了起来(前提条件:$p(y)>0$):

$$p(x|y)=\frac{p(y|x)p(x)}{p(y)}=\frac{p(y|x)p(x)}{\sum_{x^\prime}p(y|x^\prime)p(x^\prime)}\;\;(离散情况)$$

$$p(x|y)=\frac{p(y|x)p(x)}{p(y)}=\frac{p(y|x)p(x)}{\int p(y|x^\prime)p(x^\prime){\rm d}x^\prime}\;\;(连续情况)$$

如果x是一个希望由y推测出来的数值,则概率$p(x)$称为**先验概率分布()prior probability distribution**,其中y称为数据,在SLAM中即观测量/传感器测量值.$p(x)$总结了在综合y**之前**已有的关于x的概率分布.概率$p(x|y)$称为**在X上的后验概率分布(posterior probability distribution)**.贝叶斯准则允许通过"逆"概率去计算后验概率,这为机器人状态估计提供了便捷方法:如何在知道传感器数据y的条件下推出当前机器人的状态x?.在机器人学中概率$p(y|x)$经常被称为**生成模型(generative model)**,它表示了状态变量X如何引起测量数据Y.
需要指出:**贝叶斯准则的分母$p(y)$不依赖x**,因此${p(y)}^{-1}$经常写成一个归一化变量$\eta$,由此:

$$p(x|y)=\eta p(y|x)p(x)$$

由贝叶斯准则可以推出下列的式子:

$$p(x|y,z)=\frac{p(y|x,z)p(x|z)}{p(y|z)}$$

$$p(x,y|z)=p(x|z)p(y|z)$$

上式比较特殊,含义是其他变量Z为条件的相互独立的随机变量X,Y的条件联合概率分布,这种情况称为**条件独立(condition independence)**.需要特别注意:*条件概率与绝对独立不是互相充分必要的*.

### 期望值与协方差
随机变量X的期望值由下式给定:

$$
E[X]=\sum\limits_xxp(x)\;\;(离散)
$$

$$E[X]=\int xp(x){\rm d}x\;\;(连续)$$

期望是随机变量的**线性**函数:

$$E[aX+b]=aE[X]+b$$

X的协方差(协方差衡量的是偏离均值的二次方期望):

$$Cov[X]=E[X-E[X]]^2=E[X^2]-E[X]^2$$

### 熵
熵(entropy),一个概率分布的熵可由下式给出:

$$H_p(x)=E[-log_2p(x)]$$