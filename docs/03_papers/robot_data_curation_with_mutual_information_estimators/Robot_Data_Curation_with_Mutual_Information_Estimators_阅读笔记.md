---
title: Robot Data Curation with Mutual Information Estimators 阅读笔记
type: paper-note
status: active
created: 2026-07-31
updated: 2026-07-31
tags:
  - embodied-ai
  - data-quality
  - imitation-learning
---
- 资源核验日期：2026-07-31
- 上级索引：[论文笔记索引](../README.md)
- 研究地图：[具身智能研究地图](../../../RESEARCH_MAP.md)
- 关联专题：[实机机器人数据质量评估总览](../embodied_data_quality/实机机器人数据质量评估总览.md)
## 1. 基本信息

- 论文：Robot Data Curation with Mutual Information Estimators
- 作者：Hejna, Mirchandani et al.
- 时间：2025 (arXiv:2502.08623v3)
- 论文链接：https://arxiv.org/abs/2502.08623
- 项目主页：（待核验）
- 官方代码：截至 2026-07-31，未在论文页或作者公开渠道确认官方代码仓库
- 模型或数据：（待核验）
- 开源状态：（待核验）
- 核验日期：2026-07-31
- 研究领域：具身智能 / 机器人数据质量评估

- 论文 PDF：[Robot Data Curation with Mutual Information Estimators.pdf](./Robot Data Curation with Mutual Information Estimators.pdf)

# 阅读笔记：Robot Data Curation with Mutual Information Estimators

> 原文：`papers/RobotDataCurationwithMutualInformationEstimators.pdf`  
> 身份：[PAPER] Joey Hejna et al., arXiv:2502.08623v3，2025-04-22。论文类型：METHOD。  
> 当前模式：L1 核心精读；阶段：首轮方法与主实验核验。  
> 官方入口：[PAPER] https://jhejna.github.io/demonstration-info（论文首页标注“Videos and code”）。本地未发现代码，仓库/提交未核验。

## 30 秒判断

- **定位**：[PAPER] DemInf 是无监督、离线的轨迹排序与筛选方法，以状态—动作互信息的估计贡献定义相对质量。
- **价值**：把“局部控制规律是否可学”转为轨迹级筛选，可与数据集级多样性/可学习性诊断互补。
- **不提供**：[PAPER] 不验证任务是否成功，不显式处理长时程动力学，不直接删低分时间步。
- **决定**：继续 L1；下一步完成主张—证据审计，再决定是否进入代码映射。

## 论文卡

| 字段 | 内容 | 证据 |
|---|---|---|
| 质量定义 | $I(S;A)=H(A)-H(A\mid S)$：高边际动作熵避免恒定动作，低条件动作熵表示给定状态时动作更可预测。 | [PAPER] Eq. (4), Sec. IV |
| 评分粒度 | state-action 对的互信息贡献，按轨迹内时间均值聚合。 | [PAPER] Eq. (5), Sec. V-C |
| 输入 | 状态 $s_t$、动作 chunk $a_{t:t+h}$，分别输入 VAE。 | [PAPER] Fig. 1, Sec. V-A |
| 方法 | VAE 表征 + KSG k-NN 互信息估计；随机批次、多组 `k` 平均。 | [PAPER] Sec. V |
| 输出 | $S(\tau)=\frac{1}{T}\sum_{t=1}^{T}\widehat I(s_t,a_t;D_N)$，用于阈值筛选。 | [PAPER] Sec. V-C |
| 验证 | 人工质量标签排序；RoboMimic、Franka、RoboCrowd 上 BC/ACT/Diffusion Policy。 | [PAPER] Sec. VI |
| 边界 | 假设轨迹成功；可预测停顿；贪心筛选；未解决多任务/语言条件。 | [PAPER] Sec. VII-A/B |

## 方法链

```text
状态 s_t ──> State VAE ──> z_s ┐
                                 ├─> KSG k-NN ─> I_hat(s_t,a_t;D_N) ─> 时间均值 ─> S(τ)
动作块 a_t:t+h ─> Action VAE ──> z_a ┘                                  └─> 阈值筛选
```

### 目标含义

- [PAPER] 行为克隆从状态预测动作。低 $H(A\mid S)$ 使相近状态下的动作集中，更容易拟合，也较少依赖策略不可见信息。
- [PAPER] 只最小化条件熵会偏好恒定动作；高 $H(A)$ 迫使动作随状态产生变化，避免平凡解。
- [INFERENCE] 因而 DemInf 衡量的是控制映射的可辨识性，不是任务完成度、视觉覆盖度或安全性。

### 估计与筛选假设

- [PAPER] KSG 依赖 joint latent 空间的 k-NN 距离；独立 VAE 的各向同性 Gaussian latent 使距离适合非参数估计。
- [PAPER] 对相对排序，样本项正比于 $-\psi(n(z_s)+1)-\psi(n(z_a)+1)$；绝对常数项省略。分数裁剪到 1%–99% 分位后再按轨迹平均。
- [PAPER] 关键抓取阶段本来难预测，按时间步直接删除低 MI 会破坏任务，因此作者只筛轨迹。
- [INFERENCE] 轨迹聚合避免断开关键动作，但可能掩盖总体较好轨迹内的碰撞或无效片段，需与 transition-level 方法互补。

## 具体问题设定与完整数据流

论文的单个样本不是“某张图像的质量”，而是一条成功示教内的 state-action 对：`(s_t, a_t:t+h)`。[PAPER, Sec. III-B, V]

```text
成功轨迹 tau_i = (s_1, a_1, ..., s_T, a_T)
  ├─ 每个 s_t 进入 State VAE，得到 z_s,t
  ├─ 连续 h 步动作 a_t:t+h 进入 Action VAE，得到 z_a,t
  ├─ 在随机批 B 内，用 [z_s,t, z_a,t] 的邻域估计该对的 I-hat_t
  └─ clip 到全数据 1%–99% 分位后，平均所有 t：S(tau_i)=mean_t(I-hat_t)

按阈值 kappa 仅保留 S(tau_i)>kappa 的完整轨迹，再训练下游策略。
```

**Franka 的具体实例：**论文中 action chunk 为 4、$z_s\in\mathbb R^{24}$、$z_a\in\mathbb R^{16}$。因此某一时刻会把该时刻的多相机图像/状态压缩到 24 维，并把未来连续 4 个控制动作压缩到 16 维；它们的 joint latent 决定这个时刻的分数。这里的“时刻”只是打分中间粒度，最终删除单位仍是整条轨迹。[PAPER, Appendix Table I]

| 阶段 | 使用的数据 | 学到/计算的对象 | 输出 | 是否使用成功标签/奖励 |
|---|---|---|---|---|
| VAE 训练 | 全部 state 与 action chunk | $f_s$、$f_a$ | $z_s=f_s(s)$、$z_a=f_a(a)$ | 否 |
| MI 打分 | VAE latent 的随机批 | KSG 邻居计数与 $\widehat I(s,a;B)$ | 每个 state-action 分数 | 否 |
| 筛选 | 每条轨迹的分数序列 | 裁剪、均值、阈值 | 完整保留/删除轨迹 | 否 |
| 下游验证 | 全量或筛选后的轨迹 | BC/ACT/Diffusion Policy | 成功率或任务分数 | 仅在实验评估使用 |

[PAPER] 因而 DemInf 本身不使用人工质量标注、奖励、已训练策略损失或环境交互。它评估的是数据的**内部 state-to-action 结构**，不是目标完成度。

## 公式精读：符号、推导、用途与设计理由

本节把论文的式 (1)–(5)、正文中的状态分布上界、KSG 局部估计和最终轨迹分数连成一条完整推导链。论文明确写出的内容标为 `[PAPER]`；为帮助理解而补充的标准概率论/信息论恒等式标为 `[INFERENCE]`，不把它们冒充为论文新增定理。

### 0. 统一符号表

| 符号 | 含义 | 类型/形状 | 在方法中的作用 |
|---|---|---|---|
| $S$、$A$ | 状态、动作随机变量 | 随机变量；取值分别属于状态空间和动作空间 | Mutual Information 的两个变量 |
| $s_t$、$a_t$ | 轨迹第 $t$ 步观测到的状态和执行动作 | $s_t$ 可含图像和本体状态；$a_t\in\mathbb R^{d_a}$ | 一条实际数据记录 |
| $a_{t:t+h}$ | 从 $t$ 开始的动作 chunk | 展平后约为 action chunk 长度乘 $d_a$；论文图写 $a_t,\ldots,a_{t+h}$ | Action VAE 的输入，表达短时间控制意图 |
| $T$、$T_i$ | 环境时域或第 $i$ 条轨迹长度 | 正整数 | 状态访问分布和轨迹平均的归一化因子 |
| $\pi_E(a\mid s)$ | 产生示教数据的经验专家策略 | 条件动作分布 | 希望 BC 模仿的目标分布 |
| $\pi_\theta(a\mid s)$ | 参数为 $\theta$ 的学习策略 | 条件动作分布 | 下游行为克隆模型 |
| $\rho_\pi^t(s)$ | 策略 $\pi$ 在第 $t$ 步访问状态 $s$ 的概率 | 状态分布 | 描述闭环 rollout 到了哪里 |
| $\rho_\pi(s)$ | $\rho_\pi^t$ 沿时间的平均 | $\rho_\pi(s)=\frac1T\sum_t\rho_\pi^t(s)$ | 论文式 (1) 的整体状态访问分布 |
| $D_N$ | 含 $N$ 条示教的离线数据集 | $\{\tau_1,\ldots,\tau_N\}$ | DemInf 的输入数据集 |
| $\tau_i$ | 第 $i$ 条示教 | $(s_1,a_1,\ldots,s_{T_i},a_{T_i})$ | 最终评分和筛选单位 |
| $\mathcal S(\tau)$ | 轨迹质量分数 | 标量 | 本笔记用花体 $\mathcal S$ 与状态随机变量 $S$ 区分 |
| $\kappa$ | 轨迹过滤阈值 | 标量 | 保留 $\mathcal S(\tau)>\kappa$ 的轨迹 |
| $f_s$、$f_a$ | 状态 VAE、动作 VAE 的编码映射 | 可训练神经网络 | 把高维数据压到适合 k-NN 的 latent |
| $z_s$、$z_a$ | 状态、动作 latent | $z_s\in\mathbb R^{d_s}$、$z_a\in\mathbb R^{d_a^{\mathrm{lat}}}$ | KSG 计算距离的空间 |
| $B$ | 一次随机 k-NN 批次 | 最多 1024 个 state-action 样本 | 近似全数据邻域 |
| $k$ | 第几个最近邻 | 主设置为 5、6、7 | 控制局部密度估计的尺度 |
| $\rho_{k,i}$ | 样本 $i$ 在 joint latent 中到第 $k$ 个邻居的距离 | 非负标量 | 定义状态/动作边缘空间的计数半径 |
| $n_{s,i}$、$n_{a,i}$ | 在上述半径内的状态邻居数和动作邻居数 | 非负整数 | KSG 样本分数的核心统计量 |
| $\psi(\cdot)$ | digamma 函数，即 $\log\Gamma(x)$ 的导数 | 标量函数 | 把有限样本邻居计数转成对数密度估计 |
| $\widehat I_i$ | 第 $i$ 个 state-action 对的相对 MI 贡献 | 标量 | 中间分数；不是独立样本自身的真实 MI |

符号上的两个易混点：

1. 论文中的大写 $S$ 在 $I(S;A)$ 中表示状态随机变量，在 $S(\tau)$ 中又表示轨迹评分函数。本笔记把后者重写为 $\mathcal S(\tau)$，仅为避免视觉混淆，不改变论文含义。
2. 动作空间维度和动作 latent 维度容易混淆；本笔记分别写为 $d_a$ 与 $d_a^{\mathrm{lat}}$，论文未采用这套重命名。[INFERENCE]

### 1. 式 (1)：真正想优化的是闭环状态访问分布

[PAPER, Sec. III-A]

$$
\min_{\theta}
D_{\mathrm{KL}}\!\left(\rho_{\pi_\theta}\,\middle\|\,\rho_{\pi_E}\right).
\tag{1}
$$

逐项解释：

- $\theta$：学习策略的参数；训练时真正会改变的量。
- $\rho_{\pi_\theta}(s)$：让当前策略在环境中闭环运行后，它会访问哪些状态。
- $\rho_{\pi_E}(s)$：专家示教策略闭环运行时访问的状态分布。
- $D_{\mathrm{KL}}(P\|Q)$：当样本来自 $P$ 时，用 $Q$ 解释这些样本所付出的额外信息代价。标准离散展开为：

$$
D_{\mathrm{KL}}(P\|Q)
=\sum_x P(x)\log\frac{P(x)}{Q(x)}.
$$

**公式有什么用？**

它直接描述模仿学习最终在意的闭环结果：学习策略不只是单步动作像专家，而是整个 rollout 都应停留在专家熟悉的状态区域。如果早期一个动作误差把机器人推到专家数据没覆盖的状态，后续误差可能连续累积；式 (1) 会把这种闭环偏离计入目标。

**为什么不能直接优化？**

[PAPER] 计算 $\rho_{\pi_\theta}$ 需要让当前策略反复与环境交互。纯离线数据只含 $\pi_E$ 产生的轨迹，无法知道一个正在变化的 $\pi_\theta$ 会把机器人带到哪里。因此式 (1) 是理想目标，不是 DemInf 或普通 BC 的直接训练损失。

**KL 方向为什么值得注意？**

式 (1) 使用 $D_{\mathrm{KL}}(\rho_{\pi_\theta}\|\rho_{\pi_E})$：期望在学习策略实际访问的状态上检查专家是否也覆盖这些状态。若 $\rho_{\pi_\theta}$ 走入专家概率近零的区域，代价会很大。这与后面的 BC 使用相反方向的 action-level KL 不同，是论文讨论“多峰动作难拟合”的入口。[PAPER；INFERENCE：方向解释]

### 2. 状态分布上界：单步策略偏差如何累积成闭环偏差

[PAPER, Sec. IV-A，引用 Belkhale et al. 的 Theorem 4.1]

$$
D_{\mathrm{KL}}\!\left(\rho_{\pi_\theta}\,\middle\|\,\rho_{\pi_E}\right)
\le
\frac{1}{T}\sum_{t=1}^{T}(T-t)
\,\mathbb E_{s\sim\rho_{\pi_\theta}^{t}}
\left[
D_{\mathrm{KL}}\!\left(
\pi_\theta(\cdot\mid s)\,\middle\|\,\pi_E(\cdot\mid s)
\right)
\right].
$$

参数含义：

- $\rho_{\pi_\theta}^{t}$：当前学习策略运行到第 $t$ 步时的状态分布。
- $D_{\mathrm{KL}}(\pi_\theta(\cdot\mid s)\|\pi_E(\cdot\mid s))$：在同一个状态 $s$ 下，学习动作分布和专家动作分布的差异。
- $\mathbb E_{s\sim\rho_{\pi_\theta}^{t}}$：重点检查学习策略实际会到达的状态，而不是只检查数据集里的专家状态。
- $(T-t)$：第 $t$ 步动作还有多少未来步骤可影响。越早发生的策略偏差，潜在影响的后续时间越长。
- $1/T$：沿时域归一化。

**推导思路是什么？**

论文只引用已有定理并说明使用 log-sum inequality，没有在本文完整重证。可按以下直觉理解：[PAPER；INFERENCE：分步解释]

1. 下一时刻的状态分布由“当前状态分布 × 当前策略动作分布 × 环境动力学”推进。
2. 如果两个策略在当前状态下动作分布接近，经过同一环境动力学后，下一状态分布也不会突然任意分离。
3. 用 log-sum inequality 可把混合/边缘化后的状态 KL 上界到边缘化之前的 state-action KL。
4. 从第 1 步递推到第 `T` 步，早期的 action KL 会重复影响多个未来状态，因此出现权重 `(T-t)`。
5. 对全部时间求平均，得到整体状态访问 KL 的上界。

**公式有什么用？**

它建立了论文最重要的桥梁：虽然最终关心 $\rho$ 的闭环差异，但如果每个学习策略访问到的状态上，$\pi_\theta(\cdot\mid s)$ 都能贴近 $\pi_E(\cdot\mid s)$，闭环状态分布也能被控制。于是“专家动作分布是否容易被拟合”成为数据质量问题。

**它没有证明什么？**

这个上界没有直接证明“更低 $H(A\mid S)$ 必然带来更高任务成功率”。它只说明 action-level policy divergence 与 state-visitation divergence 有联系；论文随后用“更低熵、更少模态、更少特权信息依赖”论证为什么低条件动作熵通常更容易拟合。[PAPER]

### 3. 式 (2)：行为克隆是离线可算的代理目标

[PAPER, Sec. III-A]

$$
\mathcal L_{\mathrm{BC}}(\theta)
=
\mathbb E_{s\sim\rho_{\pi_E}}
\left[
D_{\mathrm{KL}}\!\left(
\pi_E(\cdot\mid s)\,\middle\|\,\pi_\theta(\cdot\mid s)
\right)
\right].
\tag{2}
$$

逐项解释：

- 外层状态来自 $\rho_{\pi_E}$：只需专家数据，不需运行当前策略。
- 内层 KL 是 $\pi_E\|\pi_\theta$：要求学习策略在专家动作出现处给出高概率。
- 训练时改变 `theta`，专家策略和数据分布固定。

把 KL 展开：[INFERENCE，标准恒等式]

$$
\begin{aligned}
D_{\mathrm{KL}}(\pi_E\|\pi_\theta)
&=
\mathbb E_{a\sim\pi_E}
\left[
\log\pi_E(a\mid s)-\log\pi_\theta(a\mid s)
\right] \\
&=
H(\pi_E,\pi_\theta)-H\!\left(\pi_E(\cdot\mid s)\right).
\end{aligned}
$$

对固定数据集，$H(\pi_E(\cdot\mid s))$ 不随 $\theta$ 改变，因此最小化式 (2) 等价于最大化专家动作的对数似然：

$$
\min_\theta \mathcal L_{\mathrm{BC}}(\theta)
\quad\Longleftrightarrow\quad
\min_\theta
\mathbb E_{(s,a)\sim D_N}
\left[-\log\pi_\theta(a\mid s)\right].
$$

若采用固定方差 Gaussian policy，这又常退化为动作均方误差或 L2 损失；具体是否如此取决于下游策略实现，DemInf 本身不训练这个策略。[INFERENCE]

**为什么论文强调 KL 方向相反？**

- 状态分布理想目标是 $\rho_{\pi_\theta}\|\rho_{\pi_E}$。
- BC 动作目标是 $\pi_E\|\pi_\theta$。
- 两种 KL 在分布完全相等时都为零，但在多峰分布上行为不同：一个方向倾向避免落到专家低概率区，另一个方向倾向覆盖专家所有有概率的模式。

[PAPER] 如果 $\pi_E(a\mid s)$ 很简单、接近单峰或低熵，两种 KL 的方向差异不易造成严重问题；如果相同状态下存在多个冲突动作，有限容量策略更难兼顾，BC 的单步拟合也更难转化为闭环状态对齐。

### 4. 条件动作熵：为什么相同状态下的动作冲突是质量问题

条件动作熵的标准定义为：[INFERENCE]

$$
\begin{aligned}
H(A\mid S)
&=
\mathbb E_{s\sim p(s)}
\left[H\!\left(p(a\mid s)\right)\right] \\
&=
-\mathbb E_{(s,a)\sim p(s,a)}
\left[\log p(a\mid s)\right].
\end{aligned}
$$

各符号含义：

- `p(s,a)`：数据集隐含的 joint state-action 分布。
- $p(a\mid s)$：在给定状态下示教数据可能给出的动作分布。
- $-\log p(a\mid s)$：动作在该状态下越罕见，信息量/意外程度越高。
- 对所有 state-action 求期望，得到平均的不确定性。

**它与 BC 的最优损失有什么关系？**

若模型容量无限且训练充分，最优模型可以令 $\pi_\theta(a\mid s)=p_D(a\mid s)$。此时数据本身不可消除的最小负对数似然就是 $H_D(A\mid S)$。[INFERENCE] 因而高条件熵代表即便模型很好，数据也要求它表达更多多峰性或不可观测因素；有限数据、有限容量下更难。

论文给出三个理由：[PAPER, Sec. IV-A]

1. **Ease of Fit**：低熵分布通常更简单；极端情况下，给定状态只有一个动作，条件熵为零。
2. **Multimodality**：低熵通常意味着模式更少；forward/reverse KL 在单峰数据上更容易表现一致。
3. **Privileged Information**：示教者可能看到策略输入中不存在的变量 `Z`，例如相机外的物体。动作依赖 `Z` 时，仅看 `S` 会显得不可预测。

特权信息关系为：[PAPER, Sec. IV-A；推导为标准恒等式]

$$
I(A;Z\mid S)=H(A\mid S)-H(A\mid S,Z).
$$

由于熵非负，$H(A\mid S,Z)\ge 0$，所以：

$$
H(A\mid S)\ge I(A;Z\mid S).
$$

**这一步有什么用？**

如果动作强烈依赖策略看不到的 $Z$，$I(A;Z\mid S)$ 大，就迫使 $H(A\mid S)$ 不能很低。筛选低条件熵轨迹，倾向于保留那些能由当前 observation space 解释的动作。注意这不是找回缺失信息；它可能直接把依赖额外视角的示教判为难学。[PAPER；INFERENCE：最后一句]

### 5. 式 (4)：为什么质量指标是 Mutual Information

[PAPER, Sec. IV]

$$
I(S;A)=H(A)-H(A\mid S).
\tag{4}
$$

这个式子可以从 Mutual Information 的密度比定义推导：[INFERENCE，标准信息论恒等式]

$$
\begin{aligned}
I(S;A)
&=
\mathbb E_{(s,a)\sim p(s,a)}
\left[
\log\frac{p(s,a)}{p(s)p(a)}
\right] \\
&=
\mathbb E_{(s,a)\sim p(s,a)}
\left[
\log\frac{p(a\mid s)}{p(a)}
\right] \\
&=
\mathbb E_{(s,a)}[\log p(a\mid s)]
-\mathbb E_a[\log p(a)] \\
&=
H(A)-H(A\mid S).
\end{aligned}
$$

每一步的意义：

1. `p(s,a)/(p(s)p(a))` 比较 joint 分布和“状态动作彼此独立”的分布。
2. 用 $p(s,a)=p(a\mid s)p(s)$ 消去 $p(s)$，变为 $p(a\mid s)/p(a)$。
3. 若看到状态后，某动作的概率相对其边际概率明显改变，则状态提供了动作信息。
4. 对数密度比取期望，即得到平均依赖强度。

**为什么必须保留 $H(A)$？**

若只最小化 $H(A\mid S)$，任何状态都输出同一个静止动作时也可达到零条件熵：

$$
p(a=a_{\mathrm{stop}}\mid s)=1
\quad\Longrightarrow\quad
H(A\mid S)=0.
$$

但此时动作边际也只有一个取值，$H(A)=0$，所以 $I(S;A)=0$。加入 $H(A)$ 相当于正则项：要求数据中动作总体有变化，同时这些变化由状态解释，而不是随机噪声。

**为什么不用状态熵 $H(S)$ 直接衡量多样性？**

MI 也可写成：

$$
I(S;A)=H(S)-H(S\mid A).
$$

[PAPER] 但 $H(S\mid A)$ 在数据质量语境里较难解释。高状态熵也可能来自相机、背景或动力学噪声，不一定对应可学动作。作者更关心 action predictability，并且后面按整条轨迹平均局部 MI，不会显式最大化初始状态多样性。

**MI 指标真正奖励什么？**

| 数据特征 | $H(A)$ | $H(A\mid S)$ | $I(S;A)$ 趋势 | 解释 |
|---|---:|---:|---:|---|
| 所有状态都静止 | 低 | 低 | 低 | 稳定但没有状态依赖 |
| 动作随机抖动 | 高 | 高 | 低 | 多样但不可预测 |
| 相同状态动作冲突 | 中/高 | 高 | 低 | 示教者之间策略不一致 |
| 状态变化对应稳定动作变化 | 高 | 低 | 高 | 闭环控制映射清晰 |

这里的“高/低”是概念趋势，不是论文给出的数值表。[INFERENCE]

### 6. 式 (3)：如何把抽象质量变成数据筛选

[PAPER, Sec. III-B]

$$
D_N(\kappa,\mathcal S)
=
\left\{
\tau_i\;\middle|\;\mathcal S(\tau_i)>\kappa,
\ i=1,\ldots,N
\right\}.
\tag{3}
$$

参数含义：

- `D_N`：原始 `N` 条轨迹。
- `S(tau_i)`：轨迹 `i` 的评分函数。
- `kappa`：阈值；增大时保留的数据更少。
- `D_N(kappa,S)`：用于训练下游 BC 的筛选数据集。

**公式有什么用？**

它把“数据质量”操作化为一个可验证问题：在相同训练算法下，使用高分轨迹子集训练的策略是否优于全量数据或等量随机子集。论文实验通过扫描过滤数量，观察剩余数据的人工质量分和下游策略性能。[PAPER]

**为什么以整条示教为单位？**

机器人数据采集和人工质量标签常以 episode 为单位；删除整条轨迹也不会破坏任务阶段的连续性。代价是轨迹内部局部坏片段无法单独处理。[PAPER；INFERENCE：代价]

**一个重要前提：**

论文假设输入轨迹都成功完成任务。因此 `S(tau)` 只需区分“都成功时谁更可学”，而无需区分成功与失败。若把失败但动作稳定的轨迹混入，式 (3) 可能保留它们。[PAPER]

### 7. 式 (5)：从数据集级 MI 到单样本贡献

[PAPER, Sec. V]

$$
\widehat I(S;A)
=
\frac{1}{|D_N|}
\sum_{(s_i,a_i)\in D_N}
\widehat I(s_i,a_i;D_N).
\tag{5}
$$

逐项解释：

- $|D_N|$ 在这个求和语境中表示参与 MI 估计的 state-action 样本总数，而不是简单的轨迹条数；论文符号沿用 $D_N$，容易混淆。[INFERENCE]
- $\widehat I(S;A)$：整个经验数据分布的 MI 估计。
- $\widehat I(s_i,a_i;D_N)$：样本 $i$ 在相对于整个数据分布时的局部贡献。
- `hat`：它是有限数据上的估计量，不是真实分布的精确 MI。

为什么 MI 可以写成样本平均？从密度比定义看：[INFERENCE]

$$
I(S;A)
=
\mathbb E_{(s,a)\sim p}
\left[
\log\frac{p(s,a)}{p(s)p(a)}
\right].
$$

把期望换成经验均值：

$$
\widehat I(S;A)
\approx
\frac{1}{M}\sum_{i=1}^{M}
\log
\frac{\widehat p(s_i,a_i)}
{\widehat p(s_i)\widehat p(a_i)}.
$$

于是每个样本都有一个局部对数密度比贡献。KSG 的工作就是在不知道连续密度函数的情况下，用最近邻计数估计这个局部贡献。

**为什么这一步对数据筛选关键？**

普通 MI 只给整个数据集一个数，无法说哪条轨迹该删。把估计量分解成样本贡献后，才能先聚合到轨迹分数，再排序轨迹。这里的“贡献”依赖当前数据集：同一轨迹放到另一数据集，邻居关系变化，分数也会变化。[PAPER；INFERENCE：最后一句]

### 8. VAE：为什么先学习两个低维空间

[PAPER, Sec. V-A] 论文训练两个独立 VAE：

$$
z_{s,i}=f_s(s_i),
\qquad
z_{a,i}=f_a(a_i).
$$

#### 输入、输出和训练状态

| 模块 | 输入 | 输出 | 是否训练 | 后续用途 |
|---|---|---|---|---|
| State VAE encoder $f_s$ | 图像/状态 $s_i$ | $z_{s,i}\in\mathbb R^{d_s}$ | 是 | 计算状态距离和状态邻居数 |
| State VAE decoder | $z_{s,i}$ | 重建状态/图像 | 是 | 提供 VAE 重建训练信号；打分时不用 |
| Action VAE encoder $f_a$ | 动作或 action chunk | $z_{a,i}\in\mathbb R^{d_a^{\mathrm{lat}}}$ | 是 | 计算动作距离和动作邻居数 |
| Action VAE decoder | $z_{a,i}$ | 重建动作 chunk | 是 | 提供训练信号；打分时不用 |

论文未在正文写出完整 VAE loss。标准 $\beta$-VAE 常写成：[INFERENCE，不是本文明确给出的完整实现]

$$
\mathcal L_{\mathrm{VAE}}(x)
=
\mathbb E_{z\sim q_\phi(z\mid x)}
\left[-\log p_\omega(x\mid z)\right]
+
\beta
D_{\mathrm{KL}}\!\left(
q_\phi(z\mid x)\,\middle\|\,\mathcal N(0,I)
\right).
$$

- $q_\phi(z\mid x)$：encoder 给出的近似后验。
- $p_\omega(x\mid z)$：decoder 的重建分布。
- 第一项：保留足够的信息以重建输入。
- 第二项：把 latent 拉向各向同性标准 Gaussian。
- $\beta$：重建保真与 Gaussian 规整之间的权衡。

**为什么这样操作？**

1. 原始图像维度高，欧氏距离会集中，近邻不再可靠，即维度灾难。
2. KSG 需要局部距离能反映密度；VAE 把数据压到较低维度。
3. 各向同性 Gaussian prior 让不同 latent 方向的尺度更规整，避免少数大尺度维度支配距离。
4. 作者指出 k-NN MI 估计器通常在 Gaussian 数据上被评估且表现较好。
5. 状态与动作分别编码，才能在 joint 空间确定半径后，分别在 state/action 边缘空间计数。[PAPER；INFERENCE：第 5 点是方法结构解释]

**为什么选择尽可能小的 latent？**

[PAPER] latent 越高维，k-NN 需要的样本越多；但太小又会丢失决定动作的状态因素。作者采用“能够充分表达变量的最小维度”这一原则，并用附录消融检查一定范围内的鲁棒性。

**尚未核验的实现点：**

- VAE 打分时使用 posterior mean、mode 还是随机 sample；
- action 和 proprioception 的归一化方式；
- action chunk 在轨迹尾部如何 padding；
- 图像 VAE 的完整 reconstruction loss 组合。

这些必须从官方代码确认，PDF 不足以回答。[OPEN]

### 9. KSG 到底是什么

KSG 是 **Kraskov–Stögbauer–Grassberger estimator** 的缩写，由 Kraskov、Stögbauer 和 Grassberger 在 2004 年提出。它是一类针对**连续随机变量 Mutual Information** 的非参数估计器：不预设数据服从 Gaussian mixture 等具体分布，也不把空间切成固定直方图格子，而是利用每个样本附近的 $k$ 个最近邻自适应估计局部密度。[KSG-2004]

DemInf 使用 KSG 的原因不是“k-NN 能直接判断轨迹好坏”，而是：

1. DemInf 想计算 $I(S;A)$；
2. $S$ 与 $A$ 是高维连续变量，真实密度 $p(s,a)$、$p(s)$、$p(a)$ 都未知；
3. 机器人数据通常只有几十到几百条轨迹，不足以稳定训练大型 MI critic；
4. KSG 可以用局部近邻距离估计密度，而且把 joint 与 marginal 的估计绑定在**同一个局部尺度**上，减少三项熵分别估计再相减产生的偏差。[PAPER；KSG-2004]

#### 9.1 KSG 要解决的原始问题

Mutual Information 可以写成三个 differential entropy 的组合：

$$
I(Z_s;Z_a)
=
H(Z_s)+H(Z_a)-H(Z_s,Z_a).
$$

一种直接做法是分别估计 $H(Z_s)$、$H(Z_a)$ 和 $H(Z_s,Z_a)$，再相减。但连续高维空间中的 entropy 估计偏差很大；如果三项各自使用不同的近邻半径，误差不一定相消，反而可能相加。[KSG-2004]

KSG 的关键设计是：**先在 joint 空间为每个样本选择一个半径，再把同一个半径投影到两个 marginal 空间计数。**这样 joint 与 marginal entropy 使用相互耦合的局部尺度，近邻半径与体积项在 MI 组合中大部分抵消。这就是 KSG 比“分别估计三个熵再相减”更关键的地方。

#### 9.2 为什么近邻距离能表示局部密度

设批次中共有 $M$ 个样本。如果以样本 $x_i$ 为中心、半径为 $\varepsilon_i$ 的小邻域刚好容纳 $k$ 个邻居，那么局部概率质量近似为 $k/M$：

$$
p(x_i)\,V_d(\varepsilon_i)
\approx
\frac{k}{M},
$$

其中：

- $p(x_i)$ 是样本附近的未知概率密度；
- $V_d(\varepsilon_i)$ 是 $d$ 维半径 $\varepsilon_i$ 邻域的体积；
- $k/M$ 是该邻域包含的经验概率质量。

因此：

$$
\widehat p(x_i)
\approx
\frac{k}{M\,V_d(\varepsilon_i)}.
$$

这给出一个非常直观的关系：

- 第 $k$ 个邻居很近，$\varepsilon_i$ 小，局部密度高；
- 第 $k$ 个邻居很远，$\varepsilon_i$ 大，局部密度低；
- 每个样本都有自己的 $\varepsilon_i$，因此 KSG 的分辨率会随数据密度自适应，而不是像固定直方图那样所有区域使用同一网格。[KSG-2004]

上式是直觉化近似。KSG 的正式估计使用 digamma 函数修正有限样本下直接用 $\log k$、$\log M$ 带来的偏差。

#### 9.3 第一步：在 joint latent 空间确定第 k 近邻半径

[PAPER, Sec. V-B] 对第 $i$ 个样本，joint latent 为：

$$
u_i=[z_{s,i},z_{a,i}].
$$

DemInf 定义两个 joint 样本的距离为：

$$
d_{\mathrm{joint}}(i,j)
=
\max\left\{
\left\|z_{s,i}-z_{s,j}\right\|_2,
\left\|z_{a,i}-z_{a,j}\right\|_2
\right\}.
$$

把所有 $j\ne i$ 按该距离排序，第 $k$ 个邻居的距离记为：

$$
\rho_{k,i}
=
d_{\mathrm{joint}}\!\left(i,j_i^{(k)}\right),
$$

其中 $j_i^{(k)}$ 表示样本 $i$ 的第 $k$ 个 joint 近邻索引。

为什么使用 max metric？因为：

$$
d_{\mathrm{joint}}(i,j)\le\rho_{k,i}
$$

等价于同时满足：

$$
\left\|z_{s,i}-z_{s,j}\right\|_2\le\rho_{k,i},
\qquad
\left\|z_{a,i}-z_{a,j}\right\|_2\le\rho_{k,i}.
$$

于是 joint 邻域可以看作 state 邻域与 action 邻域的直积，便于在 joint 与 marginal 估计之间消去共同半径项。[PAPER；KSG-2004]

#### 9.4 第二步：把同一半径投影到两个 marginal 空间

固定 joint 半径 $\rho_{k,i}$ 后，分别计算 state latent 和 action latent 中有多少点落入该半径：

$$
n_{s,i}
=
\sum_{j\ne i}
\mathbf 1\!\left[
\left\|z_{s,i}-z_{s,j}\right\|_2
\le\rho_{k,i}
\right],
$$

$$
n_{a,i}
=
\sum_{j\ne i}
\mathbf 1\!\left[
\left\|z_{a,i}-z_{a,j}\right\|_2
\le\rho_{k,i}
\right].
$$

这里 $\mathbf 1[\cdot]$ 是指示函数：条件成立为 1，否则为 0。

三个邻域之间的关系是：

- joint 邻域要求 state 和 action 两个距离条件同时成立，里面约有 $k$ 个近邻；
- state marginal 只要求 state 接近，因此通常会包含更多点；
- action marginal 只要求 action 接近，因此通常也会包含更多点；
- $n_{s,i}$ 和 $n_{a,i}$ 越大，说明相同 state 或相同 action 单独出现得很普遍，joint 配对提供的额外依赖信息越弱。

#### 9.5 第三步：完整 KSG 估计式

对批次中的 $M$ 个样本，固定 $k$ 时，KSG 第一类估计器可写成：

$$
\widehat I_{\mathrm{KSG}}(Z_s;Z_a)
=
\psi(k)+\psi(M)
-
\frac{1}{M}
\sum_{i=1}^{M}
\left[
\psi(n_{s,i}+1)+\psi(n_{a,i}+1)
\right].
$$

其中：

- $\psi(k)$：与选择的近邻阶数有关；
- $\psi(M)$：与批次样本总数有关；
- $\psi(n_{s,i}+1)$：state marginal 的局部密度校正；
- $\psi(n_{a,i}+1)$：action marginal 的局部密度校正；
- $\psi(x)=\frac{d}{dx}\log\Gamma(x)$ 是 digamma 函数。

不同 KSG 版本和边界计数约定可能有小的常数修正；上式表达的是与 DemInf 正文相对应的 KSG-1 结构。DemInf 正文明确使用 $\le\rho_{k,i}$ 的计数约定，并只保留影响样本间排序的部分。[PAPER；KSG-2004]

把整体估计器写成样本平均：

$$
\widehat I_{\mathrm{KSG}}
=
\frac{1}{M}\sum_{i=1}^{M}\widehat i_i,
$$

其中局部贡献可以写成：

$$
\widehat i_i
=
\psi(k)+\psi(M)
-\psi(n_{s,i}+1)
-\psi(n_{a,i}+1).
$$

在同一个批次、同一个 $k$ 下，$\psi(k)+\psi(M)$ 对所有样本相同。因此 DemInf 为了轨迹排序，省略常数项，只使用：[PAPER]

$$
\widehat i_i
\propto
-\psi(n_{s,i}+1)
-\psi(n_{a,i}+1).
$$

这解释了论文公式中为什么看不到 $\psi(k)$ 和 $\psi(M)$：论文要的是**相对轨迹排名**，不是报告校准后的绝对 MI 数值。

#### 9.6 KSG 为什么能减少三项熵估计的偏差

若分别估计：

$$
H(Z_s),\qquad H(Z_a),\qquad H(Z_s,Z_a),
$$

三个估计器可能为同一样本选择三个不同的近邻半径。局部非均匀性造成的误差会分别进入三项，做：

$$
H(Z_s)+H(Z_a)-H(Z_s,Z_a)
$$

时不一定抵消。

KSG 先在 joint 空间选择唯一的 $\rho_{k,i}$，再在两个 marginal 空间使用同一个半径。对数密度估计中与 $\log\rho_{k,i}$ 和邻域体积有关的主要项因此共享并在 MI 组合中抵消，剩下的核心统计量就是 marginal 邻居数 $n_{s,i}$、$n_{a,i}$。[KSG-2004]

这也是为什么 DemInf 附录中“分别使用 Kozachenko–Leonenko 方法估计 $H(S)$、$H(A)$、$H(S,A)$ 再相减”的 KL baseline 不等同于 KSG，并且实验上不如 KSG稳定。[PAPER, Appendix B, Fig. 11]

#### 9.7 怎样直观理解邻居计数与依赖强度

考虑固定 $M$、固定 $k$：

- 如果 $Z_s$ 与 $Z_a$ 接近独立，为了在 joint 空间找到 $k$ 个点，半径需要同时覆盖 state 和 action 两个方向；投影到任何一个 marginal 后，会有很多额外点进入半径，所以 $n_{s,i}$、$n_{a,i}$ 较大，局部 MI 较低。
- 如果 $Z_s$ 几乎决定 $Z_a$，数据集中 joint 点会沿一条低维关系排列；找到 $k$ 个 joint 邻居时，投影到 marginal 后新增的点较少，所以两个计数较小，局部 MI 较高。

注意，计数“小”是相对于批大小和其他样本而言；在 DemInf 的 $\le$ 计数约定下，$n_{s,i}$、$n_{a,i}$ 不应随意设为小于 $k$。此前使用 $n_s=2,n_a=3$ 且未固定 $k$ 的示例不够严谨，本版已更正。

#### 9.8 一个完整的数值例子

设一个随机批次有 $M=1024$ 个 state-action 样本，取 $k=5$。共同常数为：

$$
\psi(5)+\psi(1024)
\approx
1.506+6.931
=8.437.
$$

样本 A 的边缘邻居数较少：

$$
n_{s,A}=6,
\qquad
n_{a,A}=7.
$$

其局部 KSG 分数约为：

$$
\begin{aligned}
\widehat i_A
&\approx
8.437-\psi(7)-\psi(8) \\
&\approx
8.437-1.873-2.016 \\
&=4.548.
\end{aligned}
$$

样本 B 的边缘邻居数很多：

$$
n_{s,B}=70,
\qquad
n_{a,B}=80.
$$

其局部分数约为：

$$
\begin{aligned}
\widehat i_B
&\approx
8.437-\psi(71)-\psi(81) \\
&\approx
8.437-4.255-4.388 \\
&=-0.206.
\end{aligned}
$$

因此 $\widehat i_A>\widehat i_B$。含义是：在这个批次和局部尺度下，A 的 state-action 配对依赖更强。它不表示 A 单独“拥有 4.548 bits 的质量”，也不表示 B 是失败动作；局部分数可以为负，且会随批次组成、latent 表征和 $k$ 改变。[INFERENCE]

#### 9.9 k 的作用：偏差与方差的权衡

- $k$ 小：邻域更局部，能保留细粒度结构，但对噪声、偶然近邻和有限样本更敏感，方差较高。
- $k$ 大：统计更平滑，方差下降，但会跨越局部结构，造成更高偏差。
- DemInf 对 $k\in\{5,6,7\}$ 分别计算并平均，相当于在几个相邻局部尺度上做 ensemble。

[RESULT] 附录比较 $\{2,3,4\}$、$\{5,6,7\}$、$\{8,9,10\}$，在所测数据上趋势相对稳健；这不保证任意 latent 维度和数据规模都对 $k$ 不敏感。

#### 9.10 KSG 的边界与 DemInf 为什么仍需要 VAE

KSG 不是高维问题的魔法解法：

- 维度升高时，近邻距离仍会集中，样本需求快速上升；
- 不同 latent 维度的尺度若差异很大，距离会被少数维度支配；
- KSG 只估计 latent 中的依赖，VAE 丢失的信息不会被恢复；
- 如果 state latent 缺少决定动作的因素，$H(A\mid S)$ 会看起来很高；
- 如果 VAE 把无关视觉差异编码得很强，近邻关系也会失真。

因此 DemInf 先使用 VAE 把状态和动作压缩到低维、近似各向同性 Gaussian 的空间，再使用 KSG。[PAPER]

#### 9.11 为什么不用 InfoNCE/MINE 直接估计

[PAPER] InfoNCE、MINE 等神经估计器要训练额外 critic 或 contrastive encoder，在机器人常见的 50–300 条示教规模上样本需求和方差较大。KSG 不训练 MI critic，只在已学习的低维 latent 上做非参数邻居统计。Fig. 7 显示，InfoNCE/MINE 在图像与 RoboCrowd 小数据设置中更不稳定。

这不等于 KSG 完全“无需训练”：两个 VAE 仍需训练；只是从 latent 到 MI 分数的映射不是神经 critic。[INFERENCE]

### 10. 随机批近似：为什么不是在整个数据集上做 k-NN

[PAPER, Sec. V-B；Appendix B-C]

完整 k-NN 搜索随 state-action 样本数增加会非常昂贵。论文采用 batch size 1024，随机打乱并遍历全数据 4 次，在 $k\in\{5,6,7\}$ 上分别计算，最终对不同遍历与不同 $k$ 的分数取平均。

其近似逻辑是：每次随机批为样本提供一个不同的局部参照子集；多次重排和多 $k$ 平均降低单次邻居偶然性的影响。

**这一步有什么代价？**

- 同一样本的分数依赖同批次中恰好出现哪些点；
- 批内邻域不等于全数据真实邻域；
- 不同任务混在同批时，任务构成会影响分数；
- 数据远小于 1024 时，实际批次与论文设定的关系需看代码。

前三点是随机批近似的直接推论，最后一点因实现未核验而标为 `[OPEN]`。[INFERENCE/OPEN]

### 11. 轨迹评分：如何从每步 MI 变成最终删除列表

[PAPER, Sec. V-C]

$$
\mathcal S(\tau)
=
\frac{1}{T}
\sum_{t=1}^{T}
\widehat I(s_t,a_t;D_N).
$$

实际流程还包含分位数裁剪。若全数据 state-action 分数的 1% 和 99% 分位分别为 $q_{0.01}$、$q_{0.99}$，可写为：[INFERENCE，等价流程表达]

$$
\widetilde I_t
=
\min\!\left{
\max\!\left(\widehat I_t,q_{0.01}\right),
q_{0.99}
\right\},
$$

$$
\mathcal S(\tau)
=
\frac{1}{T}\sum_{t=1}^{T}\widetilde I_t.
$$

各操作的目的：

- **分位数裁剪**：避免极少数异常邻域估计支配整条轨迹平均。
- **除以 $T$**：比较平均质量，而不是让长轨迹只因包含更多时间步获得更大总分。
- **时间平均**：把 noisy 的逐步估计变成相对稳定的 episode-level 分数。
- **阈值/排序**：产生可直接用于训练的数据子集。

**为什么不直接删低分时间步？**

[RESULT, Fig. 2] RoboMimic Square MH 的高质量示教中，抓取阶段约第 50–75 步的 MI 反而最低；起始自由运动和抓取后阶段更高。抓取闭合时机本来就精细、难预测，但对成功必不可少。若逐步删除低 MI，会系统性删掉任务关键接触阶段。因此论文只用时间步分数做诊断和轨迹聚合，不做 transition-level 删除。[PAPER]

**为什么平均也会产生新问题？**

- 长时间静止产生大量“动作很可预测”的时间步，可能抬高均值；
- 少量碰撞/错误片段可能被大量正常步骤稀释；
- 两条轨迹采取不同但都有效的策略时，混合数据的条件熵可能升高；
- 平均没有显式关注早期动作比晚期动作对闭环影响更大的 $(T-t)$ 权重。

前两项由论文 limitations 明确讨论停顿和聚合问题；后两项是由其目标与前述上界对照得到的推论。[PAPER/INFERENCE]

### 12. 贪心筛选：为什么删除后不重新估计 MI

[PAPER, Sec. VII-A] $\widehat I(s_i,a_i;D_N)$ 是相对于当前完整数据集 $D_N$ 的贡献。一旦删除轨迹，数据分布和所有邻居关系都改变，理论上应重新计算剩余数据的 MI，再决定下一条删除对象。

论文没有这样迭代，而是：

```text
对完整数据集打分一次 -> 固定排序 -> 按阈值删除
```

原因是每删一条就重新训练/编码/做全数据 k-NN，计算成本很高。作者因此明确称其筛选是 greedy、非全局最优。[PAPER]

可能的后果是：两条各自冗余但彼此互补的轨迹，在一次性评分下可能同时被低估；或者删除一批轨迹后，原本一般的轨迹变成剩余数据中的关键覆盖点，但固定排序不会更新。[INFERENCE]

### 13. 全部公式各自解决什么问题

| 公式 | 回答的问题 | 在论文中的用途 | 不能单独回答什么 |
|---|---|---|---|
| 式 (1) 状态访问 KL | 模仿最终想让 rollout 像谁？ | 定义闭环目标 | 离线时如何直接训练 |
| 状态分布上界 | 单步动作误差为何会影响闭环？ | 把 policy divergence 与 state divergence 接起来 | 低熵是否必然成功 |
| 式 (2) BC loss | 没有在线环境时怎样拟合专家？ | 给出离线代理目标 | 专家数据本身是否易学 |
| 条件动作熵 | 相同状态下动作有多不确定？ | 刻画可拟合性、多峰性和特权信息 | 动作是否完成任务 |
| 式 (4) MI | 如何同时要动作可预测和非恒定？ | 定义数据质量信号 | 成功、进展、安全、多任务平衡 |
| 式 (3) 筛选集 | 如何把分数转为数据子集？ | 连接质量分与下游训练 | 最优阈值如何无标签选择 |
| 式 (5) 样本分解 | 数据集 MI 如何定位到样本？ | 允许聚合成轨迹分数 | 样本分数是否跨数据集可比 |
| VAE 目标 | 高维图像/动作怎样适配 k-NN？ | 学低维、规整 latent | latent 是否保留全部任务因素 |
| KSG 局部分数 | 连续密度未知时如何估计 MI？ | 用邻居数产生相对分数 | 校准后的绝对 MI bit 数 |
| `S(tau)` | 如何减少逐步估计噪声并筛整轨迹？ | 最终排序与过滤 | 轨迹内部坏片段在哪里 |

## 论文报告的实现配置

### DemInf 的 VAE 配置

| 数据设置 | action chunk | 图像分辨率 | $z_s$ | $z_a$ | $\beta$ | 图像增强/重建 |
|---|---:|---:|---:|---:|---:|---|
| RoboMimic state | 1 | — | 12 | 6 | 0.05 | — |
| RoboMimic image | 1 | 84×84 | 16 | 6 | 0.01 | 随机缩放裁剪 0.9–0.95；image recon weight 0.005 |
| Franka | 4 | 128×128 | 24 | 16 | 0.01 | 同上 |
| RoboCrowd | 10 | 128×128 | 16 | 12 | 0.01 | 同上 |

[PAPER, Appendix Table I] 所有方法共用 Adam、学习率 `1e-4`、batch size 256。state 模型训练 50k 步、image 模型训练 100k 步。state 输入用两层、每层 512 的 ReLU MLP；图像输入以 ResNet-18 + spatial softmax，随后拼接多相机与状态特征。RoboMimic 图像模型接三层 512 MLP，Franka/RoboCrowd 使用两层 1024 MLP。

[PAPER] 附录 Fig. 16 比较 $(z_s,z_a)\in\{(8,3),(16,6),(24,9)\}$，Fig. 17 比较 $\beta\in\{0.001,0.01,0.1\}$；两项只跑 2 seeds，论文称曲线相对稳健，不能将其视为强统计结论。

### 基线实际优化什么

| 方法 | 额外训练模块 | 轨迹内每步分数 | 质量偏好 |
|---|---|---|---|
| DemInf | state/action VAE | KSG 样本 MI 贡献 | 可预测、状态依赖且多样的动作 |
| MINE | joint critic $f_\theta(s,a)$ | $f_\theta(s_i,a_i)$ | 神经 MI 估计；EMA $\alpha=0.9$ |
| InfoNCE | state/action encoder | $f_s(s_i)^\top f_a(a_i)$ | 对比学习匹配的 state-action 对 |
| VIP | value encoder | 到末帧目标的 value 增量 | 朝目标推进，适合去无关 play |
| Compatibility | 5-policy ensemble | 低方差时的低 L2 行为误差 | 与全量策略一致 |
| Uncertainty | 同一 ensemble | 预测标准差 | 主动学习中的新颖性，不等于离线质量 |
| Policy Loss | BC policy | 负 L2 预测误差 | 容易被现有策略拟合 |

[PAPER, Appendix B] 所有方法最终都以 `mean_t(h_t)` 聚合成轨迹并做相同分位数裁剪，故曲线差异主要来自上述每步分数定义。

## 实验协议：数据规模、下游训练和指标

| 评测域 | 质量标签/数据规模 | 质量排序实验 | 下游策略与评估 |
|---|---|---|---|
| RoboMimic Multi-Human | Lift/Can/Square；每任务 3 操作者、每人 100 条；标签 1/2/3 | 扫阈值，报告“过滤 episode 数—剩余轨迹平均专家分数” | 图像 BC：100k 步，3 seeds×每 seed 200 rollouts，最长 400 steps，成功率 |
| Franka | PenInCup 60、DishRack 80；各一半 expert=1、一半故意低质=0 | 同上；输入第三人称和 wrist 相机 | Diffusion Policy：200k 步；每方法 15 trial；chunk 16、执行 8；二元成功 |
| RoboCrowd/ALOHA | HiChew/TootsieRoll 各 40，HersheyKiss 100；标签 0–3；另有含无关 play 版本 | 同上；wrist+overhead 相机 | ACT：200k 步、chunk 100；每方法 10 trial；0–3 的抓糖/交付评分 |

[PAPER, Sec. VI] Franka 低质数据由操作者刻意造成（掉落、冗余长路径、抖动），可分性可能高于自然采集退化。Franka 下游评测使用从头训练、GroupNorm 的 ResNet-34，而不是 DROID 论文设置中的预训练 ResNet-50；因此不应把此数值当作 DROID 原协议的直接复现。[PAPER, Appendix D]

## 首轮实验审计

| 主张 | 直接证据 | 审计结论 |
|---|---|---|
| 能筛低质轨迹 | [RESULT] Fig. 4–6：DemInf 在 RoboMimic、Franka、RoboCrowd 的剩余数据平均专家分数通常最接近 oracle。 | 支持排序，但标签多是操作员级或人为构造低质示教。 |
| 小数据高维时优于神经 MI | [RESULT] Fig. 7：图像与 40–100 条 RoboCrowd 数据下，InfoNCE/MINE 方差较高、表现较差。 | 支持该设置下 KSG+VAE；不证明任意大规模多任务数据更优。 |
| 筛选提高下游性能 | [RESULT] Fig. 8–9：RoboMimic 有相对全量超过 10% 的改进案例；实机与随机 50% 子集比较通常更优。 | 非每项胜全量：PenInCup 全量略优，存在质量—数量权衡。 |
| 设计较稳健 | [RESULT] Fig. 10 的 `k` 范围较稳健；Fig. 11 KSG 优于 KL/BiKSG。 | 仍依赖 VAE latent、数据集与参数范围。 |

## 与 L1 基线的关系

| 维度 | Data Quality in IL | Data Assessment | DemInf |
|---|---|---|---|
| 主问题 | 哪种示教分布支持闭环模仿？ | 数据集/任务总体是否丰富且值得训练？ | 哪条成功示教更应保留？ |
| 关键量 | 动作一致性、转移多样性 | Diversity Entropy、Learnability | `I(S; A)` |
| 粒度 | 数据集与 state-action 诊断 | 数据集/任务 | state-action 估计、轨迹筛选 |

## 开放问题

1. [OPEN] 官方代码仓库、提交与预处理尚未核验。
2. [OPEN] 附录已给出 latent、动作 chunk、批次与 `k`；但动作归一化、chunk 边界 padding、VAE 的完整损失与数据预处理仍需以官方代码核验。
3. [HYPOTHESIS] 含失败、停顿或多任务语言条件的数据，应先加成功/进展约束再用 DemInf 排序，避免保留稳定但无效轨迹。

## 参考来源

- [PAPER] Joey Hejna et al., *Robot Data Curation with Mutual Information Estimators*, arXiv:2502.08623v3, 2025-04-22。本地文件：`papers/RobotDataCurationwithMutualInformationEstimators.pdf`。
- [KSG-2004] Alexander Kraskov, Harald Stögbauer, Peter Grassberger, *Estimating Mutual Information*, Physical Review E 69, 066138 (2004), DOI: [10.1103/PhysRevE.69.066138](https://doi.org/10.1103/PhysRevE.69.066138)。用于核对 KSG 的来源、共享近邻尺度、估计器结构与偏差动机。


## 更新记录

- 2026-07-31：从 `VLA/数据质量评估/notes/RobotDataCurationwithMutualInformationEstimators_reading.md` 迁移至本知识库，补充 YAML 元数据与相对链接；正文内容保持不变。
