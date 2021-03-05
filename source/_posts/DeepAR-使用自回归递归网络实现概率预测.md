---
title: 'DeepAR: 使用自回归递归网络实现概率预测'
category:
  - MachineLearning
tags:
  - DeepAR
  - 时间序列
mathjax: true
date: 2021-03-03 15:51:44
img: /images/deepar-plot.png
---

### 问题定义

以商品销量举例，
* 第 $i$ 个商品在 $t$ 时刻的销量 $Z$ 表示为 $Z_{i, t}$；<!--more-->
* 我们预测条件分布 $P(Z_{i,t}|Z_{i, t-1}, Z_{i, t-2}, ..., Z_{i, 1})$
* 进一步预测更多(截止 T)时间上的条件分布 $P(Z_{i,t}, Z_{i,t+1}, Z_{i,t+2}, ..., Z_{i,T}|Z_{i, t-1}, Z_{i, t-2}, ..., Z_{i, 1})$

我们已知，商品历史销量 $$Z_{i,1:t-1}:=Z_{i, 1}, Z_{i, 2}, ..., Z_{i, t-1}$$ 和特征 $$X_{i, 1:T}:=X_{i,1}, X_{i,2}, ..., X_{i, t-1}, X_{i, t}, X_{i, t+1},..., X_{i, T}$$，预测未来销量。

模型的分布为条件概率似然函数的乘积：

![](/images/distribution-product-of-likehood-factors.png)

其中， 模型参数θ; 

也就是说，已知历史 $conditional\ range:=[1, t-1]$，预测未来 $prediction\ range:=[t, T]$

$\mathbf{h}_{i,t}$ 是自回归网络：
![](/images/autoregressive-recurrent-net.png)

其中 $\Theta$ 是参数。

对 $h_{i,0}$ 和 $Z{_{i,0}}$ 的初始化值都设置了0。

... ... 待续
