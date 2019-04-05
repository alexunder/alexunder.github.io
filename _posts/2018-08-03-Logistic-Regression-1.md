---
layout: post
title: Logistic Regression 1: 一元线性回归
category: Machine Learning
---

# 前言

要理解神经网络计算，我们应该简化问题，简化系统。所以我们先从只有一个神经元的情况入手，另外为了继续简化，我们将神经元的输出定以为二元输出，即1或0 。这样就将单一的神经元学习问题转化为二值逻辑回归问题（Binary Logistic Regression）

# 二值逻辑回归问题

为了更好的理解逻辑回归，我们先用数学归纳法的方式，从一元线性回归开始，逐渐扩展到n元回归。

## 一元线性回归

一元线性回归简而言之就是在二维平面的坐标体系下，给定m个数据点，即 $(x^{(i)}, y^{(i)}), i \in Z, i \in [1,m]$ ，找到一个最能代表这些数据点的直线，即所有数据到达该直线的距离之和为最小。为了用数学语言精确表达，我们先定义以下数据集：

$$
(x^{(1)}, y^{(1)}), (x^{(2)}, y^{(2)}) ... (x^{(m)}, y^{(m)})
$$

我们的目标就是找一条直线：

$$
y=\theta x + b
$$

即最终通过这个数据集找到最优的 $\theta$ 和 b，为此我们需要定一个代价函数（Cost Function）,为了表示上文的 “所有数据到达该直线的距离之和为最小”， 代价函数这样定义：

$$
J(\theta, b)=\frac{1}{2} \sum\limits_{i=1}^{m}((\theta x^{(i)} + b) - y^{(i)})^2
$$

通常在统计学中，解决一元线性回归问题，即为了让上面的  $J(\theta, b)$ 达到最小， 一般是用最小二乘法，即对 $J(\theta, b)$ 求 $\theta$ 和 b的偏导数，当偏导数等于0的时候，建立方程组求出具体的 $\theta$ 和 b的值，即

$$ \left\{
\begin{aligned}
\frac{\partial J(\theta, b)}{\partial \theta} & = 0 \\
\frac{\partial J(\theta, b)}{\partial b} & = 0 \\
\end{aligned}
\right.
$$

但是在这里为了给深度学习做准备，我们准备用梯度下降法来求最佳的 $\theta$ 和 b，梯度下降法的好处是很方便编写代码求出期望的参数。首先给$\theta$ 和 b设定初始值，再给定一个学习比率(learning rate) $\alpha$, 通过以下递归式，经过一定次数的迭代得到最终的期望的参数：

$$ \left\{
\begin{aligned}
\theta := \theta - \alpha\frac{\partial J(\theta, b)}{\partial \theta}\\
\theta := \theta - \alpha\frac{\partial J(\theta, b)}{\partial b}\\
\end{aligned}
\right.
$$