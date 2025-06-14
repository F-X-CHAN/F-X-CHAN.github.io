---
title: FCIQMC
date: 2025-02-04 20:45:39
license: BY-SA
tags: computational methods
---
**F**ull **c**onfigurational **i**nteraction **q**uantum **M**onte **C**arlo, 全组态量子蒙特卡洛
解二次量子化多体波函数，很贵，但能量极其精确

<!-- more -->

## <font color='#ff7f0e'>TL;DR</font>
本文包含一些 NECI ~~踩坑指南~~ 使用经验

**SUMMARY OF FCIQMC**
- 精度：sub-mHa, 基本上 ED 量级
- 体系大小：50 electrons, 100 orbitals, with a Hilbert space dimension of $10^{93}$  <font size=1> 作为对比，ED ~ $2\times10^7$</font>
- 复杂度: $\mathcal{O}(e^{N_{el}})$
- 速度: ~100 CPU, ~ days
- 物理量：energy，[1,2-body expectations](#RDMs), (a few) [excitated states](#MNECI)
- 软件包: **N**-**E**lectron **C**onfiguration **I**nteraction (NECI) [^4]
- [常见变体](#modifications)：AS-FCIQMC, i-FCIQMC

**REF:** 
FCQIMC original paper[^1], excitated states[^2], reduced density matrices[^3], initiator[^5]
目前主要是马普所的 Ali Alavi 组在开发


## <font color='#ff7f0e'> 速查 </font>
**HIGH LEVEL IDEAS:** 
- FCIQMC = ED + projector-QMC 
- walker 代表行列式的系数，分为正负两种，在不同行列式间随机游走，平衡时收敛到 FCI 波函数

**ALGORITHM**： 
- Walker 游走的规则：spawning, death/cloning, annihilation

**TAMING FCIQMC**: [goto](#devils_in_the_details)
- how to converge
- how to convince yourself it is converged


---

## <font color='#ff7f0e'> GENERAL IDEAS </font>
精确对角化(**E**xact **D**iagonalization) / 全组态相互作用 (**F**ull **C**onfiguration **I**nteraction) 将波函数表示成 [组态 (Slater 行列式) ](#configuration) 的线性叠加 $|\Psi\rang = \sum_I C_I |D_I\rang$ 
<font size=1> <p style="line-height: 1;"> 组态可以表示成 $|101001\rang$ 这种形式，1表示对应的轨道上占据电子，0未占据 
</p></font> 

对应的系数 <font size=2> $C_I$ 有 $C_\mathrm{norb}^\mathrm{nel}$</font> 个，随电子数**指数爆炸**。稍大的体系上 (~20 electrons, 40 orbitals) 内存会爆炸； 如果可以用 QMC 的方式来采样，就不需要存下完整的波函数。


**FCIQMC 可以视为蒙卡版本的 ED**：
- 波函数形式和 ED 一样；
<font size=1>所以原文章把FCIQMC称为“行列式空间的扩散” </font>
- 而蒙卡用来找出线性叠加系数 $C_I$。这些系数通过**虚时演化**得到：从一个试探波函数 $\Psi_T$ 出发，不断作用哈密顿量可以达到基态
$\Psi \propto \lim_{\tau\to\infty} {\rm e}^{-\tau H} \Psi_T \approx (1-\Delta \tau \hat{H})\cdots(1-\Delta \tau \hat{H}) \Psi_T $
<font size=1>这种方式称为投影蒙卡 (projector Monte Carlo)；DMC, AFQMC 等也属于这一类 </font>

FCIQMC 中，walker 处于行列式空间上，每一步 walker 可以在行列式之间随机游走；行列式上 walker 的数量就正比于(当前步的) $C_I$ 系数。由于 $C_I$ 系数有正负，walker 也分为 ± 两种。
足够长的演化后，$\vert D_I\rang$ 上的 walker 平均数量收敛到基态的 $C_I$ 系数；对每一步对采样到的波函数计算能量，长时平均后也收敛到基态能量。
<font size=1><p style="line-height: 1;"> 如果你不熟悉 walker 是什么，例子见 [这里](#walker-explanation)
每个组态上的 walker 数经过长时平均后，会正比于 $C_I$ 系数。但是单步的 walker 会始终在改变，即使在很长演化后也不是严格的基态波函数。 </p></font>



那么现在的问题就是
\- **每一步怎么计算能量**
\- **随机游走的规则**

### <font color='#ff7f0e'>怎么计算能量</font>
QMC 中计算能量一般有两种方式
a) Pure estimator 
$$ E =  \frac{\lang \Psi | H | \Psi \rang}{ \lang \Psi | \Psi \rang }$$
b) Mixed estimator 
$$ E =  \frac{\lang \Psi_T | H | \Psi \rang}{ \lang \Psi_T | \Psi \rang }$$

\- 一般来说，Mixed estimator 比较容易计算 (试探波函数常选成 HF，或者少数 Slater determinant 的组合)；代价是，mixed estimator 并不是变分的
<font size=1> 哈密顿量只包括了[单、双激发](#sd-excitation)，因此 $H|\Psi_T\rang$ 只对应单、双激发，只需要找到 FCIQMC 波函数中的单双激发即可 </font>
对于 FCIQMC 来说，一般选 mixed estimator，分母正比于 $\Psi_T$ 对应组态上的 walker 数，分子正比于 $\Psi_T$ 对应的[单、双激发](#sd-excitation)组态上的 walker 数。


\- 虽然比较少用到，在 FCIQMC 中还有另外一种估计能量的方式，即用 shift $S$ 作为能量估计。
FCIQMC 中，投影算符其实是 $e^{-\tau (H-S)}$，而非 $e^{-\tau H}$；多出来的这一项 $S$ 用于控制波函数的模方，也就是 walker 的个数。这个 energy shift $S$ 会动态调整，而波函数达到平衡时 $S$ 的统计平均也对应着能量。
<font size=1> 这种做法对应的能量方差会更大，但对于强关联体系而言，选取合适的试探波函数相当困难，mixed estimator 有时震荡到根本不能用，可以尝试用这个。</font>

\- NECI 程序输出中[具体的做法](#output)


### <font color='#ff7f0e'>随机游走的规则</font>
<font color='#ff7f0e'> *Spawning, death/cloning, annihilation* </font>

虚时演化满足 $-\partial_\tau |\Psi\rangle=H|\Psi\rangle$
对能量的平移只改变 $\Psi$ 的模方，还是同一个物理态；那么虚时演化可以写成 $-\partial_\tau |\Psi\rangle=(\mathbf{H}-\tilde{S})|\Psi\rangle$。
平均场经常是一个好的出发点，常常会选取一个 HF 作为参考态，将哈密顿量写成 $\mathbf{K}:=\mathbf{H}-E_\text{HF}$


进一步按照将波函数按组态展开， $|\Psi\rangle=C_I |D_I\rangle$，代入虚时演化得到 $-\partial_\tau C_I = (\mathbf{K}_{IJ}-S) C_J $

整理一下 
$$\begin{equation}
\delta C_I = \sum_{I\ne J} \delta\tau\mathbf{K}_{IJ} C_J+\delta\tau(\mathbf{K}_{II}-S)C_I
\end{equation}$$

- 第一项对应了 $C_J$ 转移到其他行列式上，也就是 walker 的 spawning
- 第二项对应了所有 $C_I$ 数量的减少/增长，也就是 walker 的 death/cloning；
- walker 有正负，同一个行列式上的正负 walker 会相互抵消，称为 annihilation


**SPAWNING:**
$D_J$ 上的每个 walker，按 $\delta\tau\mathbf{K}_{IJ}$ 的概率向 $D_I$ 上复制一个同号 walker；
- 如果 $δτ K_{IJ}<0$， 那么按概率复制异号的 walker 到 $D_I$ 上
- 如果 $δτ K_{IJ}>1$， 就复制多个同号的 walker


**DEATH/CLONING:**
对 $D_I$ 上每个 walker，按概率 $δτ (K_{II}-S)$ 复制 walker 到$D_I$ 上
- 如果概率小于0，就复制异号 walker

**ANNIHILATION:**
如果在 $D_I$ 上的 walker 有正有负，那么一一对应地互相湮灭

<font size=1> 这里丢掉了不少系数以及生成对应概率的优化，具体的游走策略见[^2] </font>

## <span id=modifications> <font color='#ff7f0e'> 常见改进 </font> </span> 
**ADAPTIVE SHIFT** (AS-FCIQMC) → 保证 walker 数不指数爆炸
FCIQMC中，投影算符取为 $e^{-\tau (H-S)}$ ；当 $S$ 恒定时，波函数的模方会变化，粒子数随着时间指数增加/减小，导致计算不稳定。因此，常见的做法是：先固定 $S$，直到 walker 数足够多；然后动态调节 $S$，保持 walker 数相对稳定。

**INITIATOR** (i-FCIQMC)[^5] → 减少收敛所需要的 walker 数
原始的 FCIQMC 需要的 walker 数大致正比于 FCI 空间的大小，所以需要想办法减少 walker 数。
I-FCIQMC 的想法是，只需要一部分行列式的占据情况是重要的，很多行列式采样不准也没关系。一般也是用双激发、CAS、RAS 来取初始 initiator。
**代价**: 需要在不同 walker 数下分别计算，测试收敛情况；对于 mixed estimator，能量可能会收敛到比真实值更低的地方
<font size=1><p> 
- initiator 可以向任意行列式 spawn
- non-initiator 只能向已占据行列式 spawn
- 除非，两个 non-initiator 同时向一个行列式 spawn 了相同符号的 walker 
- 除了初始的 initiator，占据数足够大的行列式也会成为 initiator 
</p></font>

**SEMI-STOCHASTIC** → 加速收敛
对于 FCI 空间中较小的一部分，用 ED 很快而且准确地处理。
常见选取用于 ED 空间的方法有：占据数大的行列式、HF行列式及其单双激发、取一个 CAS 空间。

**进阶: DNECI** → 计算1-, 2-RDM
TODO

<span id='mneci_brief'>**进阶：MNECI**[^3] → 激发态</span>
基本上是施密特正交化。如果限制一个态和基态正交，然后虚时演化，就可以得到第一激发态。同理，和所有能量更低的态都正交就可以得到更高的激发态。


## <span id='devils_in_the_details'><font color='#ff7f0e'> 实操 <p>*<font size=2>devil in the details* </font></p> </font></span>

### 怎么才算收敛？
**方差计算：BLOCK ANALYSIS**
由于 FCIQMC 前后两步的 walker 存在自关联，计算得到的能量也存在自关联，因此直接计算方差可能会低估不确定度。常见的做法是选取收敛后的能量进行 reblocking / block analysis，如 [```pyblock```包](https://github.com/jsspencer/pyblock) 。
Reblocking 方法把数据分成不同数量的块，给出几个不同的统计误差估计值。随着分的 block 越来越多，统计误差 (std error) 会渐渐收敛，但同时统计误差的误差 (std error error) 会越来越大。因此一般选取统计误差已经收敛，而统计误差误差还比较小的地方作为能量不确定度的估计。详见[`pyblock` 的教程](https://pyblock.readthedocs.io/en/latest/tutorial.html)。
如果 FCIQMC 的能量还没有收敛，或是迭代次数太少，可能会出现 std err 随着 block 增大一直增大/减小的情况。这意味 FCIQMC 能量还没有收敛。

注意，有时候 block analysis 结果似乎收敛，但肉眼还能看到能量很明显还在下降；所以还需要辅以肉眼判断。

**波函数收敛**
walker 占据情况：比如 `FCIMCStats` 中和 trial wavefucntion、HF 的交叠
物理性质: 比如 `fciqmc_output` 中的 `S^2`

**随 walker 数的收敛**
如果采用了 initiator 近似 / i-FCIQMC，需要测试能量随着 walker 数量的收敛情况。肉眼看。



### 可能碰到的各种问题
**不收敛：能量震荡很厉害**
主要由于 mixed estimator 的分母比较小。
强关联体系 (比如 Hubbard 模型) 的电子结构往往比较复杂，随着电子数增加，两个多体波函数的交叠指数减小，选择 trial wavefunction 往往十分重要。
\- 强 (静态) 关联体系中，基态往往需要多个行列式表示，用 HF 作为试探波函数不太行。在实际的 `NECI` 计算中，作为参考的 HF 一直在变，能量也会随之有很多跳变。但除去跳变点之后还算能用。
\- 用初始最多占据组态作为试探波函数，对于分子体系表现很好；但是对于 Hubbard 模型来说，交叠很快变成 0，效果远远不如 HF。
\- 用 CAS 作为试探波函数 则需要一些 “化学直觉”。按理来说选得好可以得到很高的信噪比。
<font size=1>可以通过 `FCIMCStats` 文件中的 `26.HF weight`, `36.TrialOverlap` 来判断试探波函数是否足够好。</font>
如果实在选不出一个好的试探波函数，可以尝试用 shift 估计能量。在其他量能收敛的情况下 (比如分子体系)，由于 shift 和 walker 数量有关，walker 的涨落使得用 shift 估计的不确定度会更大。但就我的经验，对于 Hubbard 模型而言，shift 反而是更好的估计方式。


**walker 数量爆炸 / walker 数量不增长 / 虚时 $τ$ 不断减小**
`NECI` 中，walker 数量和每一步游走的虚时间 $τ$ 耦合在一起。这两个问题基本上可以同时解决。Walker 数量受到 shift 的控制，我们希望 walker 在开始时指数增长，直到我们设定的 walker 数后保持平稳。
`totalWalkers`: 控制初始 walker 数
`diagShift`: 控制开始时 walker 数增长速度 
`shiftDamp`: 控制 shift 增长率的变化，更大的 damping 使得 shift 更容易变化，也就是 walker 数更不稳定 / 更容易增长。

`diagShift` 初始可以取成 (HF energy + 1)；如果 $τ$ 一直减小，可以考虑用 0.1。

### 算不动：frozen / CAS
FCIQMC 对于比较大的体系还是会算不动，减小计算量可以通过
比如进行 HF 计算后，会有一些能量很低的占据轨道和能量很高的未占据轨道。这些轨道在 FCI 中不重要，可以冻结，减少总的电子数和轨道数。
<font size=1>CI 系数大的行列式中，能量很低的占据轨道几乎都是占据的。</font>
`NECI` 中提供了 `freeze n m` 的命令，`n`: core orbitals, `m`: virtual orbs。

由于完整的哈密顿量实在太大，我们其实不太用这个命令，而是在产生哈密顿量的过程中，就把这些轨道和对应的电子去掉，然后读入 `NECI` 中。
<font size=1>在 `fcidump` 中去掉对应的单双电子积分</font>



### NECI 输入与输出

<span id=output> **输出: FCIMCStats** </span>
开头是这样的
```
# FCIMCStats VERSION 2 - REAL : NEl=  24 HPHF=F Lz=F Initiator=T
#     1.Step   2.Shift    3.WalkerCng  4.GrowRate     5.TotWalkers  6.Annihil  7.NoDied  8.NoBorn  9.Proj.E       10.Av.Shift 11.Proj.E.ThisCyc  12.NoatHF 13.NoatDoubs  14.AccRat  15.UniqueDets  16.IterTime 17.FracSpawnFromSing  18.WalkersDiffProc  19.TotImagTime  20.ProjE.ThisIter  21.HFInstShift  22.TotInstShift  23.Tot-Proj.E.ThisCyc   24.HFContribtoE  25.NumContribtoE 26.HF weight    27.|Psi|     28.Inst S^2 29.Inst S^2   30.AbsProjE   31.PartsDiffProc 32.|Semistoch|/|Psi|  33.MaxCycSpawn   34.TrialNumerator  35.TrialDenom  36.TrialOverlap  37.InvalidExcits  38. ValidExcits   
```
\- 计算能量: 
一般采用 mixed estimator, 并用专门的 trial wavefunction 来增大分母、减小统计不确定度，也就是 `(34.TrialNumerator / 35.TrialDenom)`；如果想用 HF 作为 trial，那么可以用 `11.Proj.E.ThisCyc` (也就是 `25.NumContribtoE / 24.HFContribtoE`)， 或者用`20.Proj.E.ThisIter`。如果收敛性比较差，可以试试用 `2.Shift`。
<font size=1>这里的 iter 指的是每一步随机游走，cyc 指的是两次调节 shift 之间所有的随机游走</font>

\- 收敛性:
Walker 数量是否稳定：可以通过 `5.TotWalkers,27.|Psi|` 大致看出来。
波函数是否稳定：`12.NoatHF, 13.NoatDoubs` 在 HF 行列式和双激发行列式上的 walker 数，以及 `26.HF weight`可以看是否选择了正确的 HF 参考组态。
<font size=1>虽然 `NECI` 会不断自动选行列式作为 HF 参考</font>
MNECI 中还有  `<psi1|psi2>` 等可以用来判断基态激发态间正交的情况。
<br>

**输入: fciqmc_input**
见[用户手册(简化版)](https://github.com/ghb24/NECI_STABLE/blob/master/docs/pages/01_user_doc/02_intro_tutorial.md) 
<br>

**输出: fciqmc_output**
其中的 `S^2` 可以用来辅助判断收敛性
<br>

**哈密顿量: FCIDUMP**
`NECI` 的 FCIDUMP 格式和 `MOLPRO` 不同；用 `PySCF` 可以比较容易地产生`NECI`能用的 FCIDUMP。
一个典型的 FCIDUMP 的开头大概是
```
&FCI NORB= 24,NELEC=24,MS2=0,
 ORBSYM=1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1
 ISYM=1,
&END
2.0     1   1   1   1
2.0     2   2   2   2
⋮
-1.0    2   1   0   0
-1.0    3   2   0   0
⋮
-0.0    1   0   0   0
-0.0    2   0   0   0
⋮
0       0   0   0   0
```
`NORB`: (空间)轨道数
`NELEC`: 电子数
`MS2`: 自旋 $S_z$
`ORBSYM`: 轨道的 irrep，不同软件的约定可能会差个1。
`ISYM`: 波函数整体对称性
接下来的是双电子积分 / 二体项：`<ik|jl> i j k l`，
或者用 chemist's notation $(ij|kl)=\int \text{d}{x_1} \text{d}{x_2} φ_i^*(x_1) φ_k^*(x_2) \frac{1}{r_{12}} φ_j(x_1)  φ_l(x_2)  $
接下来的是单电子积分 / 单体项: `<i|H0|j> i j 0 0`
接下来的是每个轨道的 HF eigenvalue: `* i 0 0 0`

如果使用 UHF， FCIDUMP 中需要用自旋轨道代替空间轨道，开头是
```
&FCI NORB= 48,NELEC=24,MS2=0,UHF=.TRUE.
 ORBSYM=1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1
 ISYM=1,
&END
```
`NORB`: 自旋轨道数
一般将偶数轨道 (0,2,4,...,2*nmo-2) 作为 ↑ 自旋，奇数作为 ↓ 自旋

注意，如果没有收敛性问题 (比如 Mott insulator) 的话，还是不建议使用 UHF-FCIDUMP，现有程序算 RDM 会有点问题。


## <font color='#ff7f0e'> 实操: 物理量 & 激发态 </font>
### <span id=RDM> 物理量: DNECI [^3] </span>
只计算能量是不够的，如果需要更多“物理”，就需要计算物理量。一般我们需要的可观测量都是单体或者二体算符，对应就需要计算 1-, 2-body **R**educed **D**ensity **M**atrices (1-RDM, 2-RDM). <font size=1>有些地方也缩写成 OBDM, TBDM</font>


### <span id=MNECI> 激发态计算: HPHF 与 MNECI [^2]   </span>
**HPHF** 可以用来算自旋激发。HPHF 控制自旋部分对称或者反称：如果 `HPHF=0`，`|↑↓>` 和 `|↓↑>` 行列式前的系数相反，也就是 **singlet**；而`HPHF=1`，行列式前的系数相同，也就是 **triplet**。
当然电子数更多的时候，只靠自旋对称或者反称并不能保证体系总的 $S^2$；FCIQMC 在输出文件中给出了 `S^2` 的估计值。
另外，HPHF 只适用于 $S_z = 0$ 的情形。
<font size=1>FCIQMC 要求总 $S_z$ 守恒，而 `HPHF` 需要包含行列式本身和翻转自旋的行列式。</font>

**MNECI** 是更一般的算激发态的方法。原理[见前](#mneci_brief)。
- 如果基态有简并，我的经验是 MNECI 能够收敛到正确的能量；但是波函数并不会稳定，walker 占据情况会不断改变。
- 在强关联体系 (比如 Hubbard) 中，MNECI 的收敛性堪忧。
\- 不同态的顺序可能会在计算过程中反复交换，不做处理，直接对输出结果平均并不能得到正确能量。
\- [TODO]




---
## Appx.
<span id=configuration> 全组态：**Slater 行列式的叠加** </span>
我们考虑一个有$n_\mathrm{el}$ 个电子的体系，有 $n_\mathrm{orb}$ 条轨道 ( $a_i^\dagger$，或者说是格点模型中的"site")

按量子化学的叫法，每一种电子在轨道上占据方式称为一个“组态” <font size=1>比如 $|1010\rang := a_1^\dagger a_3^\dagger |\mathrm{vac}\rang$ </font>，对应一个 Slater 行列式。当包括所有组态(全组态)时，波函数空间是完备的。多体波函数可以表示为这些组态的线性组合，<font size=1>$|\Psi\rang = \# |1100\rang + \# |1010\rang + \# |1001\rang + \cdots$ </font>。

这里的轨道可以是任意轨道，只要空间完备就行，原子轨道、HF轨道、DFT轨道都行。

<span id=sd-excitation> **单、双激发** </span>
有了轨道之后，并且把轨道按照能量高低排列后，就可以按照能量填充了；
Hatree-Fock 态对应着电子从最低向上占据，分出了占据轨道和未占据轨道；
而有一个电子到更高的态上，就对应称为单激发 (电子态)。

<span id=walker-explanation> **Walker** </span>
基态在多体基组下展开后 $|\Psi\rangle= C_I |I\rangle$，量子蒙卡用随机的方式求解这些系数 $C_I$；每一步，每一个 walker 都处在某个 $|I\rangle$ 态上，通过统计 walker 在不同基组出现的概率，就可以平均出对应的系数 $C_I$。
比如实空间的 DMC 中，每个 walker 记录了一套全电子的坐标；而FCIQMC 中，walker 就记录了一个组态。

<font size=1>

|step |1 Δτ| 2 Δτ | 3 Δτ| $\cdots$ | average over <br> large enough time
| --- | --- | --- | --- | ---- | ---- 
| walker distribution | 3 positive walkers on $\vert1010\rang$  | 2 positive walkers on $2\vert1010\rang$<br> and 1 negative walker on $\vert0011\rang$  | $\cdots$ |  $\cdots$ | $\cdots$ 
|sampled wavefunction | ⇒ 3$\vert1010\rang$ | ⇒ $2\vert1010\rang$-$\vert0011\rang$ | $\vert1010\rang$-$\vert1001\rang$ | $\cdots$ | $1.1\vert 0011\rang + 2.1\vert 0101\rang - 1.5 \vert 0011\rang + \cdots$

</font>


## Ref. 
[^1]: origin
[^2]: MNECI
[^3]: RDMs 
[^4]: neci-package: 
[^5]: **initiator** Cleland, Deidre, George H. Booth, and Ali Alavi. “Communications: Survival of the Fittest: Accelerating Convergence in Full Configuration-Interaction Quantum Monte Carlo.” The Journal of Chemical Physics 132, no. 4 (January 28, 2010): 041103. https://doi.org/10.1063/1.3302277.


