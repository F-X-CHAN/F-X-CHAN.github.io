---
title: 平均场、Hartree Fock 和 Fermi Liquid
date: 2024-11-18 21:43:00
license: BY-SA
tags: computational methods
---

从平均场开始，直到超越平均场
<!-- more -->

> **notation**:
$\lang \cdot \rang$: 平均；如果有下标，是对下标的平均；如果没有下标，就是量子平均
$\#$: 某些我懒得写出来的系数
哈密顿量和波函数里会丢掉一些求和号，假装爱因斯坦掉了


## 什么是关联(涨落)？

**\>\>\> 数列 \<\<\<**
两个数列 $\mathbf{S^{(1)}}=\{S^{(1)}_{i}\}$, $\mathbf{S^{(2)}}=\{S^{(2)}_{i}\}$, 一般而言 $\lang \mathbf{S^{(1)}} \mathbf{S^{(2)}} \rang \neq \lang \mathbf{S^{(1)}} \rang \lang \mathbf{S^{(2)}} \rang$
也就是说，$\mathbf{S_1}, \mathbf{S_2}$ 之间有关系，称为**关联(correlation)**


$\tt{ 
    \mathbf{S^{(1)}}=\{S^{(1)}_{1},S^{(1)}_{2},S^{(1)}_{3}\cdots\}, \mathbf{S^{(2)}}=\{S^{(2)}_{1},S^{(2)}_{2},S^{(2)}_{3}\cdots\} 
}$

$\tt{
    C_{S^{(1)},S^{(2)}}=\lang S^{(1)} S^{(2)} \rang - \lang S^{(1)} \rang \lang S^{(2)} \rang =\left\lang\;\big(S^{(1)}-\lang S^{(1)} \rang\big)\big(S^{(2)}-\lang S^{(2)}\rang\big)\;\right\rang\
}$ 
> 我们对下标做了**平均**，出现了**关联**


**\>\>\> 统计物理 \<\<\<**
for example a 1D Ising model, 
如果体系处于某个确定的态上 $\{\overset{\overset{S_1}{}}{↑},\overset{\overset{S_2}{}}{↑},\overset{\overset{S_3}{}}{↓},\overset{\overset{S_4}{}}{↑},\cdots\}$,
那么当然 $C_{S,S}:=\lang S_i \cdot S_j \rang = \lang S_i \rang \cdot \lang S_j \rang$，体系是没有**涨落(fluctuation)** 的

但是当温度有限的，体系的态随时间会变化 (or, think with ensemble)
$S_i = \lang S_i(t) \rang_t$
$C_{S_i,S_j}= \lang S_i(t) \cdot S_j(t) \rang_t \neq \lang S_i(t) \rang \cdot \lang S_j(t) \rang_t $
> 我们对时间做了**平均**，于是出现了**关联**
这是由于热运动引起的关联，称为**热涨落**

**\>\>\> 量子力学 \<\<\<**
In quantum mechanics, the ground state can be superposition of "classical" states.

$|\Psi\rang = \sum \# |\overset{\overset{S_1}{}}{↑},\overset{\overset{S_2}{}}{↑},\overset{\overset{S_3}{}}{↓},\overset{\overset{S_4}{}}{↑},\cdots\rang := \sum_I \# |I\rang$，每个“经典”构型 $|I\rang$ 都对应一组 $S_i, S_j$ 
$C_{S_i,S_j}= \lang S_i S_j \rang - \lang S_i \rang \lang S_j \rang$, average over all configuration $|I\rang$
> 我们对所有可能的“经典”构型 **平均**，于是出现了**关联**
这是个量子平均，称为**量子涨落**


**\>\>\> 量子场论 \<\<\<**
在量子场论中，$\hat{c}(r) → \varphi(r,τ)$


<br>

#### "那为什么我们需要计算涨落？"
- 哈密顿量里有 
(e.g. Ising: $H=-JS_iS_j$，我们可以猜测基态很接近铁磁$\{↑,↑,\cdots\}$，但是并不严格是)
- 和响应有关系

## (经典) 平均场
> 问题：对于 $ H = -J m \sum_i S_i $ ，求磁矩的分布 $\lang S_i \rang$

$Z[\beta] = \sum_{S_j} {\rm e}^{-\beta H_{\{S_j\}}} → \lang S_j \rang_t =\frac{\sum_{S_j} S_i e^{-\beta H}}{Z}$
对于这样的 **“单粒子”/“无相互作用”** 哈密顿量，我们可以很轻松处理；$Z[\beta]$ 可以解析算出来，进而解析求出$\lang S_i \rang$；
$$Z=\prod_i (\sum_{s_i=\pm 1} e^{\beta J m s_i}) = \prod_i 2\cosh(\beta J m)$$
为了得到第 $i$ 个格点的信息，我们在上面 couple 一个额外的磁场 $m_i$；$ H[m_i] = H -J m_i S_i $
这是个很常见的做法，以后我们还会见到
$ Z[\beta; m_i] = 2\cosh\big(\beta J (m+m_i)\big)\prod_{\ne i} 2\cosh(\beta J m)  $
$$\lang s_i \rang = \partial_{\beta J m_i} \ln Z[\beta;m_i] \big|_{m_i→0} = \partial_{\beta J m_i} \ln Z[\beta;m_i] \big|_{m_i→0} = \tanh(\beta J m)$$



对于 Ising model $H=-JS_i S_j$，$i$ 和 $j$ 格点上的磁矩有“关联”了，$Z[\beta]$ 不好解析求了
但是宏观上来看，对于时间/系综平均下来，每个格点 $i$ 在 $t$ 时刻能看到周围$i-1,i+1$的磁矩影响；
$H=-J S_i S_j \approx -J \lang S_i \rang S_j $，我们又**近似**回到单粒子图像了
那么周围磁场刚好得到了一个自洽方程
$m = \frac{1}{2} \lang s_{i-1} + s_{i+1} \rang = \lang s_i \rang$
$\lang s_i \rang = \tanh(\beta J m)$
$→ m = \tanh(\beta J m)$


## Hartree Fock (HF)
我们从一个二次量子化的哈密顿量出发
$$ H = t_{ij} c_i^\dagger c_j + V_{ijkl} c_i^\dagger c_j^\dagger c_l c_k $$

[那么，库伦为什么是这么个东西呢？]
<span  style="font-size:0.7em;"> 有些地方(比如Moire体系相关文献)喜欢写成 $V = \frac{:\rho(r) \rho(r'):}{|r-r'|}$，$:\;:$ 表示正规序，把所有 $c^\dagger$ 提到 $c$ 前面 </span>

$\Psi(r_1,r_2)$ -> $\Psi^*(r_1,r_2)\frac{1}{|r_1-r_2|} \Psi(r_1,r_2)$

平均场一下
$$ c_i^\dagger c_j^\dagger c_l c_k \approx c_i^\dagger \langle c_j^\dagger c_l \rangle c_k - c_j^\dagger \langle c_i^\dagger c_l \rangle c_k  - c_i^\dagger \langle c_j^\dagger c_k \rangle c_l + c_j^\dagger \langle c_i^\dagger c_k \rangle c_l
$$

(首先，这里的 "-" 是怎么来的)
(其次，这里丢掉了 $\lang c c\rang$ 和 $\lang c^\dagger c^\dagger \rang$ 的项)

那么平均场下，
$$ H_{\rm MF} = t_{ij} c_i^\dagger c_j + V_{ikjl} \lang c_k^\dagger c_l \rang c_i^\dagger c_j $$

## BCS

## 1st quantization

## PySCF
PySCF 代码
用什么魔法来收敛
强关联体系的混沌




---
## 附录
<a id="classic_MF"></a>

