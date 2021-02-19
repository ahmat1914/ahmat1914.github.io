---
title: Bias-Variance Tradeoff
category: MachineLearning
tags: Bias, Variance
date: 2021-02-19 20:31:30
img: /images/optimal-balance-of-bias-and-variance.png
---

While training our model Bias and Variance plays a key role in achieving the required accuracy of the model.There need to be a balance or we can say compromise between Bias and Variance, so to avoid overfitting and underfitting of the model. This compromise is called Tradeoff.
<!--more-->

### Bias（偏差）

In the simplest terms, Bias is the difference between the Predicted Value and the Expected Value. 
简而言之，偏差是预测值和实际值之间的误差。

μ = mean(Y’)
Bias = μ - Y

![](/images/bias.png)

Model with high bias pays very little attention to the training data and oversimplifies the model causing leading to underfitting.

高偏差模型无法捕捉数据规律，导致欠拟合（underfitting）。一般情况下过于简单的模型导致高偏差欠拟合。


### Variance（方差）

简而言之，方差衡量预测结果的离散程度。
μ = mean(Y’)
variance(Y’) = (Y’ - μ)^2

![](/images/variance.png)

Model with high variance pays a lot of attention to training data and does not generalize on the data which it hasn’t seen before. As a result, such models perform very well on training data but has high error rates on test data.

方差高的模型过于捕捉数据规律（甚至捕捉了噪声数据规律），导致无法泛化到新数据上。高方差的模型一般情况下训练数据上表现很好、测试数据上表现很差。

### Bias-Variance Tradeoff

![](/images/bias-variance-tradeoff.png)

If our model is too simple and has very few parameters then it may have high bias and low variance. On the other hand if our model has large number of parameters then it’s going to have high variance and low bias.

So we need to find the right/good balance without overfitting and underfitting the data.

### Total Error

![](images/total-error.png)

To build a good model, we need to find a good balance between bias and variance such that it minimizes the total error.

![](/images/optimal-balance-of-bias-and-variance.png)

An optimal balance of bias and variance would never overfit or underfit the model.
