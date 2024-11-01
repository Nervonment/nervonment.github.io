+++
title = "线性规划——对偶性和对偶单纯形法"
date = "2024-10-31"
updated = "2024-10-31"

[taxonomies]
tags=["课程「运筹学」"]
+++


# 对偶问题

||原问题|对偶问题|
|-|-|-|
|对称形式|$$\begin{aligned}\max \quad & \mathbf{c}^\top \mathbf{x}\newline \text{s.t.} \quad & \mathbf{A}\mathbf{x}\le \mathbf{b}\newline & \mathbf{x} \ge \mathbf{0} \end{aligned}$$ | $$\begin{aligned}\min \quad & \mathbf{b}^\top \mathbf{y}\newline \text{s.t.} \quad & \mathbf{A^\top}\mathbf{y}\ge \mathbf{c}\newline & \mathbf{y} \ge \mathbf{0} \end{aligned}$$ |
|非对称形式|$$\begin{aligned}\max \quad & \mathbf{c}^\top \mathbf{x}\newline \text{s.t.} \quad & \mathbf{A}\mathbf{x} = \mathbf{b}\newline & \mathbf{x} \ge \mathbf{0} \end{aligned}$$|$$\begin{aligned}\min \quad & \mathbf{b}^\top \mathbf{y}\newline \text{s.t.} \quad & \mathbf{A^\top}\mathbf{y}\ge \mathbf{c}\end{aligned}$$|

- 原问题约束条件的一行对应了对偶问题的一个变量.
- 原问题约束条件中一行变成等式, 则对偶问题的相应变量变成自由变量. 

## 弱对偶引理

$$
\mathbf{c}^\top \mathbf{x} \le \mathbf{b}^\top \mathbf{y}
$$

证明: 

(对称形式)
$$
\mathbf{c}^\top\mathbf{x} \le \mathbf{y}^\top\mathbf{A}\mathbf{x} \le \mathbf{y}^\top \mathbf{b}
$$


(非对称形式)
$$
\mathbf{c}^\top\mathbf{x} \le \mathbf{y}^\top\mathbf{A}\mathbf{x} = \mathbf{y}^\top \mathbf{b}
$$

注意: 在规范形式的证明中, 因为 $\mathbf{x}\ge\mathbf{0}$ 和 $\mathbf{y}\ge\mathbf{0}$, 才能在不等式约束两边同时乘 $\mathbf{x}$ 和 $\mathbf{y}$; 在标准形式的证明中, 因为 $\mathbf{x}\ge\mathbf{0}$, 才能在对偶问题的不等式约束中两边同时乘 $\mathbf{x}$.

## 对偶定理

如果原问题有有限最优解, 则对偶问题也有有限最优解, 且两者对应的目标函数值相等; 如果原问题无解, 则对偶问题无界. 

如果原问题的最优基础可行解对应的基为 $\mathbf{B}$, 那么 $\mathbf{B}^{-\top}\mathbf{c}_\mathbf{B}$ 是对偶问题的最优解. 

### 从原问题的最终单纯形表直接得到对偶问题的最优解

设原问题初始单纯形表为 

$$
\begin{pmatrix}
\mathbf{B} & \mathbf{N} & \mathbf{I} & \mathbf{b} \newline
\mathbf{c} _\mathbf{B} ^\top & \mathbf{c} _\mathbf{N} ^\top & \mathbf{0}^\top & z
\end{pmatrix}
$$

最终单纯形表为在原单纯形表上左乘 $\begin{pmatrix}\mathbf{B}^{-1} & \mathbf{0} \newline -\mathbf{c} _\mathbf{B} ^\top\mathbf{B}^{-1} & 1\end{pmatrix}$, 即

$$
\begin{pmatrix}
\mathbf{I} & \mathbf{B}^{-1}\mathbf{N} & \mathbf{B}^{-1} & \mathbf{B}^{-1}\mathbf{b} \newline
\mathbf{0}^\top & \mathbf{c} _\mathbf{N} ^\top-\mathbf{c} _\mathbf{B} ^\top\mathbf{B}^{-1}\mathbf{N} & -\mathbf{c} _\mathbf{B} ^\top\mathbf{B}^{-1} & z-\mathbf{c} _\mathbf{B} ^\top\mathbf{B}^{-1}\mathbf{b}
\end{pmatrix}
$$

由此可以看出对偶问题的最优解就是原问题的初始基变量对应的列构成的向量的相反向量. 

## 互补松弛定理

设 $\mathbf{x}$ 和 $\mathbf{y}$ 分别是原问题和对偶问题的最优解, 则

$$(\mathbf{y}^\top\mathbf{A}-\mathbf{c}^\top)\mathbf{x}=0$$

证明:

(非对称形式)
$$0=\mathbf{y}^\top\mathbf{b}-\mathbf{c}^\top\mathbf{x}=\mathbf{y}^\top\mathbf{A}\mathbf{x}-\mathbf{c}^\top\mathbf{x} =(\mathbf{y}^\top\mathbf{A}-\mathbf{c}^\top)\mathbf{x}$$

(对称形式)
$$0=\mathbf{y}^\top\mathbf{b}-\mathbf{c}^\top\mathbf{x}\ge\mathbf{y}^\top\mathbf{A}\mathbf{x}-\mathbf{c}^\top\mathbf{x} =(\mathbf{y}^\top\mathbf{A}-\mathbf{c}^\top)\mathbf{x}$$
$$0=\mathbf{c}^\top\mathbf{x}-\mathbf{c}^\top\mathbf{x}\le\mathbf{y}^\top\mathbf{A}\mathbf{x} -\mathbf{c}^\top\mathbf{x} =(\mathbf{y}^\top\mathbf{A}-\mathbf{c}^\top)\mathbf{x}$$

推论:

- $x_i \ne 0 \Rightarrow \mathbf{a}_i \mathbf{y}=c_i$
- $\mathbf{a}_i \mathbf{y} \ne c_i \Rightarrow x_i = 0$

## 灵敏度分析

## 已知问题的最优解, 修改问题, 求新的最优解


# 对偶单纯形法

对于标准形式的线性规划问题

$$
\begin{aligned}
\max \quad & \mathbf{c^\top} \mathbf{x} \newline
\text{s.t.} \quad & \mathbf{A}\mathbf{x} = \mathbf{b} \newline
& \mathbf{x} \ge \mathbf{0} \newline
\end{aligned}
$$
$$
(\mathbf{A} \in \mathbb{R}^{m\times n}; \mathbf{x}, \mathbf{c} \in \mathbb{R}^n;\mathbf{b} \in \mathbb{R}^m)
$$

在对偶单纯形法中, 设某步的单纯形表如下

$$
\begin{pmatrix}
1 & 0 & \cdots & 0 & a_{1,m+1} & \cdots & a_{1,n} & b_1 \newline
0 & 1 & \cdots & 0 & a_{2,m+1} & \cdots & a_{2,n} & b_2 \newline
\vdots & \vdots & \ddots & \vdots & \vdots & \ddots & \vdots & \vdots \newline
0 & 0 & \cdots & 1 & a_{m, m+1} & \cdots & a_{m,n} & b_m \newline
0 & 0 & \cdots & 0 & c_{m+1} & \cdots & c_n & z - z_0
\end{pmatrix}
$$

其中 $x_1, \dots, x_m$ 为基变量. $c_{m+1}, \dots, c_n \le 0$.

{{ note(header="注", body="如果已经列出了这样的单纯形表, 那么原问题的解就不会是无界的, 因为 $z = z_0 + c_{m+1}x_{m+1} + \dots + c_n x_n \le z_0$.") }}

对偶单纯形法单步迭代的步骤为

1. 如果所有 $b_i$ 都非负, 则停止迭代. 当前的基本可行解就是最优解. 
2. **确定出基变量**. 令 $b_k$ 为所有负的 $b_i$ 中下标最小者, 则选择第 $k$ 行对应的基变量出基. 此处假设为 $b_1$ 和 $x_1$.
3. **确定进基变量**. 如果 $a_{1,i} \ge 0, \forall m+1 \le i \le n$, 则问题无解. 否则, 令 $k = \arg\min_{ \lbrace i|m+1\le i \le n, a_{1,i} < 0 \rbrace } \dfrac{c_i}{a_{1,i}}$, 即 $\dfrac{c_k}{a_{1,k}}$ 是 $\dfrac{c_i}{a_{1,i}}$ 中非负最小者, 选择 $x_k$ 进基. 如果有多个 $k$, 选择其中最小者. 此处假设为 $x_{m+1}$.
4. **初等行变换**. 将第 $1$ 行除以 $a_{1,m+1}$, 并乘以 $-a_{i, m+1}$ 加到第 $i$ 行上, 乘以 $-c_{m+1}$ 加到最后一行上. 