---
layout: post
title: Logistic Regression 2 --- 简单神经网络的反向传播推导
category: Machine Learning
---


# 为什么要关注反向传播(Back Propagation)计算

其实单节点的神经网络也就是Logistic Regression。虽然之前的文章想从一元线性回归一直慢慢过渡到Logisitc回归，但是有些细节还不是特别明白。所以这篇小文着重说明一下，单神经节点的反向传播计算公式的推导。其实从工程的角度，软件工程师是不需要特别了解这些细节，因为具体做工程的时候都会利用现有的深度学习框架，比如 TensorFlow, Caffe，编程框架往往已经把神经网络的反向传播都已经实现的很傻瓜了，框架使用者不必关注这些细节了。但是如果能够把反向传播背后的原理搞的清楚一点，还是有利于做一些创新的工作。

# 单个神经网络的简单架构

架构如下图：

![arch](/images/notes/machine_learning/LogReg_kiank.png)


这是一个识别猫咪的简单的case study，输入像素值，输出0 or 1，代表是否是猫！只有一层，一个神经网络，也叫Logistic Regression。从图上我们可以先把前向传播的计算公式列出来：

对于一个实例数据 $x^{(i)}$:

$$z^{(i)} = w^T x^{(i)} + b $$
$$\hat{y}^{(i)} = a^{(i)} = sigmoid(z^{(i)})$$ 
$$ \mathcal{L}(a^{(i)}, y^{(i)}) =  - y^{(i)}  \log(a^{(i)}) - (1-y^{(i)} )  \log(1-a^{(i)})$$

简单解释一下，其实要解释的内容也正是逻辑回归的精髓，不知道只言片语可否表达清楚。式子1表示的是神经网络节点的前半段的计算，即多元线性回归，其中向量w和b是学习算法想要找到或者说训练的参数，这和一元线性回归是一样的，即找到一个最优的直线，多元回归则是找到最优的超平面。但是逻辑回归要求输出的值只有1或者0，所以必须加上式子2的补充， 即sigmoid函数，sigmoid函数定义如下：

$$
sigmoid(x) = \frac{1}{1+e^{-x}}
$$

他的函数图像是这样的：

![Sigmoid](https://upload-images.jianshu.io/upload_images/11430943-f1265640a15c0c75.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



从图像可知，sigmoid函数可以把几乎是全实数的定义域，转化成只有(0, 1)之间的实数，满足Logistic Regresion(逻辑回归)的要求

然后基于全部m个样本数据的总代价函数为:
$$ J = \frac{1}{m} \sum_{i=1}^m \mathcal{L}(a^{(i)}, y^{(i)})$$

式子3代表的代价函数的细节和线性回归不太一样，在这里先做为已知结论，以后有时间再做解释。

# 什么是反向传播计算

反向传播这个名比较酷，好像不明觉厉。但是其实说白了就是在梯度下降法里的对代价函数J求w和b的偏导数，可以继续迭代下去找到最优解即：

$$ \left\{
\begin{aligned}
w := w - \alpha\frac{\partial J(w, b)}{\partial w}\\
b := b - \alpha\frac{\partial J(w, b)}{\partial b}\\
\end{aligned}
\right.
$$

反向传播计算其实就是要计算这两个偏导数。

# 开始推导

推导背后的原理其实很简单，就是对多元函数的链式求导数，把高等数学的微积分好好了解一下就有希望推导出来。说得简单，其实还不是很容易，深度神经网络学习里的反向传播公式计算简直是灾难！所以先从单节点的推导比较容易理解。

将上面的代价函数结合起来：

$$ J = \frac{1}{m} \sum_{i=1}^m (- y^{(i)}  \log(a^{(i)}) - (1-y^{(i)} )  \log(1-a^{(i)}))$$

再加上其他前向计算公式:

$$Z = w^T X + b$$
$$\hat{Y} = A = sigmoid(Z)$$ 

##a. 先求$\frac{\partial J}{\partial w}$ 

通过链式求导得：

$$ \left\{
\begin{aligned}
\frac{\partial J}{\partial w} = \frac{\partial J}{\partial Z} \frac{\partial Z}{\partial w}\\
\frac{\partial J}{\partial Z} = \frac{\partial J}{\partial A} \frac{\partial A}{\partial Z}\\
\end{aligned}
\right.
$$

A是$a^{(i)}$组成的向量，所以先直接对J求A的偏导数得： 

$$\frac{\partial J}{\partial A}= - \frac{1}{m} \sum_{i=1}^m(\frac{y^{(i)}}{a^{(i)}} - \frac{1 - y^{(i)}}{1 - a^{(i)}})   $$

而 $\frac{\partial A}{\partial Z}$ 就是对Sigmoid函数的求导，直接用结论就行，因为比较容易：

$$
\frac{\partial A}{\partial Z}=sigmoid'(Z)= (1 - A)A
$$

然后将$\frac{\partial A}{\partial Z}$和$\frac{\partial J}{\partial A}$  相乘得到：

$$\frac{\partial J}{\partial Z} =  - \frac{1}{m} \sum_{i=1}^m(\frac{y^{(i)}}{a^{(i)}} - \frac{1 - y^{(i)}}{1 - a^{(i)}}) (1 - A)A$$
$$ \frac{\partial J}{\partial Z} =  \frac{1}{m} \sum_{i=1}^m (a^{(i)} - y^{(i)}) $$

$$ \frac{\partial J}{\partial w} = \frac{1}{m}X(A-Y)^T $$

再求 $\frac{\partial Z}{\partial w}$

$$ \frac{\partial Z}{\partial w} =  (w^T X + b)' = X$$

最后得到：

$$\frac{\partial J}{\partial w} = \frac{\partial J}{\partial Z} \frac{\partial Z}{\partial w}= \frac{1}{m} \sum_{i=1}^m (a^{(i)} - y^{(i)})X$$

$$ \frac{\partial J}{\partial w} = \frac{1}{m}X(A-Y)^T $$


## b. 再求$\frac{\partial J}{\partial b}$

同样是应用链式求导大法：

$$\frac{\partial J}{\partial b}=\frac{\partial J}{\partial A} \frac{\partial A}{\partial Z} \frac{\partial Z}{\partial b} $$

可以将上面的中间结果代入：

$$\frac{\partial J}{\partial b} = - \frac{1}{m} \sum_{i=1}^m(\frac{y^{(i)}}{a^{(i)}} - \frac{1 - y^{(i)}}{1 - a^{(i)}}) (1 - A)A $$

$$\frac{\partial J}{\partial b} = \frac{1}{m} \sum_{i=1}^m (a^{(i)} - y^{(i)})$$  这里是因为$\frac{\partial Z}{\partial b}=(w^T X + b)'=1$

# 最后结论总结

再把结果综合一下在这里：
$$ \frac{\partial J}{\partial w} = \frac{1}{m}X(A-Y)^T$$

$$ \frac{\partial J}{\partial b} = \frac{1}{m} \sum_{i=1}^m (a^{(i)}-y^{(i)})$$