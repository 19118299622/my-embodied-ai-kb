---
type: paper-note
status: active
created: 2026-08-03
updated: 2026-08-03
tags:
  - embodied-ai
  - data-quality
  - failure-detection
  - vlm
migrate_source: "VLA/数据质量评估/notes/D6-Multimodal-Handover-Failure-Detection-Dataset-and-Baselines-阅读笔记.md"
title: "A Multimodal Handover Failure Detection Dataset and Baselines"
short_name: "Handover-Failure"
authors: "Santosh Thoduka, Nico Hochgeschwender, Juergen Gall, Paul G. Plöger"
venue: "ICRA 2024"
year: 2024
paper_link: "https://arxiv.org/abs/2402.18319"
code_link: ""
project_link: ""
local_file: "A-Multimodal-Handover-Failure-Detection-Dataset-and-Baselines.pdf"
pdf_index: "VLA/数据质量评估/papers/A-Multimodal-Handover-Failure-Detection-Dataset-and-Baselines.pdf"
paper_version: "arXiv:2402.18319v1, 2024-02"
read_date: 2026-08-03
read_status: "L2-定向补充完成"
primary_dimension: "A-直接质量评估与数据筛选"
related_dimensions: ["A-D6", "A-D5", "B"]
primary_level: "视频 / 力—力矩 / 夹爪状态"
keywords:
  - handover
  - failure detection
  - multimodal
  - force-torque
  - video classification
  - temporal action segmentation
  - I3D
  - MSTCN
---

> **迁移与核验说明（2026-08-03）：** 本文档由用户原始阅读笔记迁移至知识库，内容为用户本人笔记，未作改写。论文版本 / arXiv / 官方代码 / 项目主页已在原笔记 YAML 中记录，并于 2026-08-03 联网核验（其中 4 篇 2025-12 / 2026 arXiv 编号均确认可访问）。PDF 仅作索引、不纳入知识库 Git（遵循默认边界：大型文件只记录索引，不直接纳入 Git），本地路径见同目录 `INDEX.md`。原笔记的证据强度标签 `[PAPER]`（论文陈述）/ `[RESULT]`（实验结果）/ `[INFERENCE]`（面向当前项目分析）/ `[OPEN]`（未公开或未验证）予以保留。


# A Multimodal Handover Failure Detection Dataset and Baselines 阅读笔记

> **本文档定位：**  
> 这是一份以"面向人机递接(handover)场景的多模态失败检测数据集与基线"为主线的定向补充笔记。论文的核心贡献：发布含**视频 + 力—力矩 + 夹爪/关节状态**的多模态失败检测数据集（含由人诱发的不可预防失败），并给出视频分类与时空动作分割两类基线。对当前项目（遥操质量评估）最有价值的是其**力—力矩信号对失败检测的补强**证据——这正是 D5 多模态完整性中"触觉/力"维度的失败证据，可与 Guardian/AHA 的视觉失败分类、Foresight 的在线监测对照。

> **论文定位：**  
> 该论文落在 A-D6（失败检测与失败数据分析），同时服务 A-D5（多模态完整性——力/触觉维度）。它是失败检测在具体子场景（handover）的实例与基线，与 Guardian/AHA 的通用失败分类、Foresight 的在线监测互补。

> **证据说明：**  
> `[PAPER]` 论文直接陈述；`[RESULT]` 实验结果；`[INFERENCE]` 面向当前项目分析；`[OPEN]` 未公开/未验证。页码对应 arXiv v1（2024-02）。

<a id="目录"></a>

## 目录

- [0. 阅读结论速览](#sec-0)
- [1. 论文定位与研究问题](#sec-1)
- [2. 数据集](#sec-2)
- [3. 基线方法](#sec-3)
- [4. 实验与发现](#sec-4)
- [5. 在 A-D6 / A-D5 中的角色](#sec-5)
- [6. 局限](#sec-6)
- [7. 与其他方法的关系](#sec-7)
- [8. 对当前项目的启示](#sec-8)
- [9. 最终摘要](#sec-9)
- [附录 A：术语](#appendix-a)

<a id="sec-0"></a>

## 0. 阅读结论速览

- 发布**多模态递接失败检测数据集**：视频 + 力—力矩 + 机器人关节状态，含 R2H（robot-to-human）与 H2R（human-to-robot）两类递接，失败由**人 participant 诱发**（如忽视机器人、不释放物体）的不可预防失败。
- 给出两类基线：(i) I3D 3D-CNN 视频分类；(ii) MSTCN 时空动作分割（联合分类人动作/机器人动作/整体结果）。
- 关键发现：**视频是关键模态**，力—力矩与夹爪位置进一步提升；video + F-T 组合优于 video + gripper；MSTCN 多任务（加人动作分割损失）最佳。

<a id="sec-1"></a>

## 1. 论文定位与研究问题

[PAPER] 现有递接失败检测多防物体打滑/外部扰动，缺考虑"由人引起的不可预防失败"（如忽视机器人、不释放物体）的数据集与评测。论文旨在补全这一缺口。

<a id="sec-2"></a>

## 2. 数据集

[PAPER] 数据采集含视频、力—力矩、机器人关节状态，覆盖成功与失败递接。每视频标注人的动作、机器人动作与整体结果。失败由人诱导，更贴近真实交互但分布偏理想。

<a id="sec-3"></a>

## 3. 基线方法

[PAPER]
- **视频分类（I3D）**：3D-CNN 对整个 trial 分类整体结果。
- **时空动作分割（MSTCN）**：联合分类人动作、机器人动作、整体结果，支持更细的时间定位。比较不同模态组合（video / F-T / gripper）与融合策略（早期/中期融合）。

<a id="sec-4"></a>

## 4. 实验与发现

[RESULT]
- 视频是关键模态；加入力—力矩与夹爪位置提升失败检测与动作分割准确率。
- video + F-T 组合优于 video + gripper（与 I3D-D 中期融合结论一致）。
- MSTCN 多任务（加人动作分割损失 ℒ_seg^H）最佳；仅视频的 MSTCN-A 也优于视频-only I3D-A。
- F1@{10,25,50} 与帧级准确率报告（如 MSTCN-A 三模态 F1@50 57.1、帧级 75.9）。

<a id="sec-5"></a>

## 5. 在 A-D6 / A-D5 中的角色

- 提供"**多模态(视觉 + 力—力矩)失败检测**"的具体实例与基线，是 A-D6 中场景化失败检测的代表。
- 其力—力矩信号补充了 D5 多模态完整性中"触觉/力"维度的**失败证据**——视觉无法捕捉的打滑/力异常可由力—力矩捕获。
- 与 Guardian/AHA（视觉失败分类）、Foresight（在线监测）对照：handover 场景强调人因失败，方法以多模态融合为主。

<a id="sec-6"></a>

## 6. 局限

- [OPEN] 失败由人诱导、非自然发生，分布偏理想，真实长尾失败覆盖有限。
- 基线仅在整段 trial 后分类，未做在线因果预测（论文指出可用因果卷积改造成在线）。
- 数据规模与场景有限，跨对象/跨人泛化未充分验证。
- 力—力矩 fusion 的具体增益在不同任务上不稳定（部分设定边际）。

<a id="sec-7"></a>

## 7. 与其他方法的关系

- **Guardian / AHA（A-D6）**：通用视觉失败分类；本文是 handover 子场景的多模态实例。
- **Foresight（A-D6）**：在线时变阈值监测；本文基线为离线分类，可借鉴其因果化改造做在线。
- **Kaiwu / RH20T（A-D5）**：多模态同步完整性基准；本文的力—力矩失败证据与其触觉维度呼应。

<a id="sec-8"></a>

## 8. 对当前项目的启示

- 当前遥操质检若仅有视觉，可借鉴本文明确的"力—力矩补强视觉"结论：在抓取/递接/装配等接触丰富的遥操任务中，加入力—力矩信号可提升失败检测准确率。
- handover 的"人因不可预防失败"提醒：质检不仅要查机器人侧错误，也要查人—机交互侧的异常（如人未配合）。

<a id="sec-9"></a>

## 9. 最终摘要

该论文发布含视频 + 力—力矩 + 关节状态的多模态递接失败检测数据集（含人诱发的不可预防失败），并给出视频分类与时空动作分割两类基线，证明视频是关键模态、力—力矩进一步补强。它是 A-D6 中场景化多模态失败检测的代表，对当前项目补全"力—力矩失败证据"维度有借鉴价值。

<a id="appendix-a"></a>

## 附录 A：术语

- **Handover (R2H / H2R)**：人机递接，机器人→人 / 人→机器人。
- **Force-torque (F-T)**：力—力矩传感器信号。
- **I3D**：Inflated 3D ConvNet，3D-CNN 视频分类骨干。
- **MSTCN**：Multi-Stage Temporal Convolutional Network，时空动作分割。
- **F1@{10,25,50}**：动作分割的时序交并比阈值下 F1 分数。
