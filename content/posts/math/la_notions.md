---
title: "线性代数-基本符号"
date: 2022-02-02T22:13:59+08:00
draft: false
categories: [数学]
--- 

## 向量空间

向量空间$V$，由向量$v$组成，一同向量加法与标量乘法，满足一下8个属性（向量空间的公里）：
1. **交换律**：$v+w=w+v \textrm{ for all }v,w\in V$
2. **结合律**：$(u+v)+w=u+(v+w) \textrm{ for all }u,v,w\in V$
3. **零向量**：存在特殊向量（用$0$标识），满足$v+0=v\textrm{ for all } v\in V $
4. **加法逆元**：任意向量$v\in V$都存在一个向量$w\in V$，满足$v+w=0$。这个向量一般用$-v$标识
5. **乘法单位**：$1v=v\textrm{ for all }v\in V$
6. **乘法结合律**：$(\alpha\beta)v=\alpha(\beta v)\textrm{ for all }v\in V\textrm{ and all scalars }\alpha,\beta$
7. **分配律**：$\alpha (u+v)=\alpha u+\alpha v\textrm{ for all }u,v\in V\textrm{ and all scalars }\alpha   $
8. **分配律**：$(\alpha +\beta )v=\alpha v+\beta v\textrm{ for all }v\in V\textrm{ and all scalars }\alpha,\beta $

|符号|释义|
|:---:|:---|
|$ \mathbb{R}^{n} $|$n$维实数向量空间：由所有的 {{<math>}} \mathrm{v} = \left( \begin{array}{c} v_{1} \\ v_{2} \\ \vdots \\ v_{n} \end{array} \right) {{</math>}} 组成，其中所有$ v $都是实数|
|$ \mathbb{C}^{n} $|$n$维复数向量空间：由所有的 {{<math>}} \mathrm{v} = \left( \begin{array}{c} v_{1} \\ v_{2} \\ \vdots \\ v_{n} \end{array} \right) {{</math>}}组成，其中所有$ v $都是复数|
|$ \mathbb{F}^{n} $|$n$维向量空间，即可以是$\mathbb{R}^{n}$，也可以是$\mathbb{C}^{n}$|
|{{<math>}} \mathit{M}_{m \times n} {{</math>}} 或者 {{<math>}} \mathit{M}_{m,n}{{</math>}} | 加法、乘法运算标量 |
|$ \mathit{M}_{m,n}^{\mathbb{R}} $|分元都是实数的$ \mathit{M}_{m,n} $|
|$ \mathit{M}_{m,n}^{\mathbb{C}} $|分元都是复数的$ \mathit{M}_{m,n} $|
|$ \mathbb{P}^{n}$|n次多项式空间：由素有的 $p(t) = a_{0} + a_{1}t + a_{2}t^{2} + \dots + a_{n}t^n$组成。<br/>其中，$t$是自变量，任意$a_{k}\textrm{ for all } 0\leqslant k \leqslant n$都可以是$0$|
|$A=\left(a_{j, k}\right)_{j=1, k=1}^{m,\ \ \ n} $|$m$行$n$列的矩阵 {{<math>}}\left(\begin{array}{cccc} a_{1,1} & a_{1,2} & \ldots & a_{1, n} \\ a_{2,1} & a_{2,2} & \ldots & a_{2, n} \\ \vdots & \vdots & & \vdots \\ a_{m, 1} & a_{m, 2} & \ldots & a_{m, n}\end{array}\right){{</math>}}|
|$A_{j,k}$或者$(A)_{j,k}$|矩阵$A$的$j$行$k$列的项，$A$也可小写|
|$A^{T}$|矩阵$A$的转置，即$(A^{T})j,k = (A)_{k,j}$|

## 线性组合，基

> 定义 2.1：如果任意向量$v\in V$，都可以被 一组向量$v_1,v_2,\dots\in V$ 唯一线性组合表示，则称这组向量为向量空间$V$的基（bases）。
> $$
> v=\alpha_1v_1+\alpha_2v_2+\dots+\alpha_nv_n=\sum_{k=1}^{p}\alpha_kv_k 
> $$

这组系数$\alpha_1,\alpha_2,\dots,\alpha_n$被称为在基下的**坐标**。

$ \textbf{Example} $ ：如下一组向量：

{{< math >}}
\left\{
\begin{array}{l}
e_1=(1,0,0,\dots,0)^T \\
e_2=(1,0,0,\dots,0)^T \\
\dots\dots\\
e_n=(1,0,0,\dots,0)^T
\end{array}
\right.
{{< /math >}}

是$ \mathbb{F}^{n} $的基，即任意向量$v\in\mathbb{F}^n$都可以被表示为如下线性组合：
$$
v=x_1e_1+x_2e_2+\dots+x_ne_n = \sum_{k=1}^{n}x_ke_k
$$
且唯一。这组向量被称为$\mathbb{F}^n$的**标准基**。

$\textbf{Example}$：基于如下一组向量：
$$
e_0:=1,\ e_1:=t,\ e_2:=t^2,\ \dots,\ e_n:=t^n \textrm{ for all }e_k \in \mathbb{P}
$$
任意多项式$p,p(t)=\alpha_0+\alpha_1t+\alpha_2t^2+\dots+\alpha_nt^n$，有唯一表示:
$$
p=\alpha_0e_0+\alpha_1e_1+\alpha_2e_2+\dots+\alpha_ne_n
$$
这组向量被称为$\mathbb{P}^n$的标准基。

基的定义是空间内任意向量有且唯一的在基向量下的线性组合。该定义有两层含义：存在性，唯一性。如果只考虑存在性，则有如下定义：

> 定义 2.2：如果任意向量$v\in V$，都可以被 一组向量$v_1,v_2,\dots\in V$ 线性组合表示，则称这组向量为的generating system（spanning system, cpmplete system）
> $$
> v=a_1v_1+a_2v_2+\dots+a_nv_n=\sum_{k=1}^{p}a_kv_k 
> $$

跟基的定义的唯一差别是，不强调线性组合唯一性。易知，基（例如：$v_1,v_2,\dots,v_n$）增加几个向量（例如：$v_{n+1},\dots,v_p$）即可得到一个generating（complete） system。

> 定义 2.3：线性组合$\alpha_1v_1+\alpha_2v_2+\dots+\alpha_pv_p\qquad\alpha_k=0 \forall k$ 称为 trivial（平凡的）

> 定义 2.4：如果只有trivial线性组合（$ \sum_{k=1}^{p}\alpha_kv_k \textrm{ with }\alpha_k=0\forall k$）可以使得$v_1,v_2,\dots,v_p \in V$为0，则称$v_1,v_2,\dots,v_p \in V$线性无关

换句话说，如果方程$x_1v_1+x_2v_2+\dots+x_pv_p=0$有且仅有trivial解（$x_1=x_2=\dots=x_p=0$），则$v_1,v_2,\dots,v_p$线性无关。

如果一组向量不是线性无关，则被称为线性相关：

> 定义 2.5：如果存在non-trivial（非平凡的）线性组合（$\sum_{k=1}^p\left|a_k\right|\ne0$）可以使$0=\sum_{k-1}^p\alpha_kv_k$，则称$v_1,v_2,\dots,v_p$线性相关

> 命题 2.6：当且仅当，一组向量（$v_1,v_2,\dots,v_p\in V$）中$v_k$可以被其他向量的线性组合表示，则称这组向量线性相关
> $$
> v_k=\sum_{j=1 \atop j\ne k}^p\beta_kv_j
> $$



