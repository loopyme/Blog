---
layout:     post
title: 一种回归决策树的快速遍历划分算法
subtitle: A Fast Traversal Partition Algorithm for CART
date:       2019-12-08
author:     Loopy
header-img: img/home-bg-geek.jpg
catalog: true
tags:
    - ML
    - Algorithm
---

<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    tex2jax: {
      inlineMath: [ ['$','$'], ['\\(','\\)'] ],
      processEscapes: true
    }
  });
  </script>
<script type="text/javascript" async src="//cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

> 代码仓库: [ML-Algorithm](https://github.com/CQU-AI/ML-Algorithm)

## 问题的提出

众所周知,建立CART树有一个关键步骤:**遍历数据空间中的所有划分界限,寻找最优切分特征$\alpha$与阈值$c$**,以最小化分出的两个集合的方差,也就是下面这个式子:

$$\min\limits_{\alpha,c}{[\sum_{x_i[\alpha]<c}{(y_i-\bar{y_1})}^2+\sum_{x_i[\alpha]>c}{(y_i-\bar{y_2})}^2]}$$

其中, $\bar{y_1},\bar{y_2}$分别是$x_i[\alpha]<c,x_i[\alpha]>c$样本点的y均值.

问题在于,经典的CART树要遍历所有划分界限,上面的最小化式就需要计算$n_{sample} \times n_{feature}$次,每次先计算平均值,再计算方差,所以总的时间复杂度为$n_{sample}^2 \times n_{feature}$,**对于一个较大大数据集来说,这样的时间复杂度是不能够接受的.**

于是,我查看了Sklearn的源码,发现它并没有计算上面的式子,而是对$\sum{y_i}$在做操作,查遍了资料也无法理解为什么要这样做(应该在某些老文献里有,但我没找到),于是只有从上式开始一步一步自己推导.

## 算法思路

首先,拎出一个方差式来变形
$$
\begin{aligned}
\sum{(x-\bar{x})^2}
&=\frac{\sum{(nx-\sum{x})}^2}{n^2}\\
&=\frac{\sum{[n^2x^2+(\sum{x})^2-2nx\sum{x}]}}{n^2}\\
&=\sum{x^2}+\frac{(\sum{x})^2}{n}-\frac{\sum(2nx\sum{x})}{n^2}\\
&=\sum{x^2}+\frac{(\sum{x})^2}{n}-\frac{2(\sum{x})^2}{n}\\
&=\sum{x^2}-\frac{(\sum{x})^2}{n}
\end{aligned}
$$

那么,对一个切分特征$\alpha$与阈值$c$的划分(假设依据$x_i[\alpha]<c$划分为了L,R两个集合):

$$
\begin{aligned}
J&=\min\limits_{\alpha,c}{[\sum_{(x,y) \in L}{(y-\bar{y_1})}^2+\sum_{(x,y) \in R}{(y-\bar{y_2})}^2]}\\
&=\min\limits_{\alpha,c}{[\sum{y^2}-\frac{(\sum_{(x,y) \in L}{y})^2}{n_{L}}-\frac{(\sum_{(x,y) \in R}{y})^2}{n_{R}}]}\\
&=\sum{y^2} + \max\limits_{\alpha,c}{[\frac{(\sum_{(x,y) \in L}{y})^2}{n_{L}}+\frac{(\sum_{(x,y) \in R}{y})^2}{n_{R}}]}
\end{aligned}
$$

由于$\sum{y^2}$对任意划分都相同,故我们现在只需要$\max\limits_{\alpha,c}{[\frac{(\sum_{(x,y) \in L}{y})^2}{n_{L}}+\frac{(\sum_{(x,y) \in R}{y})^2}{n_{R}}]}$得到这个式子以后,我们就知道了,每一次迭代,只需要获知两个集合中样本点个数和样本点y的和就足够了,而无需重新计算方差,时间复杂度降至了$n_{sample} \times n_{feature}$

## 算法代码:

> 完整决策树见: [cquai-ml/DecisionTreeRegressor](https://github.com/CQU-AI/ML-Algorithm/blob/master/cquai_ml/DecisionTreeRegressor.py)

```python
def _build_tree(self, X, y, cur_depth, parent, is_left):
    if cur_depth == self.max_depth:
        self._build_leaf(X, y, cur_depth, parent, is_left)

    best_gain = -np.inf
    best_feature = None
    best_threshold = None
    best_left_ind = best_right_ind = None

    sum_all = np.sum(y)
    step = lambda x, y, a: (x + a, y - a)

    for i in range(X.shape[1]):  # for features
        sum_left, sum_right = 0, sum_all
        n_left = 0
        n_right = X.shape[0]
        ind = np.argsort(X[:, i])

        for j in range(ind.shape[0] - 1):  # for all sample
            # step by step
            sum_left, sum_right = step(sum_left, sum_right, y[ind[j]])
            n_left, n_right = step(n_left, n_right, 1)

            cur_gain = (
                sum_left * sum_left / n_left + sum_right * sum_right / n_right
            )
            if cur_gain > best_gain:  # found better choice
                best_gain = cur_gain
                best_feature = i
                best_threshold = X[ind[j], i]
                best_left_ind, best_right_ind = ind[: j + 1], ind[j + 1 :]
    
    # ...
    
```
