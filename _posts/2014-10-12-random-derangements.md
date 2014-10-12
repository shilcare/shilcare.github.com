---
layout: post
title: 随机生成错排序列
comment: true
categories: algorithms

---

问题：设计一个洗牌算法，条件是每张牌不能在它原来的位置。

这是网易游戏的一道面试题。题目要求的洗牌算法要满足两个条件：

1. 随机，算法应该等概率的生成各种合法排列；
2. 错排，每张牌都不能在它原来的位置。

## 错排 (Derangements)
首先，回顾一下离散数学里面错排的概念。我们用 $$D{}_{n}$$ 表示集合 $$\lbrace 1, \dots, n \rbrace$$ 的错排数。如果令 $$N~(P_i)$$ 表示满足第 $$i$$ 个数在它原来的位置上这个特性的排列数，那么根据容斥原理

$$ D{}_n = n! - \sum_i N\,(P_i) + \sum_{i~<~j} N\,(P_i\,P_j) + \dots + (-1)^nN\,(P_1P_2\dots P_n)$$

而求满足$$P_{i_1}P_{i_2} \dots P_{i_m}$$这些条件的排列数，相当于固定这些数字后，余下其他数字的排列数，因此

$$ \sum \limits_{1\le i_1 \le \dots \le i_m \le n} N\,(P_{i_1}P_{i_2} \dots P_{i_m}) = {n \choose m}(n-m)! $$

把上式代入后，就得到了错排数的计算公式

$$
\begin{equation}
\begin{split}
D{}_n & = n! - {n \choose 1}(n-1)! + \dots + (-1)^n {n \choose n}(n-n)!  \newline
& = n!\displaystyle\biggl(1-\frac{1}{1!}+\frac{1}{2!}-\dots +(-1)^n\frac{1}{n!}\displaystyle\biggr)
\end{split}
\end{equation}
$$

进一步可以求出 $$n$$ 个元素的所有全排列中错排序列所占的比例

$$ \frac{D{}_n}{n!} = 1-\frac{1}{1!}+\frac{1}{2!}-\dots +(-1)^n\frac{1}{n!} $$

由微积分的知识，函数 $$\mathrm{e}{}^x$$ 的泰勒展开式为

$$ \mathrm{e}{}^x = 1+x+\frac{x^2}{2!}+\frac{x^3}{3!}+\cdots $$

当 $$x=-1$$ 时，

$$ \mathrm{e}{}^{-1} = 1-\frac{1}{1!}+\frac{1}{2!}-\frac{1}{3!}+\dots + (-1)^n\frac{1}{n!} + \cdots $$

由此可以看出当 $$n$$ 趋向于无穷时，错排所占的比例收敛于$$\mathrm{e}{}^{-1}\approx 0.368$$。

### 错排数的递归公式
先考虑元素 1 的放法，1 可以放在除了第一个位置以外的其他任何位置，有 $$n - 1$$ 种方法，假设 1 放在了第 k 个位置，那么元素 k 有两种选择：

1. k 放在第 1 个位置。这样问题转化为余下 $$n-2$$ 个数的错排，有$$D{}_{n-2}$$ 种方法；
2. k 不放在第 1 个位置。那么问题转化为 k 不能在第 1 个位置，剩下的 $$n - 2$$ 个元素不能在其原来的位置，也就是 $$n-1$$ 个元素的错排，有$$D{}_{n-1}$$ 种方法。

由此我们得到错排数的二阶递归式

$$ D{}_n = (n-1)(D{}_{n-1} + D{}_{n-2})$$

继续计算 $$D{}_n - nD{}_{n-1}$$ ：

$$
\begin{equation}
\begin{split}
~~D{}_n - nD{}_{n-1} & = -(D{}_{n-1} - (n-1)D{}_{n-2}) \newline
& = (-1)^2(D{}_{n-2} - (n-2)D{}_{n-3}) \newline
& \dots \newline
& = (-1)^{n-2}
\end{split}
\end{equation}
$$

我们就得到了 $$D{}_n$$ 的一阶递推公式

 $$D{}_n = nD{}_{n-1} + (-1)^{n-2}$$
 

## 全排列随机生成

如果不加错排限制，常用的洗牌算法是[Fisher-Yates' shuffle](http://en.wikipedia.org/wiki/Fisher–Yates_shuffle)。

{% highlight python %}

def shuffle(n):
    buff = [None] * n
    for i in range(0, n):
        buff[i] = i
        k = random.randint(0, i)  # [0, i]
        buff[i], buff[k] = buff[k], buff[i]
    return buff

{% endhighlight %}

注意在Fisher-Yates' shuffle算法中，随机数是在**包含两个端点的闭区间[0, i]**中产生的。这是很容易出错的地方，不过有意思的是，如果不小心写成了不包含右端点的左闭右开区间，Fisher-Yates' shuffle 就变成了随机生成n-阶[轮换](http://en.wikipedia.org/wiki/Cycle_%28mathematics%29)的Sattolo's算法。

{% highlight python %}

def sattolo(n):
    buff = [0] * n
    for i in range(1, n):
        buff[i] = i
        k = random.randint(0, i - 1)  # [0, i - 1]
        buff[i], buff[k] = buff[k], buff[i]
    return buff

{% endhighlight %}

一个n-阶轮换一定是一个大小为n的错排序列，但是错排不一定是n-阶的轮换，可能是多个k-阶轮换的积。

## 随机生成错排

### 简单的方法
一种简单的方法是利用 Fisher-Yates' shuffle 结合 rejection，不断产生随机排列，直到产生的排列是一个错排序列为止。

{% highlight python %}

def isDerangement(perm):
    for i, v in enumerate(perm):
        if v == i:
            return False
    return True
    
def randDerangement(n):
    permutation = shuffle(n)
    while not isDerangement(permutation):
        permutation = shuffle(n)
    return permutation

{% endhighlight %}

由上面的分析，一个随机生成的排列 $$A$$ 是错排的概率为

$$ P(A~is~a~derangement) \approx \frac{1}{\mathrm{e}} $$

那么产生一个错排，平均需要多少次shuffle函数调用呢？设随机变量 $$X$$ 为产生一个错排shuffle函数被调用的次数，显然 $$X$$ 服从参数为 $$1/\mathrm{e}$$ 的几何分布，其数学期望为 $$\operatorname{E}[X] = \mathrm{e}$$，这表明平均每 2.7 次shuffle调用就会生成一个错位排列。这在实际的纸牌游戏中是可以接受的。

### 直接生成随机错排的算法
Martínez(2008)等人提出了一种直接生成随机错排的算法，该算法在Sattolo's算法基础上加了一个标记数组，每个元素以一个适当的概率被标记，被标记的元素不会再被交换到其他的位置。在遍历的过程中，用 $$u$$ 记录剩下的未被标记的元素个数，元素被标记的概率为 $$(u-1)D{}_{u-2}/D{}_{u} $$。

