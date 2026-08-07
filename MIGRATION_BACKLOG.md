---
title: 历史知识迁移清单
type: project
status: active
created: 2026-07-31
updated: 2026-07-31
tags:
  - project-maintenance
  - migration
---

# 历史知识迁移清单

## 1. 定位

该清单用于记录用户过去已经学习、讨论、复现或调研过，但尚未系统迁入 `my-embodied-ai-kb` 的内容。

它不是必须立即完成的任务列表。

> **详细逐条编目**（每个候选材料的原始路径 / 资料类型 / 研究主题 / 是否已有总结 / 是否需联网核验 / 目标目录 / 处理方式 / 来源和版权风险，以及排除项）见 [`MIGRATION_CHECKLIST.md`](./MIGRATION_CHECKLIST.md)。本文件保留为「优先级路线图」。
>
> 整体推进按五阶段：① 建立迁移清单（本阶段）→ ② 数据质量评估（已完成）→ ③ 强化学习与 VLA 后训练 → ④ VLA 论文库 → ⑤ 世界模型 / 数据生成 / 跨具身。

默认策略：

> 在当前研究真正需要引用旧知识时，再进行校验、提炼和迁移。

## 2. 迁移原则

迁移前需要判断：

1. 原材料是否完整；
2. 结论是否仍然准确；
3. 是否存在重复版本；
4. 应进入基础学习、专题、论文、实验还是想法目录；
5. 是否需要保留原始时间背景；
6. 是否需要拆成多个文档。

不要直接复制整段历史对话。

## 3. 当前已知迁移候选

| 方向 | 已有内容概况 | 建议目标目录 | 优先级 | 状态 |
|---|---|---|---:|---|
| 强化学习基础 | 马尔可夫决策过程、策略梯度、优势估计等 | `docs/01_foundations/` | 高 | 待迁移 |
| 部分可观测马尔可夫决策过程 | 已启动支线系统学习 | `docs/01_foundations/` | 中 | 待迁移 |
| 近端策略优化 | 手写 CartPole、广义优势估计、裁剪目标、价值损失 | `docs/01_foundations/` 与 `docs/04_experiments/` | 高 | 待迁移 |
| VLA 后训练 | PPO、RLOO、确定动作头问题、信用分配 | `docs/02_topics/` | 高 | 部分完成（RIPT-VLA 全迁移 + 后训练专题 + 动作块/流匹配专题；SimpleVLA-RL/ConRFT/VLA-RL 仅索引） |
| 流匹配动作生成 | 速度场、时间步、动作块、归一化 | `docs/01_foundations/` | 高 | 待迁移 |
| 视觉—语言—动作模型 | SmallVLA、OpenVLA、动作表示与多模态融合 | `docs/02_topics/` 与 `docs/03_papers/` | 高 | 部分完成（AR-VLA/X-VLA/LingBot-VLA/Qwen-VLA 笔记 + 9 个纯 PDF 模型索引） |
| 世界模型基础 | Dreamer、WorldVLN、BAGEL、Cosmos 等 | `docs/01_foundations/` 与 `docs/03_papers/` | 中 | 部分完成（世界模型+跨具身种子专题；LingBot-World/VA、DreamVLA、TriVLA、Foresight、X-VLA、Qwen-VLA、VLA_Aerial 已挂接） |
| 实机数据质量评估 | 数据集级、episode 级、多模态一致性、轨迹质量 | `docs/02_topics/` | 最高 | 已完成 |
| 数据质量论文地图 | Data Quality in Imitation Learning、Data Assessment、DemInf 等 | `docs/03_papers/` 与 `docs/02_topics/` | 最高 | 部分完成 |
| 互信息与数据筛选 | 互信息估计、KSG、轨迹筛选与下游验证 | `docs/01_foundations/` 与 `docs/02_topics/` | 高 | 待迁移 |
| VLA “金鱼记忆”问题 | 动作块停顿、相似观测下阶段混淆 | `docs/05_ideas/` | 高 | 待迁移 |
| Git 与 Agent 工作流 | 分支、任务治理、Codex 维护方式 | 可另建工程实践项目，不强制放入本库 | 低 | 暂缓 |
| Seedance 视频评测 | 钻框、环绕、数数、提示词模板 | 更适合独立项目 | 低 | 暂缓 |

## 4. 推荐首批迁移顺序

1. 实机机器人数据质量评估总览；
2. 三篇数据质量基线论文笔记；
3. episode 级质量评估模块说明；
4. 数据集级评估研究地图；
5. VLA 后训练与奖励设计专题；
6. 强化学习基础和近端策略优化实验；
7. 世界模型基础与代码复现记录。

## 5. 单次迁移任务模板

```markdown
## 迁移任务

- 原材料：
- 原始时间：
- 目标目录：
- 目标文件：
- 需要保留的结论：
- 需要重新核验的内容：
- 需要删除的过时内容：
- 关联文档：
- 完成状态：
```

## 6. 更新记录

- 2026-07-31：建立初始迁移清单，明确项目不要求一次性补齐所有历史知识。
- 2026-07-31：第一批试点迁移完成——实机数据质量评估（迁移中→已完成）：迁入专题《实机机器人数据质量评估总览》+ 4 篇支撑文档，以及 4 篇论文笔记（Data Quality in IL、Data Assessment、DemInf、VLA 数据综述）与对应 PDF；数据质量论文地图相应由待迁移改为部分完成。源材料保留在 `VLA/数据质量评估/`，知识库内以相对链接引用其代码与数据。
- 2026-07-31：建立 [`MIGRATION_CHECKLIST.md`](./MIGRATION_CHECKLIST.md)（Phase 1 详细编目）。基于 `VLA/` 父文件夹只读盘点，逐条登记 Phase 2–5 全部候选材料（原始路径/类型/主题/已有总结/需核验/目标目录/处理方式/版权风险），并落实排除规则（.git/缓存/数据集/权重/压缩包/大 PDF/公司材料）。Phase 2 已迁移项标记为已完成；Pegasus、CG-World 未发现，记为暂缓。
- 2026-07-31：Phase 3 启动。3a RIPT-VLA 完成——4 篇中文分析笔记（主笔记 + GRPO-vs-RLOO / PPO-ε / Rollout 存储与 On-Off-Policy 3 篇配套）+ 论文 PDF（LFS）迁入 `docs/03_papers/ript_vla/`；联网核验论文版本（仅 v1）、项目主页（GitHub Pages 404 失效）、官方代码（Ariostgx/ript-vla，license=null 无许可证）、实验数字（均为作者报告，未独立复现）。`MIGRATION_CHECKLIST.md` §7 Phase 3 状态由待启动改为进行中。
- 2026-07-31：Phase 3 推进至 3b–3e。3b 在 `docs/01_foundations/reinforcement_learning/`（双文档学习模式）沉淀 RL 基础（MDP / 策略梯度 / GAE / PPO），锚定用户 `spinningup-simplified` PPO 实现；3c 在 `docs/02_topics/vla_post_training/` 建立「VLA 后训练中的奖励与信用分配」专题（六路线横向比较）；3d 在 `docs/04_experiments/` 建立 PPO/CartPole 可复现实验记录（结果待回填）；3e 为 `RL_and_VLA`（5 PDF）、`SimpleVLA_RL`（1 PDF）建纯索引（PDF 不纳入 git），并联网识别其中 4 篇 arXiv 论文、RPD/SLIM 标题待核验。Diffusion / ACT / action_chunk 子方向尚未展开，留待后续。
- 2026-08-03：Phase 3 收尾 + Phase 4 启动。3c 补全「动作块与流匹配动作生成」专题（`docs/02_topics/action_chunk_flow_matching/`），基于用户 `ACT梳理.docx` 与 4 篇已核验论文（ACT 2304.13705、动作分块理论 Zhang et al. 2025-11-27、Diffusion Policy 2303.04137、π0 2410.24164），范式演进与权衡标为综合判断（未验证）。Phase 4 启动：将用户高质量笔记 AR-VLA（2 篇）、X-VLA（1 篇）改写为规范 paper-note（15 节 + YAML + 核验头）；联网核验 arXiv 2603.10126v2（RSS 2026，主页 arvla.insait.ai 可访问）、2510.10274v1（preprint，用户笔记标 ICLR 2026 未确认）。`MIGRATION_CHECKLIST.md` §7 Phase 3 改已完成、Phase 4 改进行中。LingBot 7 篇笔记与纯 PDF 模型（OpenVLA/π0/RT-1/SmolVLA 等）待后续。
- 2026-08-03（续）：数据质量评估源目录新增 D6（失败检测）9 篇笔记（`VLA/数据质量评估/notes/D6-*.md`）与对应 PDF（8-3 新建）已迁入 `docs/03_papers/d6_failure_detection/`（保留用户内容 + 规范化 YAML + 迁移核验头）；`实机机器人数据质量评估总览.md` 补 A-D6 专节（维度表 + 目录同步）。Phase 4 推进：LingBot 系列 7 篇分析笔记迁入 `docs/03_papers/lingbot/`（含 LingBot-VA/VLA/World/Depth/Map + Qwen-VLA + 汇总）；9 个纯 PDF 模型（OpenVLA/π0/RT-1/SmolVLA/TriVLA/UD-VLA/VLA-Adapter/OpenVLA-OFT/π0.5）建 `INDEX.md`（arXiv 全部联网核验，其中 SmolVLA 文件名误标 2506.00944 已纠正为 2506.01844）。Phase 5 启动：建「世界模型与跨具身 VLA（种子）」专题，挂接 LingBot-World/VA、DreamVLA、TriVLA、Foresight、X-VLA、Qwen-VLA、VLA_Aerial，并补 DreamVLA / VLA_Aerial 纯索引。所有 PDF 仅索引不进 git（默认边界）。`MIGRATION_CHECKLIST.md` §7 Phase 4 改已完成、Phase 5 改进行中（种子级）。
