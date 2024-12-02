---
title: 平均场、Hartree Fock 和 Fermi Liquid
date: 2024-11-18 21:43:00
license: BY-SA
tags: computational methods
---

从平均场开始，直到超越平均场
<!-- more -->

## 平均场


## Hartree Fock (HF)
我们从一个二次量子化的哈密顿量出发
$$ H = t_{ij} c_i^\dagger c_j + V_{ijkl} c_i^\dagger c_j^\dagger c_l c_k $$

平均场一下
$$ c_i^\dagger c_j^\dagger c_l c_k \approx c_i^\dagger \langle c_j^\dagger c_l \rangle c_k - c_j^\dagger \langle c_i^\dagger c_l \rangle c_k  - c_i^\dagger \langle c_j^\dagger c_k \rangle c_l + c_j^\dagger \langle c_i^\dagger c_k \rangle c_l
$$

(首先，这里的 "-" 是怎么来的)
(其次，这里丢掉了 $\lang c c\rang$ 和 $\lang c^\dagger c^\dagger \rang$ 的项)

那么平均场下，
$$ H_{\rm MF} = t_{ij} c_i^\dagger c_j + 2 V_{ikjl} \lang c_k^\dagger c_l \rang c_i^\dagger c_j - 2 V_{ilkj} \lang c_l^\dagger c_k \rang c_i^\dagger c_j $$



## Fermi Liquid
### 准粒子