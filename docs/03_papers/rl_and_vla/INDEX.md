---
title: RL_and_VLA 论文索引（仅建索引）
type: paper-index
status: seed
created: 2026-07-31
updated: 2026-07-31
tags:
  - vla-post-training
  - reinforcement-learning
  - index-only
---

# RL_and_VLA 论文索引（仅建索引）

> 本目录为 `VLA/RL_and_VLA/` 的**纯索引**。按项目默认边界：PDF 仅记录路径与一句话，**不纳入 Git**（大文件只建索引）。
> 详细论文笔记待后续按「一篇论文一个笔记」补建。
> 核验日期：2026-07-31（标题与摘要来自 arXiv 摘要页；RPD / SLIM 因文件名无 arXiv 号，标题待核验）。

## 索引表

| 源文件 | 论文 | arXiv | 一句话 | 笔记状态 |
|---|---|---|---|---|
| `VLA/RL_and_VLA/2502.05450v2.pdf` | ConRFT: A Reinforced Fine-tuning Method for VLA Models via Consistency Policy（Chen et al., 2025-02-08, v2 2025-04-14） | 2502.05450 | 离线 BC+Q-learning 提取策略、在线一致性策略（consistency policy）+ 人类介入微调 VLA；8 项真实任务在线微调 45–90 分钟达 96.3% 成功率 | 待建笔记 |
| `VLA/RL_and_VLA/2505.18719v1.pdf` | VLA-RL: Towards Masterful and General Robotic Manipulation with Scalable Reinforcement Learning（Lu et al., 2025-05-24） | 2505.18719 | 轨迹级 RL 微调自回归 VLA；用伪标签训练「过程奖励模型」缓解稀疏奖励；OpenVLA-7B 在 LIBERO 40 任务 +4.5%，匹配 π0-FAST；观察到测试时扩展迹象 | 待建笔记 |
| `VLA/RL_and_VLA/2505.19789v3.pdf` | What Can RL Bring to VLA Generalization? An Empirical Study（Liu et al., NeurIPS 2025） | 2505.19789 | 实证：PPO 微调相比 SFT 显著提升 VLA 在语义理解与执行鲁棒性上的泛化（视觉鲁棒性相当） | 待建笔记 |
| `VLA/RL_and_VLA/RPD.pdf` | 标题待核验（文件名无 arXiv 号） | — | 待联网核验 | 待建笔记 |
| `VLA/RL_and_VLA/SLIM.pdf`（约 40 MB） | 标题待核验（大文件，仅索引） | — | 约 40 MB，仅记录路径，不纳入 Git | 待建笔记 |

## 与已迁移内容的关联

- 三篇已识别论文（ConRFT / VLA-RL / 泛化实证研究）与 [`../ript_vla/RIPT-VLA_阅读笔记.md`](../ript_vla/RIPT-VLA_阅读笔记.md) 同属「VLA 后训练」主线，可汇入 [`../../02_topics/`](../../02_topics/) 下的 VLA 后训练专题横向比较。
- RIPT-VLA 用稀疏二元奖励 + DS-LOOP；VLA-RL 用过程奖励模型；ConRFT 用一致性策略 + 人类介入——三者代表不同的「后训练奖励/优化」路线，值得在专题中并列。

## 更新记录

- 2026-07-31：建立索引，识别其中 3 篇 arXiv 论文（ConRFT / VLA-RL / 泛化实证研究）；RPD、SLIM 标题待核验。
