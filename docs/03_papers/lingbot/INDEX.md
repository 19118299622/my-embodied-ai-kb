---
title: "LingBot 系列论文索引"
type: index
status: active
created: 2026-08-03
updated: 2026-08-03
tags:
  - embodied-ai
  - lingbot
  - vla
  - world-model
---

# LingBot 系列论文索引

> **本目录定位：** LingBot 体系（Robbyant Team 等）围绕具身智能的多篇论文深度分析，由用户 `VLA/lingbot/analysis_*.md` 笔记迁移而来（2026-08-03）。每篇分析笔记已带规范化 YAML，位于本目录。
> **PDF 边界：** 以下 PDF 位于用户源仓库 `VLA/lingbot/`，仅作索引、不纳入知识库 Git。

| 短名 | 论文 | arXiv / 版本 | 发表 | 一句话定位 | 笔记 | 本地 PDF |
|---|---|---|---|---|---|---|
| LingBot-VA | Causal World Modeling for Robot Control | `submit/7211108`（非标准投稿追踪号，待核验） | 2026-01 | 因果世界建模用于机器人控制 | [笔记](./analysis_LingBot_VA.md) | `VLA/lingbot/LingBot_VA.pdf` |
| LingBot-VLA | A Pragmatic VLA Foundation Model | [2601.18692](https://arxiv.org/abs/2601.18692) v2 | 2026-02 | 务实 VLA 基础模型（20k 小时、9 种双臂具身、Flow Matching） | [笔记](./analysis_LingBot_VLA.md) | `VLA/lingbot/lingbot_VLA.pdf` |
| LingBot-World | Advancing Open-source World Models | 无 arXiv（项目站） | 2026 | 推进开源世界模型 | [笔记](./analysis_LingBot_World.md) | `VLA/lingbot/LingBot_World.pdf` |
| LingBot-Depth | Masked Depth Modeling for Spatial Perception | 无 arXiv（项目站 + 代码） | 2026 | 面向空间感知的掩码深度建模 | [笔记](./analysis_LingBot_Depth.md) | `VLA/lingbot/lingbot_depth.pdf` |
| LingBot-Map | Geometric Context Transformer for Streaming 3D Reconstruction | 无 arXiv（项目站 + 代码） | 2026 | 面向流式三维重建的几何上下文 Transformer | [笔记](./analysis_LingBot_Map.md) | `VLA/lingbot/lingbot_map.pdf` |
| Qwen-VLA | Unifying VLA Modeling across Tasks, Environments, and Robot Embodiments | [2605.30280](https://arxiv.org/abs/2605.30280) v1 | 2026-05 | 跨任务 / 跨环境 / 跨实体的统一 VLA 建模（外部对比） | [笔记](./analysis_Qwen_VLA.md) | `VLA/lingbot/Qwen_vla.pdf` |

- **系列汇总：** [analysis_summary.md](./analysis_summary.md)（五篇 + Qwen-VLA 横向对比、技术体系关系图、关键创新点 / 数据规模 / 性能对比）。

## 维度归属

- **VLA 路线：** LingBot-VLA、Qwen-VLA（跨具身统一）→ 关联 [`docs/03_papers/x_vla/`](../x_vla/X-VLA_阅读笔记.md)（跨具身 Soft-Prompt）。
- **世界模型路线：** LingBot-VA（因果世界建模）、LingBot-World（开源世界模型）→ 关联 Phase 5 世界模型专题（待建）。
- **感知 / 重建：** LingBot-Depth（深度）、LingBot-Map（三维重建）。

## 待补充 / 待核验

- **LingBot-VA 的 arXiv 编号为用户笔记记录的 `submit/7211108`，属非标准投稿追踪号，正式公开 ID 待核验**（2026-08-03 未确认）。
- LingBot-World / Depth / Map 笔记未记录 arXiv（仅有项目站与代码仓库），以项目站为权威来源。
- 各笔记内实验数字均为论文作者报告，知识库尚未独立复现。
