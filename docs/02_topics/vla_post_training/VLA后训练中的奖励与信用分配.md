---
title: VLA 后训练中的奖励与信用分配
type: topic
status: seed
created: 2026-07-31
updated: 2026-07-31
tags:
  - embodied-ai
  - vla
  - reinforcement-learning
  - post-training
  - credit-assignment
---

# VLA 后训练中的奖励与信用分配

## 1. 专题边界

本专题汇总「用强化学习对视觉—语言—动作模型（Vision-Language-Action Model，VLA）做后训练」这条技术路线，聚焦两个纠缠的核心难题：

- **奖励从哪来**（reward sourcing）：实机/仿真环境能直接给出的信号通常稀疏（任务成败 $\{0,1\}$）或需要额外训练奖励模型；
- **信用如何分配**（credit assignment）：长时序多阶段操作里，远端稀疏奖励如何归因到具体动作。

边界：本专题不重复单篇论文细节（见 `docs/03_papers/` 下各笔记），只做横向比较与共性提炼；动作块/流匹配的表示层面问题见同目录「动作块与流匹配动作生成」专题（待建）。

## 2. 核心问题

- 行为克隆（Behavior Cloning，BC）/ 监督微调（Supervised Fine-Tuning，SFT）策略如何进一步提升？
- 稀疏二元奖励下，长时序任务如何做信用分配？（连接基础笔记第 3 讲 GAE）
- 确定性动作头如何支持策略梯度方法？（RIPT-VLA 的核心动机之一）
- 离线到在线微调如何控制分布漂移？
- 过程奖励模型 vs 环境二元奖励 vs 组内相对优势，各自的适用边界？
- 实机 RL 如何保证安全与样本效率？

## 3. 技术路线

| 路线 | 代表工作 | 奖励来源 | 优势/优化 | 是否需要 critic |
|---|---|---|---|---|
| 交互式后训练（RLOO+PPO） | RIPT-VLA | 环境稀疏二元 $\{0,1\}$ | DS-LOOP：RLOO 留一法优势 + PPO 裁剪 + 动态采样 | 否（RLOO 无 critic） |
| 过程奖励模型 | VLA-RL | 伪标签训练的「过程奖励模型」 | 轨迹级 RL + 课程选择 + critic warmup | 是 |
| 一致性策略 + 人类介入 | ConRFT | 离线 BC+Q / 在线一致性策略 + 人类介入 | 离线提取 + 在线微调 | 是（Q-learning 侧） |
| 规模化 RL 框架 | SimpleVLA-RL | 规则/环境奖励（探索增强） | 基于 veRL，VLA 专属采样与并行 | 是（PPO critic） |
| 组内相对优势 | GRPO / RLOO | 同 prompt 多轨迹回报的组内相对 | 去 critic，组标准化（GRPO）或留一（RLOO） | 否 |
| 实证基线 | RL→VLA 泛化实证 | PPO 微调 | 证明 PPO 微调 > SFT 泛化 | 是 |

## 4. 代表工作

- **RIPT-VLA**（已迁移，[阅读笔记](../docs/03_papers/ript_vla/RIPT-VLA_阅读笔记.md)）：VLA 训练「第三阶段」RL 交互式后训练；DS-LOOP；核心洞察「RL 是 SFT 的补集」。
- **VLA-RL**（arXiv 2505.18719，摘要级）：轨迹级 RL 微调自回归 VLA；过程奖励模型由自动切分的任务段伪标签训练；OpenVLA-7B 在 LIBERO 40 任务 +4.5%，匹配 π0-FAST；观察到测试时扩展（inference scaling）迹象。
- **ConRFT**（arXiv 2502.05450，摘要级）：离线 BC+Q-learning 提取策略、在线一致性策略（consistency policy）+ 人类介入；8 项真实任务在线 45–90 分钟达 96.3% 成功率。
- **SimpleVLA-RL**（arXiv 2509.09674，摘要级）：基于 veRL 的高效 RL 框架；OpenVLA-OFT 在 LIBERO SoTA，RoboTwin 1.0&2.0 超过 π0；提出训练中「pushcut」现象（策略发现训练外新模式）；官方代码 `github.com/PRIME-RL/SimpleVLA-RL`。
- **What Can RL Bring to VLA Generalization?**（arXiv 2505.19789，NeurIPS 2025，摘要级）：系统实证，PPO 微调相比 SFT 显著提升语义理解与执行鲁棒性泛化。
- **GRPO / RLOO**：组内相对优势估计谱系，详见 [RIPT-VLA GRPO vs RLOO 对比分析](../docs/03_papers/ript_vla/RIPT-VLA_GRPO_vs_RLOO_对比分析.md)。

## 5. 横向比较

- **是否依赖 critic**：RIPT-VLA（RLOO）、GRPO/RLOO 路线**去掉 critic**，改用组内相对回报；VLA-RL、ConRFT、SimpleVLA-RL 仍用 critic（价值网络 / Q 网络）。
- **奖励粒度**：环境二元（RIPT-VLA）最便宜但信用分配最难；过程奖励模型（VLA-RL）提供稠密但需训练且可能带偏；人类介入（ConRFT）贵但安全。
- **动作表示假设**：RIPT-VLA 明确讨论「确定性动作头如何支持策略梯度」；其余多默认连续动作扩散/确定性头，具体假设待深读。

## 6. 共同假设

1. 预训练 + SFT 提供的初始策略已具备基本能力，RL 负责「补长尾 / 提鲁棒」；
2. 在线/交互数据能覆盖离线演示未访问的状态（OOD）；
3. 稀疏或相对奖励足以驱动策略改进（配合优势估计或奖励模型）；
4. PPO 类裁剪目标是稳定后训练的默认骨架。

## 7. 已有结论与证据强度

- **强（已发表实证）**：「PPO 微调在 VLA 泛化上优于 SFT」（NeurIPS 2025 实证研究，摘要级确认）。
- **作者报告（未独立复现）**：RIPT-VLA 的 LIBERO-LONG 87.5% vs SFT 78.3% 等数字（见其笔记，明确标注为作者报告）；VLA-RL 的 +4.5%、SimpleVLA-RL 的 LIBERO SoTA、ConRFT 的 96.3% 均为**论文作者报告，本库尚未复现**，且目前仅到摘要级。
- **个人综合判断**：RIPT-VLA 的「RL 是 SFT 的补集」与 SimpleVLA-RL「RL 降低对大规模数据依赖」方向一致，可作为研究假设，尚非领域共识。

## 8. 局限与失败模式

- 稀疏二元奖励带来严重信用分配困难（研究地图问题二）；
- 冷启动失效：若初始策略成功率 $p=0$，RL 无法启动（RIPT-VLA 笔记 §11）；
- 过程奖励模型可能把错误伪标签注入训练（VLA-RL 路线风险）；
- 人类介入成本高、难规模化（ConRFT）；
- 去 critic 路线（RLOO/GRPO）在长轨迹上方差大、对组大小 $K$ 敏感（RIPT-VLA 配套 ε 分析提及 $K$ 取值论文未给）；
- 实机 RL 的安全与样本效率仍是真实部署瓶颈。

## 9. 开放问题

- 奖励粒度（二元 / 过程 / 相对）在不同任务上是否存在最优谱？
- 世界模型能否提供稠密、无偏的在线反馈，替代过程奖励模型？（连接研究地图 §2.3）
- 组内相对优势（RLOO/GRPO）的组大小 $K$ 与 PPO 内循环步数 $N$ 是否存在统一理论？（见基础笔记第 4 讲问答区）
- 测试时扩展（VLA-RL 观察）是否稳健、可否作为部署期在线提升手段？

## 10. 对当前研究的启发

- 与**数据质量评估**联动：离线数据评分器（研究地图问题四）能否复用为在线奖励/验证信号；
- 与**世界模型**联动：世界模型作为稠密反馈源，可能缓解过程奖励模型的偏置；
- 与**执行验证**联动：任务成败的二元信号正是执行验证器能给出的，RIPT-VLA 的奖励定义与执行验证天然同构；
- 与**动作块「金鱼记忆」**联动：长时序信用分配失败可能表现为阶段混淆（研究地图问题五）。

## 11. 可执行实验

1. 在 LIBERO 上复现 RIPT-VLA 的 DS-LOOP，验证 87.5% vs 78.3%（当前为作者报告）；
2. 扫描 RLOO 组大小 $K$（论文未给，待核实）对长任务成功率的影响；
3. 用 [`../../04_experiments/PPO_CartPole_可复现实验记录.md`](../../04_experiments/PPO_CartPole_可复现实验记录.md) 的 PPO 基线做 on-policy 偏离消融（$N$ 扫描），外推到 VLA 后训练。

## 12. 论文、项目和代码索引

- RIPT-VLA：项目主页（GitHub Pages 404，疑似下线）、官方代码 `github.com/Ariostgx/ript-vla`（license=null，无许可证）；见[笔记](../docs/03_papers/ript_vla/RIPT-VLA_阅读笔记.md)。
- SimpleVLA-RL：官方代码 `github.com/PRIME-RL/SimpleVLA-RL`。
- 其余论文 GitHub 待深读核验（arXiv 2502.05450 / 2505.18719 / 2505.19789）。
- 纯索引见 [`../../03_papers/rl_and_vla/INDEX.md`](../../03_papers/rl_and_vla/INDEX.md) 与 [`../../03_papers/simplevla_rl/INDEX.md`](../../03_papers/simplevla_rl/INDEX.md)（PDF 不纳入 git）。

## 13. 关联文档

- 基础：[`../../../kb/machine-learning/reinforcement-learning/长期学习笔记.md`](../../../kb/machine-learning/reinforcement-learning/长期学习笔记.md)（MDP / 策略梯度 / GAE / PPO）
- RIPT-VLA 主笔记与 3 篇配套：[`../../03_papers/ript_vla/`](../../03_papers/ript_vla/)
- 实验基线：[`../../04_experiments/PPO_CartPole_可复现实验记录.md`](../../04_experiments/PPO_CartPole_可复现实验记录.md)
- 研究地图：[`../../RESEARCH_MAP.md`](../../RESEARCH_MAP.md) §2.4

## 14. 更新记录

- 2026-07-31：建立 VLA 后训练奖励与信用分配专题；横向比较 RIPT-VLA / VLA-RL / ConRFT / SimpleVLA-RL / GRPO-RLOO / 泛化实证六条路线；明确标注各论文结论的证据强度（作者报告 vs 已发表实证 vs 个人判断）。RL_and_VLA 与 SimpleVLA_RL 详细结果待深读核验。
