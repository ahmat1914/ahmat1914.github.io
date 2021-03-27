---
title: Adaboost 算法流程
category:
  - MachineLearning
tags:
  - Adaboost
mathjax: true
date: 2021-03-27 19:37:25
img:
---
对于二分类任务，Adaboost 几本思路是训练d 个弱分类器 $G_1(x), G_2(x), ..., G_d(x)$，然后把这些弱分类器线性组合成强分类器 $G(x)$。
<!--more-->
$$
G(x)=sign(f(x))
$$

$$
f(x)=\sum_{d=1}^{D}\alpha_dG_d(x)
$$

那如何求出 第d个模型 $G_d(x)$ 和 对应的系数 $\alpha_d$? 流程如下：

1. 求$G_1, \epsilon_1$。使用初始训练数据 $D_1$ 训练出误差率最小的基分类器 $G_1$， 并记录对应的最小误差率 $\epsilon_1$
2. 使用 
