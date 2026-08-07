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
migrate_source: "VLA/数据质量评估/notes/D6-Foresight-Failure-Detection-for-Long-Horizon-阅读笔记.md"
title: "Foresight: Failure Detection for Long-Horizon Robotic Manipulation with Action-Conditioned World Model Latents"
short_name: "Foresight"
authors: "Haoran Zhang, Yifu Lu, Boyang Wang, Xuhui Kang, Yen-Ling Kuo, Zezhou Cheng, Mengdi Wang, Odest Chadwicke Jenkins"
venue: "arXiv preprint"
year: 2026
paper_link: "https://arxiv.org/abs/2606.23085"
code_link: ""
project_link: "https://haoranzhangumich.github.io/Forsight_web/"
local_file: "Foresight-Failure-Detection-for-Long-Horizon-Robotic-Manipulation-with-Action-Conditioned-World-Model-Latents.pdf"
pdf_index: "VLA/数据质量评估/papers/Foresight-Failure-Detection-for-Long-Horizon-Robotic-Manipulation-with-Action-Conditioned-World-Model-Latents.pdf"
paper_version: "arXiv:2606.23085v1, 2026-06-22"
read_date: 2026-08-03
read_status: "L1-定向精读完成"
primary_dimension: "A-直接质量评估与数据筛选"
related_dimensions: ["A-D6", "A-D5", "外部验证"]
primary_level: "轨迹 / 时步（逐帧失败分）"
keywords:
  - Foresight
  - failure detection
  - long-horizon
  - world model
  - V-JEPA
  - weak label
  - conformal prediction
  - online monitoring
---

> **迁移与核验说明（2026-08-03）：** 本文档由用户原始阅读笔记迁移至知识库，内容为用户本人笔记，未作改写。论文版本 / arXiv / 官方代码 / 项目主页已在原笔记 YAML 中记录，并于 2026-08-03 联网核验（其中 4 篇 2025-12 / 2026 arXiv 编号均确认可访问）。PDF 仅作索引、不纳入知识库 Git（遵循默认边界：大型文件只记录索引，不直接纳入 Git），本地路径见同目录 `INDEX.md`。原笔记的证据强度标签 `[PAPER]`（论文陈述）/ `[RESULT]`（实验结果）/ `[INFERENCE]`（面向当前项目分析）/ `[OPEN]`（未公开或未验证）予以保留。


# Foresight: Failure Detection for Long-Horizon Robotic Manipulation with Action-Conditioned World Model Latents 阅读笔记

> **本文档定位：**  
> 这是一份以"仅用轨迹级成败弱标签，做长时程机器人失败在线监测"为主线的定向精读笔记。Foresight 用动作条件世界模型（V-JEPA 2-AC）的潜变量 mismatch 作为失败信号，并用函数型共形预测 (FCP) 校准**时变阈值**。对当前项目（遥操质量评估）最有价值的是：其**弱标签训练**（只需最终成败，无需逐帧标注）与**策略无关、即插即用**的监测定位——可作为已部署 VLA 的安全监控层。

> **论文定位：**  
> Foresight 落在 A-D6（失败检测与失败数据分析）。它与 DQAF 同属 episode/轨迹级失败监测，但 DQAF 用遥测 + 语义进度（采集期闭环），Foresight 用世界模型潜变量（策略无关、可在线、长时程）。二者可组合：Foresight 做在线失败预警，DQAF 做采集后诊断反馈。

> **证据说明：**  
> `[PAPER]` 论文直接陈述；`[RESULT]` 实验结果；`[INFERENCE]` 面向当前项目分析；`[OPEN]` 未公开/未验证。页码对应 arXiv v1（2026-06-22）。

<a id="目录"></a>

## 目录

- [0. 阅读结论速览](#sec-0)
- [1. 论文定位与研究问题](#sec-1)
- [2. 方法](#sec-2)
  - [2.1 动作条件世界模型潜变量](#sec-2-1)
  - [2.2 因果 Transformer 失败评分](#sec-2-2)
  - [2.3 函数型共形预测时变阈值](#sec-2-3)
- [3. 实验](#sec-3)
- [4. 在 A-D6 中的角色](#sec-4)
- [5. 局限](#sec-5)
- [6. 与其他方法的关系](#sec-6)
- [7. 对当前项目的启示](#sec-7)
- [8. 最终摘要](#sec-8)
- [附录 A：术语](#appendix-a)

<a id="sec-0"></a>

## 0. 阅读结论速览

- Foresight 仅用**轨迹级成败弱标签**（无逐帧时间标注）训练，通过动作条件世界模型潜变量 mismatch 输出逐时步失败分。
- 用**函数型共形预测 (FCP)** 基于留出成功 rollout 构造时变决策带，给统计可控的报警规则——阈值随时间变化而非固定。
- 把策略当**黑盒**，只需观测—动作接口，跨策略/跨具身可迁移。
- 仿真（LIBERO-Long / ManiSkill-Long / BEHAVIOR-1K，平均 8,557 步）+ 真实 ReactorX/Franka；Foresight-Transformer ROC-AUC 0.79–0.93，BEHAVIOR-1K 平衡准确率 +0.14。

<a id="sec-1"></a>

## 1. 论文定位与研究问题

[PAPER] 长时程任务失败起点模糊、密集时间标注昂贵；现有方法难在失败真正发生前预警，且阈值固定、跨策略/具身泛化弱。Foresight 把失败检测建模为**运行时轨迹监测**而非已完成 rollout 的 post-hoc 分类。

<a id="sec-2"></a>

## 2. 方法

<a id="sec-2-1"></a>

### 2.1 动作条件世界模型潜变量

[PAPER] 用 V-JEPA 2-AC 作骨干世界模型。当前观测上下文编码为 z^h（observed latent），由动作块预测的未来潜变量为 z^p（predicted latent）。若实际进展与"按计划动作应产生的未来状态"不符，z^h 与 z^p 的 mismatch 即失败信号——这本质是"意图进展"与"实际进展"的偏差。

<a id="sec-2-2"></a>

### 2.2 因果 Transformer 失败评分

[PAPER] 把 z^h、z^p 投影为 monitoring token + 位置编码，输入因果 Transformer 合成逐时步失败分 $s_t$（越高越可能走向失败）。实验显示动作条件（用动作块而非仅观测）与时序建模均必要——帧级 MLP 不足。

<a id="sec-2-3"></a>

### 2.3 函数型共形预测时变阈值

[PAPER] 不固定全局阈值，而是用 FCP 在留出成功 rollout 上构造随 rollout 时间变化的决策带（mean ± bandwidth×quantile）。给出统计保证：在显著性水平 α 下，报警率可控。这解决了"失败概率随时间变化、固定阈值失准"的问题。

<a id="sec-3"></a>

## 3. 实验

[RESULT]
- **仿真**：LIBERO-Long / ManiSkill-Long / BEHAVIOR-1K（平均 8,557 步）。Foresight-Transformer 在 BEHAVIOR-1K 平衡准确率超最佳基线 +0.14。
- **真实**：ReactorX-200 三任务（ACT/π0.5/SmolVLA）+ Franka 一任务（GR00T N1.5）。ROC-AUC 0.79–0.93，多数设定最佳。
- **跨策略/具身**：训练于一种策略可迁移到另一种（π0.5→ACT 达 0.94 ROC-AUC），且仅 Foresight 在 ReactorX→Franka 上一致迁移。

<a id="sec-4"></a>

## 4. 在 A-D6 中的角色

- 提供**策略无关、只需观测—动作接口**的在线失败监测层，可作为任何已部署 VLA 的即插即用安全监控。
- **弱标签训练**极大降低标注成本——当前项目若只有 episode 成败标签而无逐帧标注，Foresight 范式可直接复用。
- 其"意图 vs 实际"mismatch 信号与 DQAF 的"语义进度 vs 遥测"、ProgressVLA 的"进度异常"同源。

<a id="sec-5"></a>

## 5. 局限

- [OPEN] 预训练世界模型（V-JEPA 2-AC）计算/延迟大，难在板上实时，可能排除需快闭环控制的任务。
- FCP 校准依赖部署分布与留出分布匹配，实际部署漂移时需重新校准（论文未给重校准成本）。
- 极快闭环控制任务的预警可能来不及响应。
- 主要验证操纵任务，长时程非操纵（如导航）未涉及。

<a id="sec-6"></a>

## 6. 与其他方法的关系

- **DQAF（A-D3）**：采集期诊断反馈；Foresight 在线监测，二者可串成"在线预警 + 采集后诊断"。
- **Robometer（A-D6）**：离线密集奖励/排序；Foresight 在线时变阈值监测，互补。
- **REFLECT / AHA**：基于文本/VLM 的失败解释；Foresight 提供数值失败分，可作其上游触发。

<a id="sec-7"></a>

## 7. 对当前项目的启示

- 遥操质检若只标了 episode 成败（常见），Foresight 的弱标签 + 世界模型范式比需要逐帧标注的方法更省成本。
- "意图—实际 mismatch"信号可直接接入当前项目的失败预警环节，尤其适合长时程双臂任务。

<a id="sec-8"></a>

## 8. 最终摘要

Foresight 仅用轨迹级成败弱标签，通过动作条件世界模型潜变量 mismatch 与因果 Transformer 输出逐时步失败分，并用函数型共形预测校准时变阈值，实现策略无关、可在线、长时程的失败监测，在仿真与真实机器人上显著优于基线。它是 A-D6 中"在线弱标签失败监测"的代表方法，对当前项目的低成本失败预警有借鉴价值。

<a id="appendix-a"></a>

## 附录 A：术语

- **World Model**：世界模型，预测未来状态/潜变量的模型。
- **V-JEPA 2-AC**：动作条件的 Joint-Embedding Predictive Architecture 视觉世界模型。
- **FCP (Functional Conformal Prediction)**：函数型共形预测，构造统计可控的时变决策带。
- **Weak label**：弱标签，此处指仅轨迹级成败、无逐帧时间标注。
- **Policy-agnostic**：策略无关，不依赖被监测策略内部结构。
