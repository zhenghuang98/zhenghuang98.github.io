---
title: Reading list
author: Zheng Huang
date: 2024-12-03
category: Jekyll
layout: post
---

## NMR  
1. [What is coherence in quantum mechanics? - Physics Stack Exchange](https://physics.stackexchange.com/questions/77495/what-is-coherence-in-quantum-mechanics)

---

### 相干态（或纯态）

考虑两个基本态 $|0\rangle$ 和 $|1\rangle$。  
（如果你对量子态不熟悉，可以将它们看作普通的复向量。）  
这里假设 $|0\rangle$ 和 $|1\rangle$ 是正交的（即 $\langle 0|1\rangle = 0$）。

现在，考虑态  
$
|c\rangle = \frac{1}{\sqrt{2}}(|0\rangle + e^{i\phi}|1\rangle)。
$

态 $|c\rangle$ 是一个（归一化的）相干态，因为相位 $\phi$ 是一个常数。

我们可以观察其密度矩阵 $\rho$，定义为  
$
\rho = |c\rangle\langle c|，
$  
（即 $\langle i|\rho|j\rangle = \rho_{ij} = c_i^* c_j$，其中 $c_1 = \frac{1}{\sqrt{2}}, c_2 = \frac{1}{\sqrt{2}} e^{i\phi}$。  
于是我们得到：  
$
\rho = \frac{1}{2} \begin{pmatrix} 1 & e^{i\phi} \\ e^{-i\phi} & 1 \end{pmatrix}。(1)
$

这个密度矩阵描述了一个相干态。你可以验证 $\rho^2 = \rho$，也就是说 $\rho$ 是投影到相干态 $|c\rangle$ 的算符。

现在，假设相位 $\phi$ 是随机的（也就是说，$c_1$ 和 $c_2$ 之间的相位差是随机的），那么 $e^{i\phi}$ 的期望值为零，密度矩阵变为：  
$
\rho' = \frac{1}{2} \begin{pmatrix} 1 & 0 \\ 0 & 1 \end{pmatrix}。(2)
$

密度矩阵 $\rho'$ 不再表示一个相干态，它仅仅是一个经典统计概率的描述。矩阵 $\rho$ 的非对角元素已经消失。

在这两种情况下，我们都处理的是单个粒子，找到粒子处于 $|0\rangle$ 态或 $|1\rangle$ 态的概率是相同的，均为 $\frac{1}{2}$。  
$
\langle 0|\rho|0\rangle = \langle 1|\rho|1\rangle = \langle 0|\rho'|0\rangle = \langle 1|\rho'|1\rangle = \frac{1}{2}。(3)
$

以下是图片内容的翻译：



### 纠缠

纠缠涉及至少两个粒子，例如纯态  
$
\frac{1}{\sqrt{2}}(|0\rangle|0\rangle + |1\rangle|1\rangle)
$
是一个最大纠缠的双粒子态。

从一个给定的纠缠态中，你可以计算两个粒子联合测量的相关性。

这些纠缠量子相关性显然比经典统计相关性更强。


---
