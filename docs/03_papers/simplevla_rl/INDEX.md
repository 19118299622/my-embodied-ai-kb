---
title: SimpleVLA_RL 论文索引（仅建索引）
type: paper-index
status: seed
created: 2026-07-31
updated: 2026-07-31
tags:
  - vla-post-training
  - reinforcement-learning
  - index-only
---

# SimpleVLA_RL 论文索引（仅建索引）

> 本目录为 `VLA/SimpleVLA_RL/` 的**纯索引**。按项目默认边界：PDF 仅记录路径与一句话，**不纳入 Git**。
> 详细论文笔记待后续补建。核验日期：2026-07-31（标题与摘要来自 arXiv 摘要页）。

## 索引表

| 源文件 | 论文 | arXiv | 一句话 | 笔记状态 |
|---|---|---|---|---|
| `VLA/SimpleVLA_RL/2509.09674v1.pdf` | SimpleVLA-RL: Scaling VLA Training via Reinforcement Learning（Li et al., 2025-09-11） | 2509.09674 | 基于 veRL 的 VLA 高效 RL 框架（VLA 专属轨迹采样、可扩展并行、多环境渲染、优化损失计算）；在 OpenVLA-OFT 上 LIBERO SoTA，RoboTwin 1.0&2.0 超过 π0；提出训练中「pushcut」现象（策略发现训练外的新模式）；官方代码 `github.com/PRIME-RL/SimpleVLA-RL` | 待建笔记 |

## 与已迁移内容的关联

- 同属「VLA 后训练」主线，与 [`../ript_vla/RIPT-VLA_阅读笔记.md`](../ript_vla/RIPT-VLA_阅读笔记.md) 及 `../rl_and_vla/` 三篇论文可并入 VLA 后训练专题横向比较。
- SimpleVLA-RL 强调「RL 降低对大规模数据依赖 + 实机超越 SFT」与 RIPT-VLA 的「RL 是 SFT 的补集」论点呼应。

## 更新记录

- 2026-07-31：建立索引，识别 SimpleVLA-RL（arXiv 2509.09674）及其官方代码仓库。
