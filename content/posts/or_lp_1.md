+++
title = "线性规划——单纯形法"
date = "2024-10-28"
updated = "2024-10-31"

[taxonomies]
tags=["课程「运筹学」"]
+++

> 我写这篇笔记本来只是为了期中复习, 但是写着写着就忍不住写成了教程的口吻, 以至于你只需要一点基本的线性代数知识就能够完全看懂本文. 如果能对后来者有所帮助, 那就再好不过了.

# 线性规划模型

我们在中学时候就接触过低维的线性规划问题, 例如

$$
\begin{aligned}
\max \quad & x + y \newline
\text{s.t.} \quad & x + 2y \le 3 \newline
& 4x + 3y \le 7
\end{aligned}
$$

其中 $x+y$ 称为**目标函数**, 后面的两个不等式称为**约束**. 当时我们的解决方法是, 将不等式约束看成是平面 $\mathbb{R}^2$ 上的一个区域, 然后移动直线 $z=x+y$, 在保持直线和区域有交点的情况下, 使得截距 $z$ 最大. 通过这种方法, 轻易就能看出上面这个问题的**最优解**为 $(x,y)=(1,1)$, **最优目标值**为 $z=2$. 

但是当变量的个数变得更多, 解空间的维数变得更高的时候, 我们就们没有办法再画图去观察了. 这时候我们需要寻求一种对于任意规模的线性规划问题都*普适*的, *机械化*的方法. 

为了达到这个目标, 首先需要将所有的线性规划用一个统一的形式表示. 即**标准型**: 

## 标准型

以下形式的线性规划问题, 称为线性规划的标准型:

$$
\begin{aligned}
\max \quad & \mathbf{c^\top} \mathbf{x} \newline
\text{s.t.} \quad & \mathbf{A}\mathbf{x} = \mathbf{b} \newline
& \mathbf{x} \ge \mathbf{0}
\end{aligned}
$$

这里的 $\mathbf{x}, \mathbf{c}$ 是 $n$ 维向量, 即解空间是 $n$ 维的. $\mathbf{A}$ 是 $m \times n$ 满秩矩阵 ($m<n$), $\mathbf{b}$ 是 $m$ 维向量, $\mathbf{A}\mathbf{x} = \mathbf{b}$ 就是关于 $\mathbf{x}$ 的 $m$ 个独立的线性方程(要求 $m<n$, 是因为若 $m\ge n$, 则方程组可以直接解出来或者无解, 从而没有必要讨论). $\mathbf{x} \ge \mathbf{0}$ 的意思是 $\mathbf{x}$ 的所有分量都不小于 $0$.

乍一看可能不太好理解, 为什么标准型中的约束有等式形式的呢? 我们在中学时接触过的线性规划(例如上面给的例子), 不都是不等式约束吗? 其实通过一些操作, 我们可以强行把这些不等式约束变成等式约束, 并且引入所有变量非负的约束, 从而变成标准型. 

## 将任意线性规划问题化为标准型

以上面的例子为例. 首先, $x$ 和 $y$ 并没有非负的限制, 这不符合标准型的要求. 但是我们知道, 任何一个实数都可以拆成两个非负实数的差, 所以只要将 $x$ 和 $y$ 分别拆成 $x=x_1-x_2, y=x_3 - x_4$ 即可:

$$
\begin{aligned}
\max \quad & x_1 - x_2 + x_3 - x_4 \newline
\text{s.t.} \quad & x_1-x_2 + 2x_3 - 2x_4 \le 3 \newline
& 4x_1 - 4x_2 + 3x_3 - 3x_4 \le 7 \newline
& x_i \ge 0, i=1,2,3,4
\end{aligned}
$$

接下来还差一步, 我们要将除了非负约束外的不等式约束都变成等式约束. 这其实也很简单. 我们观察第一个不等式, 既然 $x_1-x_2 + 2x_3 - 2x_4 \le 3$, 那么 $x_1-x_2 + 2x_3 - 2x_4$ 加上一个正数不就等于 $3$ 吗? 于是我们引入一个变量 $x_5$: 

$$
\begin{aligned}
& x_1-x_2 + 2x_3 - 2x_4 + x_5 = 3 \newline
& x_5 \ge 0
\end{aligned}
$$

对于第二个不等式, 也是一样的操作. 于是原问题就变成了下面的标准型: 

$$
\begin{aligned}
\max \quad & x_1 - x_2 + x_3 - x_4 \newline
\text{s.t.} \quad & x_1-x_2 + 2x_3 - 2x_4 + x_5 = 3 \newline
& 4x_1 - 4x_2 + 3x_3 - 3x_4 + x_6 = 7 \newline
& x_i \ge 0, i=1,2,\dots,6
\end{aligned}
$$

虽然问题的变量个数从 $2$ 个增加到了 $6$ 个, 但是不用怕. 只要化成了标准型, 就可以使用**单纯形法**机械地求出最优解. 

{{ note(header="注", body="有些线性规划问题要求目标函数的最小值, 此时将 $\min z$ 写作 $-\max -z$ 即可.") }}

# 可行域和顶点

在介绍单纯形法之前, 首先要对标准型的可行域和顶点有所理解. **可行域**就是指解空间 $\mathbb{R}^n$ 中, 满足约束的那些点的集合. 这些满足约束的点就称为**可行解**. 

## 可行域是凸集

标准型中的 $m$ 个(互相独立的)等式约束, 首先在解空间 $\mathbb{R}^n$ 中确定了一个 $n-m$ 维的*线性子空间*. 只有在这个子空间中的点, 才有可能是可行解. 例如, 假如问题的解是 $3$ 维的, 如果我们有 $1$ 个等式约束, 那么它就确定了一个平面($2$ 维子空间), 只有这个平面上的点才有可能是可行解; 如果有 $2$ 个等式约束, 那么它们就确定了一条直线($1$ 维子空间), 只有这条直线上的点才有可能是可行解. 

接下来, 所有非负约束把这个 $n-m$ 维的线性子空间截断, 最终使得可行域是一个 $n-m$ 维的*凸多面体*(也有可能是一个无界的凸集, 但是不影响下面得出的结论). 

## 最优解是顶点

想象 $3$ 维解空间中的等式约束 $x_1+x_2+x_3=1$, 它所确定的平面被非负约束 $x_1 \ge 0, x_2 \ge 0, x_3 \ge 0$ 截断成了顶点为 $(1,0,0), (0,1,0), (0,0,1)$ 的等边三角形 $\Delta$. 而我们的目标函数 $z=ax_1+bx_2+cx_3$ 是一个平面. 当我们移动这个平面, 使得它与三角形 $\Delta$ 有相交区域时, $z$ 就是可能取到的目标函数值. 什么时候目标函数值能最大呢? 如果这个平面与三角形 $\Delta$ 不平行的话, 不难想象, 必然是两者交于三角形 $\Delta$ 的顶点的时候, 在此顶点处目标函数值最大. 于是我们有下面的定理: 

{{ note(header="定理", body="线性规划问题若有有界最优解, 则必有一个最优解是可行域的顶点.") }}

再回到一般的情况, 我们若想在 $n-m$ 维的凸可行域中找到使得目标函数值最大的点, 必然只能在可行域的顶点中去寻找. 所以这些顶点解被称为**基本可行解**. 那么要怎么获得这些顶点呢? 我们再回到代数的视角, 现在有 $m$ 个等式, 要确定一个 $n$ 维的点, 所以我们还需要 $n-m$ 个等式. 既然是顶点, 那么这 $n-m$ 个等式, 当然就是在 $n$ 个非负变量中, 任意取 $n-m$ 个为 $0$. 这样的取法共有 $\mathrm{C}_{n}^{n-m}$ 种, 对于这么多可能的取法, 单纯形法提供了一种较为系统且完备的遍历方法.

# 单纯形法

单纯形法的基本思想是, 假设我们已经知道可行域的某个顶点, 然后用这个顶点出发, 移动到下一个顶点, 再移动到下一个顶点, 在此过程中让目标函数逐渐增加到最大值. 

接下来仍然沿用之前的例子来对这一过程进行演示:

$$
\begin{aligned}
\max \quad & x_1 - x_2 + x_3 - x_4 = z\newline
\text{s.t.} \quad & x_1-x_2 + 2x_3 - 2x_4 + x_5 = 3 \newline
& 4x_1 - 4x_2 + 3x_3 - 3x_4 + x_6 = 7 \newline
& x_i \ge 0, i=1,2,\dots,6
\end{aligned}
$$

## 单纯形表

为了简洁, 我们将等式约束和目标函数列在一个增广矩阵 $\begin{pmatrix}\mathbf{A} & \mathbf{b} \newline \mathbf{c^\top} & z \end{pmatrix}$ 里:

$$
\begin{pmatrix}
    1 & -1 & 2 & -2 & 1 & 0 & 3 \newline
    4 & -4 & 3 & -3 & 0 & 1 & 7 \newline
    1 & -1 & 1 & -1 & 0 & 0 & z \newline
\end{pmatrix}
$$

这个矩阵叫**单纯形表**. 其中前面的行是等式约束, 最后一行是目标函数. 注意到 $\mathbf{A}$ 的第 $5$ 和 $6$ 列 ($x_5$ 和 $x_6$ 的系数) 构成了二阶单位阵, 因此如果我们让 $x_1, x_2, x_3, x_4$ 都为 $0$, 立刻就可以得到一个基本可行解 $\mathbf{x} = (0, 0, 0, 0, 3, 7)^\top$. 在基本可行解中, 不为零的变量 (此处的 $x_5$ 和 $x_6$) 称为**基变量**, 为零的 (此处的 $x_1, x_2, x_3, x_4$) 则称为**非基变量**. 

## 最优性判据

那么这个解是不是最优的呢? 显然不是, 但是我们还是需要有依据来判断一下. 注意看最后一行, 目标函数被表示为了*非基变量*的线性组合, 即 $z=x_1 - x_2 + x_3 - x_4$. 现在因为所有非基变量都是 $0$, 所以 $z=0$. 如果我们把 $x_1$ 的值改为一个正数, 其他的非基变量不变, 那么就可以得到另一个解, 这个解的 $x_5$ 和 $x_6$ 会有新的值, 但是因为目标函数的表达式中不含 $x_5$ 和 $x_6$, 所以目标函数只会受 $x_1$ 的影响, 而且*因为 $x_1$ 的系数为正*, 目标函数会增大. 所以这个解并不是最优的. 由此可见: 

{{ note(header="定理", body="一个基本可行解是最优解, 当且仅当目标函数表示为其非基变量的线性组合时, 所有系数非正. ") }}

这些系数因此被称为**检验数**.

## 进基与出基

接下来我们要去寻找另一个顶点, 从而让目标函数更优. 方法是让一个基变量变为非基变量 (称为**出基变量**), 同时让一个非基变量变为基变量 (称为**进基变量**). 我们首先选择进基变量. 由刚刚的讨论我们已经知道, 如果让一个检验数非负的非基变量变为正的值, 则目标函数会增加 (或者至少是不变); 而相反地, 如果让一个检验数为负的非基变量变为正的值, 则目标函数会减少. 因此我们要选择*检验数非负的非基变量*进基. 这里我们选择 $x_1$. 

在选择出基变量之前, 我们首先要知道如何实现进出基. 在方程形式的约束条件中, 我们会通过将一个方程加到另一个方程上面来实现进出基. 对应到单纯形表里面, 就是做*初等行变换*. 例如这里如果我们想要 $x_6$ 出基, 因为 $x_6$ 只出现在第 $2$ 行, 所以我们先将第 $2$ 行除以 $4$, 使得 $x_1$ 的系数为 $1$: 

$$
\begin{pmatrix}
    1 & -1 & \dfrac{3}{4} & -\dfrac{3}{4} & 0 & \dfrac{1}{4} & \dfrac{7}{4} \newline
\end{pmatrix}
$$

然后从第 $1$ 行中减去第 $2$ 行, 使得 $x_1$ 只在第 $2$ 行出现:

$$
\begin{pmatrix}
    0 & 0 & \dfrac{5}{4} & -\dfrac{5}{4} & 1 & -\dfrac{1}{4} & \dfrac{5}{4} \newline
    1 & -1 & \dfrac{3}{4} & -\dfrac{3}{4} & 0 & \dfrac{1}{4} & \dfrac{7}{4} \newline
    1 & -1 & 1 & -1 & 0 & 0 & z \newline
\end{pmatrix}
$$

这样 $x_1$ 就变成了基变量, $x_6$ 就变成了非基变量. 此时的基本可行解为 $\mathbf{x}=\left(\dfrac{7}{4},0,0,0,\dfrac{5}{4},0\right)^\top$.

可不可以不让 $x_6$ 出基, 而让 $x_5$ 出基呢? 如果尝试之后, 你会发现单纯形表变成了这样: 

$$
\begin{pmatrix}
    1 & -1 & 2 & -2 & 1 & 0 & 3 \newline
    0 & 0 & -5 & 5 & -4 & 1 & -5 \newline
    1 & -1 & 1 & -1 & 0 & 0 & z \newline
\end{pmatrix}
$$

如果这时候我们尝试写出相应的解, 会发现 $x_6 = -5$ 变成了负数, 因此这个解不再是基本可行解! 我们来分析一下其中的原因. 假设在变换前, 进基变量在各行的系数为 $\mathbf{a}=\begin{pmatrix}
a_1 \newline a_2 \newline \vdots \newline a_m
\end{pmatrix}$, 等式约束右边的常数为 $\mathbf{b}=\begin{pmatrix}
b_1 \newline b_2 \newline \vdots \newline b_m
\end{pmatrix}$. 如果我们让 $x_1$ 出基, 也就是让进基变量在第 $1$ 行的系数为 $1$, 其它行的系数都为 $0$, 那么经过初等行变换可以得到:

$$
\begin{pmatrix}
1 & \dfrac{b_1}{a_1} \newline 
0 & a_2\left(\dfrac{b_2}{a_2} - \dfrac{b_1}{a_1}\right) \newline 
\vdots & \vdots \newline 
0 & a_m\left(\dfrac{b_m}{a_m} - \dfrac{b_1}{a_1}\right)
\end{pmatrix}
$$

因为右边一列的数必须都都是非负的 (否则得到的解中有变量是负数), 所以 $\dfrac{b_1}{a_1}$ 必须是所有 $\dfrac{b_i}{a_i}$ 中最小者. 在刚刚的例子中, 观察 $x_1$ 的系数 $\begin{pmatrix}1 \newline 4\end{pmatrix}$ 和右边的常数 $\begin{pmatrix}3 \newline 7\end{pmatrix}$, 因为 $\dfrac{7}{4} < \dfrac{3}{1}$, 所以我们选择第 $2$ 行对应的基变量 $x_6$ 出基. 

{{ note(header="注意", body="其实 $\dfrac{b_i}{a_i}$ 中也有可能有负数. 之后我们会讨论, 应该选择 *$\dfrac{b_i}{a_i}$ 中的非负最小者对应的基变量* 出基. ") }}

{{ note(header="注", body="在多个非基变量都可以进基或多个基变量都可以出基时, 没有一个通用的选取进出基变量的规则, 能够使得迭代次数最少. 事实上, 可以证明对于任何一个选取进出基变量的规则, 都可以构造一个线性规划问题, 使得单纯形法需要遍历可行域所有的顶点. (注: 需要来源) 但是*总是选择下标最小的进出基变量*能够保证单纯形法收敛. 这个规则称为 **Bland 规则**. ") }}


回到正题, 刚刚我们让 $x_1$ 进基, $x_6$ 出基, 得到了下面的单纯形表和基本可行解 $\left(\dfrac{7}{4},0,0,0,\dfrac{5}{4},0\right)^\top$. 

$$
\begin{pmatrix}
    0 & 0 & \dfrac{5}{4} & -\dfrac{5}{4} & 1 & -\dfrac{1}{4} & \dfrac{5}{4} \newline
    1 & -1 & \dfrac{3}{4} & -\dfrac{3}{4} & 0 & \dfrac{1}{4} & \dfrac{7}{4} \newline
    1 & -1 & 1 & -1 & 0 & 0 & z \newline
\end{pmatrix}
$$

像刚才一样, 我们需要判断这个基本可行解是不是最优解. 但是观察最后一行, 目标函数 $z$ 并不是全部由非基变量表示, 还不能直接判断. 我们可以同样通过初等行变换, 从第 $3$ 行减去第 $2$ 行, 使得进基变量 $x_1$ 的系数为 $0$: 

$$
\begin{pmatrix}
    0 & 0 & \dfrac{5}{4} & -\dfrac{5}{4} & 1 & -\dfrac{1}{4} & \dfrac{5}{4} \newline
    1 & -1 & \dfrac{3}{4} & -\dfrac{3}{4} & 0 & \dfrac{1}{4} & \dfrac{7}{4} \newline
    0 & 0 & \dfrac{1}{4} & -\dfrac{1}{4} & 0 & -\dfrac{1}{4} & z-\dfrac{7}{4} \newline
\end{pmatrix}
$$

*到这里, 我们就完成了单纯形法的一步完整的迭代*. 接下来因为目标函数中 $x_3$ 的系数为正, 我们选择它进基, 并且因为 $\dfrac{\frac{5}{4}}{\frac{5}{4}} < \dfrac{\frac{7}{4}}{\frac{3}{4}}$, 我们选择第 $1$ 行对应的基变量 $x_5$ 出基:

$$
\begin{pmatrix}
    0 & 0 & 1 & -1 & \dfrac{4}{5} & -\dfrac{1}{5} & 1 \newline
    1 & -1 & 0 & 0 & -\dfrac{3}{5} & \dfrac{2}{5} & 1 \newline
    0 & 0 & 0 & 0 & -\dfrac{1}{5} & -\dfrac{1}{5} & z-2 \newline
\end{pmatrix}
$$

现在所有非基变量的检验数都为非正数, 说明我们找到了最优解 $\mathbf{x}=(1,0,1,0,0,0)^\top$. 由最后一行可得最优目标值为

$$
z=-\dfrac{1}{5}x_5-\dfrac{1}{5}x_6+2=2
$$

再计算最初问题中的 $x$ 和 $y$:

$$
\begin{aligned}
x & = x_1 - x_2 = 1 \newline
y & = x_3 - x_4 = 1 \newline
\end{aligned}
$$

和我们最开始得出的结论一样. 单纯形法诚不欺我.

# 还没有解决的问题...

读者看到这里可能已经疲倦了, 我也是. 但是不幸的是我们还没有处理单纯形法中的一些特殊情况, 上面的例子只是一个非常完美的个例. 

TODO.

## 无法进基

## 退化的基本可行解

## 初始基本可行解的确定

# 总结