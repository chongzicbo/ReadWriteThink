[TOC]

# KL散度简介

KL散度的概念来源于概率论和信息论中。KL散度又被称为：相对熵、互熵、鉴别信息、`Kullback`熵、`Kullback-Leible`散度(即KL散度的简写)。在机器学习、深度学习领域中，KL散度被广泛运用于变分自编码器中(`Variational AutoEncoder`,简称VAE)、EM算法、GAN网络中。

# KL散度定义

KL散度的定义是建立在熵(Entropy)的基础上的。此处以离散随机变量为例，先给出熵的定义，再给定KL散度定义。

若一个离散随机变量$X$的可能取值为$X=\{x_1,x_2,\dots,x_n\}$，而对应的概率为$p_i=p(X=x_i)$，则随机变量$X$的熵定义为：
$$
H(X) = - \sum_{i=1}^{n} p(x_i) \log p(x_{i})
$$
规定当$p(x_i)=0$时，$p(x_i)logp(x_i)=0$

若有两个随机变量$P、Q$,其概率分布分别为$p(x)、q(x)$，则$p$相对$q$的相对熵为：
$$
D_{KL}(p||q)=\sum_{i=1} ^np(x)log \frac{p(x)}{q(x)}
$$
之所以称之为相对熵，是因为其可以通过两随机变量的[交叉熵(Cross-Entropy)](https://www.wikiwand.com/zh-hans/交叉熵)以及信息熵推导得到：
针对上述离散变量的概率分布$p(x)、q(x)$而言，其交叉熵定义为：
$$
H(p,q)=\sum_x p(x)\log \frac 1 {q(x)} \\
= -\sum _x p(x)\log q(x)
$$
在信息论中，交叉熵可认为是对预测分布$q(x)$用真实分布$p(x)$来进行编码时所需要的信息量大小。

因此，KL散度或相对熵可通过下式得出：
$$
\begin{aligned}
D_{KL}(p||q) = & H(p,q)-H(p) \\
=& - \sum_{x} p(x) \log q(x) - \sum_{x}-p(x)\log p(x)\\
=& -\sum_{x} p(x) (\log q(x)-\log p(x))\\
=& -\sum_{x}p(x) \log \frac{q(x)}{p(x)}
\end{aligned}
$$

# KL散度的数学性质

KL散度可以用来衡量两个分布之间的差异，其具有如下数学性质：

## 正定性

$$

D_{KL}(p||q) \geq  0
$$

可用Gibbs 不等式直接得出。先给出Gibbs不等式的内容：

若$\sum_{i=1}^{n} p_i = \sum_{i=1}^{n} q_i  = 1$,且$p_i,q_i \in(0,1]$,则有：
$$

 -\sum_{i}^{n} p_i \log p_i \leq -\sum_{i}^{n} p_i \log q_i
$$
当且仅当$p_i=q_i \forall i$等号成立。

## 不对称性

KL散度并不是一个真正的度量或者距离，因为它不具有对称性：
$$

D(p \| q) \neq D(q \| p)
$$

> 各种散度中，Jensen-Shannon divergence(JS散度)是对称的。

对KL散度不对称性的直观解释可见[链接](https://zhuanlan.zhihu.com/p/45131536)。

# KL代码实现

```python
import numpy as np
from scipy import stats

# 生成两个离散分布
x = [np.random.randint(1, 11) for i in range(10)]
print(x)
p_x = x / np.sum(x)
print(p_x)

y = [np.random.randint(1, 11) for i in range(10)]
p_y = y / np.sum(y)
print(p_y)

KL_P = stats.entropy(p_x, p_y)
KL = stats.entropy(x, y)
print(KL)
print(KL_P)

#验证不对称性
print(stats.entropy(p_y,p_x))

```

# 参考

[KL散度(Kullback-Leibler Divergence)介绍及详细公式推导 | HsinJhao's Blogs](https://hsinjhao.github.io/2019/05/22/KL-DivergenceIntroduction/)