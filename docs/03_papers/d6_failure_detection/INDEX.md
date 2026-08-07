---
title: "D6 失败检测与失败数据分析 — 论文索引"
type: index
status: active
created: 2026-08-03
updated: 2026-08-03
tags:
  - embodied-ai
  - data-quality
  - failure-detection
  - vlm
---

# D6 失败检测与失败数据分析 — 论文索引

> **本目录定位：** 实机机器人数据质量评估体系中 **A-D6（失败检测与失败数据分析）** 维度的单篇论文笔记集合，由用户 `VLA/数据质量评估/notes/D6-*.md` 阅读笔记迁移而来（2026-08-03）。
> **PDF 边界：** 以下 PDF 均位于用户源仓库 `VLA/数据质量评估/papers/`，仅作索引、不纳入知识库 Git（遵循默认边界：大型文件只记录索引）。如需查阅原文，按"本地 PDF 路径"打开源仓库。

| 短名 | 论文 | arXiv / 版本 | 发表 | 一句话定位 | 本地 PDF 路径 |
|---|---|---|---|---|---|
| AHA | A Vision-Language-Model for Detecting and Reasoning Over Failures in Robotic Manipulation | [2410.00371](https://arxiv.org/abs/2410.00371) v1 | ICLR 2025 | FailGen 程序化扰动合成失败数据 + 指令微调 LLaVA-13B，自由文本失败推理；AHA 数据集 49K+ 对 / 79 任务 | `VLA/数据质量评估/papers/AHA-A-Vision-Language-Model-for-Detecting-and-Reasoning-Over-Failures-in-Robotic-Manipulation.pdf` |
| Foresight | Failure Detection for Long-Horizon Robotic Manipulation with Action-Conditioned World Model Latents | [2606.23085](https://arxiv.org/abs/2606.23085) v1 | 2026-06-22 | 用动作条件世界模型潜变量监测长时程轨迹，仅用最终成败标签训练 + FCP 校准阈值；LIBERO-Long/ManiSkill-Long/BEHAVIOR-1K + 实机 | `VLA/数据质量评估/papers/Foresight-Failure-Detection-for-Long-Horizon-Robotic-Manipulation-with-Action-Conditioned-World-Model-Latents.pdf` |
| Guardian | Detecting Robotic Planning and Execution Errors with Vision-Language Models（原题为 Scaling Cross-Environment Failure Reasoning Data…） | [2512.01946](https://arxiv.org/abs/2512.01946) v3 | IEEE sub. (2026-03-30) | FailCoT 大规模失败推理数据集 + 多视角推理 VLM；RoboFail/RoboVQA/UR5-Fail 上 SOTA | `VLA/数据质量评估/papers/Guardian-Detecting-Robotic-Planning-and-Execution-Errors-with-Vision-Language-Models.pdf` |
| KITE | Keyframe-Indexed Tokenized Evidence for VLM-Based Robot Failure Analysis | [2604.07034](https://arxiv.org/abs/2604.07034) v1 | ICRA 2026 | 免训练、关键帧锚定 + BEV 示意的前端，把长视频压缩为可解释 token 证据喂给 VLM；RoboFAC 基准 | `VLA/数据质量评估/papers/KITE-Keyframe-Indexed-Tokenized-Evidence-for-VLM-Based-Robot-Failure-Analysis.pdf` |
| MotIF | Motion Instruction Fine-tuning | [2409.10683](https://arxiv.org/abs/2409.10683) v1 | RA-L 2024 | 运动指令微调，构建 MotIF-1K 失败/指令数据集 | `VLA/数据质量评估/papers/MotIF-Motion-Instruction-Fine-tuning.pdf` |
| Handover-Failure | A Multimodal Handover Failure Detection Dataset and Baselines | [2402.18319](https://arxiv.org/abs/2402.18319) v1 | ICRA 2024 | 多模态递手失败检测数据集 + 基线方法 | `VLA/数据质量评估/papers/A-Multimodal-Handover-Failure-Detection-Dataset-and-Baselines.pdf` |
| REFLECT | Summarizing Robot Experiences for Failure Explanation and Correction | [2306.15724](https://arxiv.org/abs/2306.15724) v4 | CoRL 2023 | 多模态摘要 → LLM 生成失败解释与纠正建议 | `VLA/数据质量评估/papers/REFLECT-Summarizing-Robot-Experiences-for-Failure-Explanation-and-Correction.pdf` |
| RoboFAC | A Comprehensive Framework for Robotic Failure Analysis and Correction | [2505.12224](https://arxiv.org/abs/2505.12224) v4 | 2026-03-22 | 三级失败分类 schema + 基准 + 纠正框架（SJTU MINT） | `VLA/数据质量评估/papers/RoboFAC-A-Comprehensive-Framework-for-Robotic-Failure-Analysis-and-Correction.pdf` |
| Robometer | Scaling General-Purpose Robotic Reward Models via Trajectory Comparisons | [2603.02115](https://arxiv.org/abs/2603.02115) v2 | RSS 2026 | 帧级进度 + 轨迹对比偏好双目标奖励模型；RBM-1M（百万轨迹）数据集 | `VLA/数据质量评估/papers/Robometer-Scaling-General-Purpose-Robotic-Reward-Models-via-Trajectory-Comparisons.pdf` |

## 维度归属与横向关系

- 主维度：**A-D6（失败检测与失败数据分析）**，跨 A-D5（数据完整性与多模态质量）、A-D4（任务贡献/奖励）、B（数据资源与采集质量）、外部验证。
- 失败数据合成同源脉络：**AHA（自由文本解释）↔ Guardian（细粒度分类 + CoT）↔ RoboFAC（结构化 schema）↔ REFLECT（多模态摘要→解释）**。
- 失败监测表征：**Foresight（世界模型潜变量）**、**KITE（关键帧 token 证据）** 偏前端/表征；**Robometer** 把失败/次优轨迹用于奖励模型训练。
- 数据集/基准：**Handover-Failure**、**MotIF-1K**、**RoboFAC benchmark**、**RBM-1M**。

## 待补充 / 待核验

- Guardian 原题（arXiv 标题）为 *Scaling Cross-Environment Failure Reasoning Data for Vision-Language Robotic Manipulation*，与用户笔记副标题（Detecting Robotic Planning…）不一致，已在索引中注明；以 arXiv 官方标题为准。
- AHA / Foresight / KITE / Robometer 四篇 2025-12 / 2026 arXiv 编号已于 2026-08-03 联网核验可访问；其余为稳定 2023–2024 编号。
- 各笔记内实验数字均为论文作者报告，知识库尚未独立复现。
