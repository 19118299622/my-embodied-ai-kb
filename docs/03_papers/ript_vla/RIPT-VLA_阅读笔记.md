---
title: RIPT-VLA（Interactive Post-Training for VLA）阅读笔记
type: paper-note
status: active
created: 2026-07-31
updated: 2026-07-31
tags:
  - embodied-ai
  - vla
  - reinforcement-learning
  - post-training
  - ppo
---

- 资源核验日期：2026-07-31
- 上级索引：[论文笔记索引](../README.md)
- 研究地图：[具身智能研究地图](../../../RESEARCH_MAP.md)
- 配套分析：
  - [GRPO 与 RLOO/DS-LOOP 对比分析](./RIPT-VLA_GRPO_vs_RLOO_对比分析.md)
  - [PPO 裁剪参数 ε 调优分析](./RIPT-VLA_PPO_clip_epsilon_调优分析.md)
  - [Rollout 存储与 On/Off-Policy 问答](./RIPT-VLA_Rollout存储与OnOffPolicy_问答.md)
- 论文 PDF：[RIPT-VLA_2505.17016v1.pdf](./RIPT-VLA_2505.17016v1.pdf)

# RIPT-VLA：Interactive Post-Training for Vision-Language-Action Models

## 1. 基本信息

- 论文：Interactive Post-Training for Vision-Language-Action Models
- 作者：Shuhan Tan（德克萨斯大学奥斯汀分校）、Kairan Dou（南开大学）、Yue Zhao（德克萨斯大学奥斯汀分校）、Philipp Krähenbühl（德克萨斯大学奥斯汀分校）
- 时间：2025-05-22 提交
- 论文链接：<https://arxiv.org/abs/2505.17016>
- 版本状态：截至 2026-07-31，arXiv 仅有 v1，未见修订版本
- 项目主页：<https://ariostgx.github.io/ript_vla/> —— **截至 2026-07-31 该地址返回 GitHub Pages 404（Site not found），主页疑似已下线**
- 官方代码：<https://github.com/Ariostgx/ript-vla>
  - 确认依据：仓库 README 自述为 "Official implementation of RIPT-VLA"；仓库所有者 `Ariostgx` 与项目主页域名 `ariostgx.github.io` 及第一作者 Shuhan Tan 对应；仓库 `homepage` 字段回指该项目主页
  - 仓库状态（2026-07-31 查询）：168 stars，最后一次推送为 2025-06-04，此后约一年无更新
- 模型或数据：README 提及提供 HuggingFace model zoo；本次未逐一核验各权重条目的可用性
- 开源状态：代码公开，但**仓库未声明任何许可证（GitHub API `license` 字段为 `null`）**。在无明示许可的情况下，默认保留全部权利，商用或再分发存在版权风险
- 核验日期：2026-07-31
- 研究领域：具身智能 / 视觉—语言—动作模型（Vision-Language-Action Model，VLA）/ 强化学习后训练

## 2. 一句话定位

RIPT-VLA 在「大规模预训练 → 监督微调（Supervised Fine-Tuning，SFT）」之外补上**第三阶段——交互式强化学习后训练**：让 VLA 模型直接在环境中 rollout，仅凭稀疏二元成功/失败奖励，用一个无价值函数（critic-free）的 DS-LOOP 算法直接优化任务成功率。

## 3. 核心问题

现行 VLA 训练是纯监督的两阶段范式：

- **阶段一 预训练**：在多样化人类示范上学习通用视觉—运动技能，目标为

$$L_{\text{pretrain}} = -\mathbb{E}_{(c, a) \sim D_{\text{pretrain}}} \left[ \log \pi_\theta(a \mid c) \right]$$

- **阶段二 SFT**：在小规模任务特定数据上适配具体环境与指令，

$$L_{\text{sft}} = -\mathbb{E}_{(c, a) \sim D_{\text{sft}}} \left[ \log \pi_\theta(a \mid c) \right]$$

其中 $c = (g, o_1)$ 是初始上下文（任务目标 $g$ 与初始观测 $o_1$），$a$ 为动作序列。

论文指出这一范式的两个结构性局限：

| 问题 | 描述 |
|---|---|
| 离线数据隔离 | 模型只学习**模仿**交互，从未观察自身行为在环境中的真实后果，导致误差累积（compounding errors）与分布偏移（distribution gap） |
| 数据饥渴 | 高质量人类示范采集成本极高；低数据量下 SFT 性能急剧退化 |

更根本的一点是：**最大化示范动作的对数似然 $L_{\text{sft}}$ 并不等价于最大化任务成功率**。

## 4. 方法概览

RIPT-VLA 直接优化环境中的期望成功率：

$$J(\theta) = \mathbb{E}_{c \sim D_c} \left[ \mathbb{E}_{a \sim \pi_\theta(\cdot \mid c)} \left[ R(c, a) \right] \right]$$

- $D_c$：上下文数据集，**仅含初始状态与任务目标**，可从 $D_{\text{sft}}$ 直接提取，无需额外人工动作标注
- $R(c, a) \in \{0, 1\}$：轨迹级二元稀疏奖励，无形状奖励（reward shaping）、无奖励模型

算法命名为 **DS-LOOP（Dynamic-Sampling Leave-One-Out Proximal Policy Optimization，动态采样留一法近端策略优化）**，由三个部件拼装而成：留一法优势估计（RLOO）+ PPO 裁剪更新 + 动态采样过滤。

论文列出的主要贡献：

1. 提出一个简单、可扩展、适用于多种 VLA 架构的后训练范式；
2. 设计 DS-LOOP，在稀疏奖励、长周期、多任务条件下实现稳定高效的策略优化；
3. 在仅 1 条示范的极端低数据设定下，把 SFT 模型从 4% 成功率提升到 97%，且在 15 轮迭代内完成；
4. 展示跨场景（cross-scenario）与跨目标（cross-goal）的泛化能力。

## 5. 输入输出与模块结构

完整算法流程（论文伪代码的中文整理）：

```text
Algorithm: DS-LOOP (Dynamic-Sampling Leave-One-Out PPO)

输入: 预训练/SFT 后的 VLA π_θ, 奖励函数 R(c,a), 上下文数据集 D_context

for step = 1 to M do:
    1. 更新采样策略快照: π_ψ ← π_θ

    2. Rollout 收集阶段:
       while |D_rollout| < B do:
           a. 采样上下文: c = (g, o_1) ∼ D_context
           b. 分组采样 K 条轨迹: {a_k ∼ π_ψ(·|c)}, k = 1..K
           c. 环境交互获取奖励: R_k ← R(c, a_k) ∈ {0,1}
           d. 留一法基线: b_k ← (1/(K-1)) · Σ_{j≠k} R_j
           e. 优势: A_k ← R_k - b_k
           f. 【动态拒绝】若所有 A_k == 0，跳过该上下文
           g. 否则将 (c, a_k, A_k) 全部加入 D_rollout

    3. 策略优化阶段:
       for iteration = 1 to N do:
           for each mini-batch in D_rollout do:
               r = π_θ(a|c) / π_ψ(a|c)
               L_PPO = -min( r·A, clip(r, 1-ε, 1+ε)·A )
               梯度下降更新 π_θ

输出: 优化后的策略 π_θ*
```

关键超参数：

| 参数 | 含义 | 对算法行为的影响 |
|---|---|---|
| $K$ | 每个上下文采样的轨迹数（组大小） | 越大 → 基线估计越准、方差越低，但环境交互成本线性增长 |
| $B$ | 每轮收集的总轨迹数 | 越大 → 梯度估计越稳，交互成本越高 |
| $N$ | 每批数据的内循环优化步数 | $N=1$ 为严格 on-policy；$N>1$ 进入近似 on-policy / 部分 off-policy |
| $\varepsilon$ | PPO 裁剪阈值 | 越大 → 允许更大更新步长，风险更高；越小 → 更保守 |
| $M$ | 外循环总迭代轮数 | 决定总训练量 |

> 关于 rollout 具体需要存储哪些张量、$\pi_\theta$ 与 $\pi_\psi$ 的时序关系、以及显存占用估算，见配套问答：[Rollout 存储与 On/Off-Policy 问答](./RIPT-VLA_Rollout存储与OnOffPolicy_问答.md)。

## 6. 数学形式

### 6.1 从 REINFORCE 出发

$$\nabla_\theta J(\theta) = \mathbb{E}_{c \sim D_c} \left[ \mathbb{E}_{a \sim \pi_\theta(\cdot \mid c)} \left[ R(c, a) \cdot \nabla_\theta \log \pi_\theta(a \mid c) \right] \right]$$

推导（似然比技巧 / score function estimator）：

$$
\begin{aligned}
\nabla_\theta J(\theta) &= \nabla_\theta \mathbb{E}_{c} \mathbb{E}_{a \sim \pi_\theta(\cdot \mid c)} [R(c, a)] \\
&= \mathbb{E}_{c} \left[ \sum_a R(c, a) \nabla_\theta \pi_\theta(a \mid c) \right] \\
&= \mathbb{E}_{c} \left[ \sum_a R(c, a) \pi_\theta(a \mid c) \nabla_\theta \log \pi_\theta(a \mid c) \right] \\
&= \mathbb{E}_{c} \mathbb{E}_{a \sim \pi_\theta(\cdot \mid c)} \left[ R(c, a) \cdot \nabla_\theta \log \pi_\theta(a \mid c) \right]
\end{aligned}
$$

直觉：$R=1$ 就抬高该 rollout 的对数概率，$R=0$ 就不更新。问题在于**方差极高**——所有成功轨迹被同等鼓励，完全忽略上下文难度差异。

### 6.2 基线与优势

引入只依赖上下文、不依赖动作的基线 $b(c)$：

$$\nabla_\theta J(\theta) = \mathbb{E}_{c} \mathbb{E}_{a \sim \pi_\theta} \left[ (R(c, a) - b(c)) \cdot \nabla_\theta \log \pi_\theta(a \mid c) \right]$$

无偏性来自 $\mathbb{E}_{a \sim \pi_\theta} [b(c) \nabla_\theta \log \pi_\theta(a \mid c)] = b(c) \nabla_\theta \mathbb{E}_a[1] = 0$，因此基线只改变方差、不改变期望梯度方向。

优势 $A(c,a) = R(c,a) - b(c)$：$A>0$ 鼓励，$A<0$ 抑制，$A=0$ 无梯度信号。

### 6.3 RLOO：留一法优势估计

在 VLA 场景中用价值网络 $V_\phi(c)$ 估计基线有三个障碍：

1. 稀疏奖励 + 长周期下，时序差分（Temporal Difference，TD）误差极度噪声化，信用分配模糊；
2. 需要与 7B 级策略网络同量级的价值网络，显存与训练成本剧增；
3. 在有限 rollout 上训练奖励/价值模型容易出现奖励黑客（reward hacking）。

RLOO（REINFORCE Leave-One-Out）改用同上下文其余样本的经验均值作基线：

$$b_k = \frac{1}{K-1} \sum_{j \neq k} R(c, a_j), \qquad A_k = R(c, a_k) - b_k$$

$$g_{\text{RLOO}} = \frac{1}{K} \sum_{k=1}^K A_k \cdot \nabla_\theta \log \pi_\theta(a_k \mid c)$$

三个值得强调的性质：

- **纯蒙特卡洛基线**，完全摆脱 critic，显存与实现成本大幅下降；
- **自适应难度校准**：上下文全失败时 $b_k=0$ 且 $R_k=0$ → 优势全零；全成功时 $b_k=1$ 且 $R_k=1$ → 优势同样全零。学习信号天然集中在「部分成功、部分失败」的有区分度上下文上；
- **方差由 $K$ 控制**：$K$ 越大基线越准，但成本线性增长。

### 6.4 PPO 裁剪目标

定义重要性比率 $r = \dfrac{\pi_\theta(a \mid c)}{\pi_\psi(a \mid c)}$，其中 $\pi_\psi$ 是采样时的策略快照：

$$L_{\text{PPO}} = -\min \left( r \cdot A,\ \text{clip}(r, 1-\varepsilon, 1+\varepsilon) \cdot A \right), \qquad \varepsilon = 0.2$$

| 组件 | 含义 | 效果 |
|---|---|---|
| $r \cdot A$ | 标准策略梯度目标 | 正优势增大 $\pi_\theta(a\mid c)$，负优势减小 |
| $\text{clip}(r, 1-\varepsilon, 1+\varepsilon) \cdot A$ | 裁剪版本 | 将 $r$ 限制在 $[0.8, 1.2]$ |
| $\min(\cdot,\cdot)$ | 取较小者 | $A>0$ 时防止过度增大 $r$；$A<0$ 时防止过度减小 $r$ |

- **正优势**：期望 $r>1$；当 $r > 1+\varepsilon$ 时目标被截断，防止对单条好轨迹过度优化、保住探索能力。
- **负优势**：期望 $r<1$；当 $r < 1-\varepsilon$ 时 $\text{clip}(r)\cdot A$ 绝对值更小被 $\min$ 选中，防止过度惩罚导致策略崩溃。

> ε 的语义、与 $N$/$K$/动态采样的联动，以及自适应 ε 的可能性，见配套分析：[PPO 裁剪参数 ε 调优分析](./RIPT-VLA_PPO_clip_epsilon_调优分析.md)。

### 6.5 LOOP：RLOO + PPO

$$
\begin{aligned}
\text{采样} &: \{a_k \sim \pi_\psi(\cdot \mid c)\}_{k=1}^K \\
\text{优势} &: A_k = R(c, a_k) - \frac{1}{K-1}\sum_{j \neq k} R(c, a_j) \\
\text{更新} &: \max_\theta \ \mathbb{E}\left[ L_{\text{LOOP}}(c, a, A) \right] \quad \text{(PPO 裁剪目标)}
\end{aligned}
$$

与标准 Actor-Critic 的对比：

| 特性 | 标准 Actor-Critic（PPO + GAE） | LOOP |
|---|---|---|
| 基线估计 | 训练价值网络 $V_\phi$ | 留一法蒙特卡洛基线，无 critic |
| 信用分配 | GAE 的 TD($\lambda$)，时间步级 | 轨迹级蒙特卡洛，无步级信用分配 |
| 显存开销 | 高（额外价值网络） | 低（仅策略网络） |
| 适用场景 | 密集奖励、较短周期 | 稀疏二元奖励、长周期、多任务 |
| 偏置—方差 | 有偏（bootstrapping）、低方差 | 无偏（MC）、方差较高（由 $K$ 控制） |

LOOP 契合 VLA 的三点理由：奖励本就是轨迹级二元信号（GAE 的 TD 误差在此反而是噪声源）；7B 级模型去掉 critic 可省下大约一半显存；多任务下不同任务成功率差异大，RLOO 天然自校准。

### 6.6 动态采样

多任务环境中，「已解决」（$R_k=1,\forall k$）与「无法解决」（$R_k=0,\forall k$）的上下文都会产生全零优势、零梯度。把它们放进批次不仅浪费算力，还会**稀释有效梯度信号**。DS-LOOP 加了一行过滤：

```text
if all A_k == 0:
    continue   # 跳过该上下文，不加入 D_rollout
```

论文与源笔记给出的四点有效性解释：

1. **信噪比提升**：$D_{\text{rollout}}$ 中有效样本占比从 $p$ 提升至接近 100%；
2. **避免梯度抵消**：防止随机 mini-batch 划分出「整批无信号」的无效更新；
3. **隐式课程学习**：训练初期大量上下文全失败被跳过，随策略变强逐步进入训练集，形成基于策略能力的自适应课程；
4. **计算效率**：被跳过的上下文不消耗 PPO 更新步。

## 7. 数据与实验设置

- **基准**：LIBERO（含 LIBERO-90、LIBERO-LONG 长周期子套件）、MetaWorld45（45 个操作任务）
- **骨干模型**：QueST（轻量，约 20M 参数）与 OpenVLA-OFT（7B 级）
- **上下文数据集**：从 SFT 数据中提取初始状态与任务目标，**不需要新增动作标注**
- **奖励**：环境返回的轨迹级二元成功/失败信号
- **内循环步数**：OpenVLA-OFT 用 $N=1$，QueST 用 $N=20$；$\varepsilon = 0.2$（PPO 默认值，论文未做消融）
- 论文未明确给出 $K$ 的取值

## 8. 主要结果

> 以下均为**论文作者报告**的数字，本仓库尚未独立复现。

| 实验设定 | SFT 基线 | RIPT-VLA | 提升 |
|---|---|---|---|
| QueST + LIBERO-90 | 88.6% | **94.3%** | +5.7 pp |
| QueST + LIBERO-LONG（5-shot） | 50.2% | **71.4%** | +21.2 pp |
| QueST + MetaWorld45（5-shot） | 63.6% | **76.0%** | +12.4 pp |
| OpenVLA-OFT + LIBERO Suites | 96.7% | **97.5%** | +0.8 pp |
| 1-shot 极端低数据 | 4% | **97%** | +93 pp（15 轮迭代内） |

**跨场景泛化（cross-scenario）**：任务目标相同（如「打开炉灶并放上煎锅」），但场景背景布局与物体配置从 A 换到 B。1-shot 下 SFT 平均仅约 5% 成功率；RIPT-VLA 最高提升 93.7 pp（3.5% → 97.2%）；3–5 条示范即可接近 100%。

**跨目标泛化（cross-goal）**：场景相同、目标不同（如「红杯放左盘」对「红杯放右盘」）。难度更高：3-shot SFT 平均仅 0.7%，RIPT-VLA 提升至平均 59.7%，最好案例 84.7%；10-shot 下 RIPT-VLA 79.7%、SFT 29.4%。

**消融与鲁棒性**：

| 实验 | 发现 |
|---|---|
| 去掉动态采样 | 平均成功率 −3.3 pp（93.5% → 90.2%）；LIBERO-LONG 上差距更大（87.5% 对 78.3%，约 −9.2 pp） |
| 上下文数据集规模 | 上下文越多泛化越强，且扩充上下文**不需要额外人工标注** |
| 初始状态方差 | 将 LIBERO-LONG 物体初始位置标准差（约 2.5 cm）放大到 7 倍（约 17.5 cm），RIPT-VLA 仍保持对 SFT 的显著优势 |

> 数字来源说明：表格与消融数字来自源笔记对论文正文/表 3 的整理；初始状态方差一节的「2.5 cm → 17.5 cm（7 倍）」表述与二手中文报道一致。若后续要引用具体数值，建议回到 PDF 逐一比对表格编号。

## 9. 最有特色的贡献

1. **把「无价值函数 RL」在 VLA 上做成了可用工程方案**。RLOO 已有（Kool et al. 2019），PPO 已有，动态采样思想也不新；RIPT-VLA 的贡献是把三者组合成一个在稀疏二元奖励 + 长周期 + 多任务下真正稳定的配方，并验证其对轻量与 7B 两级骨干都成立。
2. **上下文比动作便宜**。$D_c$ 只需初始状态和任务目标，扩展成本远低于扩展示范数据集——这把「数据规模」这个瓶颈从「标注动作」转移到了「摆初始状态」。
3. **1-shot → 97% 的极端数据效率结果**，说明 SFT 只要能提供非零成功率的「火种」，RL 就能接手放大。

## 10. 与已有工作的关系

| 编号 | 工作 | 关联 |
|---|---|---|
| [4] | Ahmadian et al. — LOOP | LOOP 算法基础 |
| [9] | DeepSeekMath — GRPO | 大语言模型推理中的 RL 后训练思路，与 DS-LOOP 同属组内相对优势估计谱系 |
| [16] | Kool et al. — RLOO | 留一法 REINFORCE 基线 |
| [28] | Schulman et al. — PPO | 近端策略优化 |
| [38] | Zhai et al. — VLA + RL | 用 RL 微调 VLA 的先前工作 |
| [40] | Zitkovich et al. — RT-2 | VLA 模型的定义性工作 |

统一视角：GRPO 与 RLOO/DS-LOOP 都是**组内相对优势估计（group-relative advantage estimation）** 的实例，分叉点在于「是否做标准差归一化」与「是否加显式 KL 正则」，而这取决于奖励是学习出来的连续值还是环境返回的二元真值。详细的十维度对比见 [GRPO 与 RLOO/DS-LOOP 对比分析](./RIPT-VLA_GRPO_vs_RLOO_对比分析.md)。

## 11. 局限与失败模式

1. **必须有可交互环境**。RIPT-VLA 假定能反复 rollout 并拿到奖励。对物理机器人，在线交互昂贵且有安全风险——这是它落到实机上最直接的障碍。
2. **二元奖励粒度太粗**。0/1 无法区分「差一点就成功」和「完全跑偏」，学习信号的分辨率受限。
3. **轨迹级信用分配**。LOOP 不对序列内单步动作分配信用，难以做动作序列内部的精细调整。
4. **冷启动失效**。若 SFT 后成功率严格为零（连偶然成功都没有），留一法基线恒为零、优势恒为零，动态采样会拒绝所有上下文，RL 根本无法启动。这是该方法的硬边界。
5. **超参数研究不充分**（个人判断）。论文未给出 $K$ 的取值，也未对 $\varepsilon$ 做消融，而配套分析显示 $K$ 直接决定基线精度与动态采样通过率，很可能是最有杠杆的超参。
6. **工程可持续性存疑**（截至 2026-07-31 的观察）。项目主页已 404，官方仓库最后推送停在 2025-06-04 且无许可证声明，复现与二次开发存在实际风险。

## 12. 对当前研究的参考价值

对照 [`RESEARCH_MAP.md`](../../../RESEARCH_MAP.md) 的关注点：

- **数据闭环 / 数据质量**：RIPT-VLA 提供了一条「不靠加标注、靠加交互」的能力提升路径。它的 $D_c$ 概念值得借用——在数据质量评估里，「初始状态覆盖」可能是一个比「轨迹条数」更有信息量的指标。动态采样的「保留有区分度样本」逻辑，与数据筛选中的「保留高信息量轨迹」在动机上同构。
- **VLA 后训练**：这是把 RL 接到 VLA 上的一个干净基线，值得作为后训练专题的参照系。
- **执行验证 / 运行时验证**：论文默认「环境就是完美的奖励模型」。真实机器人上不成立——如何得到可靠的成功判定，正好落在运行时验证这条线上，参见 [VLA 运行时验证与在线学习](../../02_topics/vla_runtime_verification_online_learning/)（若该专题已建立相关条目）。
- **世界模型**：论文自己提出的未来方向之一就是用世界模型做 model-based rollout 以降低物理交互成本——这是世界模型与 RL 后训练的天然接口。
- **强化学习**：DS-LOOP 是「稀疏奖励 + 无 critic + 组内相对优势」这一族方法在机器人上的代表实现，可作为 RL 基础笔记的落点。

## 13. 可迁移实验

1. **冷启动阈值实验**：系统扫描 SFT 成功率 $p \in \{0\%, 1\%, 2\%, 4\%, 8\%\}$，在固定 $K$ 下测量 DS-LOOP 能否启动及所需迭代数，量化「火种」的最低门槛。理论预期：非全零优势概率为 $1 - p^K - (1-p)^K$，$p=4\%$、$K=8$ 时约 28%。
2. **$K$ 的消融**：在 $K \in \{2,4,8,16\}$ 下比较基线方差、动态采样通过率、总环境交互次数与最终成功率，验证「$K$ 是最有杠杆超参」的判断。
3. **$\varepsilon \times N$ 联动消融**：在 $N=20$ 下测 $\varepsilon \in \{0.1, 0.2, 0.3\}$ 及线性调度 $0.1 \to 0.3$，检验 $\varepsilon=0.2$ 在内循环后期是否偏保守。
4. **奖励粒度实验**：把二元奖励换成分级奖励（如「接触/抓起/移动到位/完成」四级），观察是否缓解冷启动与信用分配粗糙的问题。
5. **数据质量视角的复用**：用 DS-LOOP 的动态采样通过率作为「上下文难度分布」的度量，反过来评估现有示范数据集的初始状态覆盖是否均衡。

## 14. 待验证问题

- $K$ 的实际取值是多少？论文正文未给，需要回到官方仓库配置文件确认（尚未做）。
- 「−3.3 pp」「87.5% vs 78.3%」等消融数字对应论文中哪张表的哪一行？需回 PDF 核对表格编号。
- HuggingFace model zoo 中各权重是否仍可下载？未逐项核验。
- 官方仓库无许可证——若要在本研究中复用其代码，需先联系作者确认授权。
- KL 正则在 VLA 后训练中是否真的多余？当使用 LoRA 或冻结部分层时，约束策略偏离 SFT 初始化可能有助于保持泛化能力，这一点论文未讨论，值得实验验证。
- 动态采样的阈值能否从「全零即跳过」改为「优势方差 > 阈值」，从而更细致地筛选？

## 15. 更新记录

| 日期 | 变更 |
|---|---|
| 2026-07-31 | 从 `VLA/RIPT-VLA/RIPT-VLA_论文解析.md` 迁移并按论文笔记模板重构；联网核验 arXiv 版本（仅 v1）、项目主页（已 404）、官方代码仓库（`Ariostgx/ript-vla`，无许可证，最后推送 2025-06-04）；补充局限、可迁移实验与待验证问题 |
