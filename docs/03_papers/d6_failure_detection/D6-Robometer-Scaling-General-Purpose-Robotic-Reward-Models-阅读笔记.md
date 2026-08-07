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
migrate_source: "VLA/数据质量评估/notes/D6-Robometer-Scaling-General-Purpose-Robotic-Reward-Models-阅读笔记.md"
title: "Robometer: Scaling General-Purpose Robotic Reward Models via Trajectory Comparisons"
short_name: "Robometer"
authors: "Anthony Liang, Yigit Korkmaz, Jiahui Zhang, Minyoung Hwang, Abrar Anwar, Sidhant Kaushik, Aditya Shah, Alex S. Huang, Luke Zettlemoyer, Dieter Fox, Yu Xiang, Anqi Li, Andreea Bobu, Abhishek Gupta, Stephen Tu, Erdem Biyik, Jesse Zhang"
venue: "Robotics: Science and Systems (RSS) 2026"
year: 2026
paper_link: "https://arxiv.org/abs/2603.02115"
code_link: "https://robometer.github.io/"
project_link: "https://robometer.github.io/"
local_file: "Robometer-Scaling-General-Purpose-Robotic-Reward-Models-via-Trajectory-Comparisons.pdf"
pdf_index: "VLA/数据质量评估/papers/Robometer-Scaling-General-Purpose-Robotic-Reward-Models-via-Trajectory-Comparisons.pdf"
paper_version: "arXiv:2603.02115v2, 2026-05-13"
read_date: 2026-08-03
read_status: "L1-定向精读完成"
primary_dimension: "A-直接质量评估与数据筛选"
related_dimensions: ["A-D6", "A-D4", "A-D5", "外部验证"]
primary_level: "轨迹 / 帧（progress+success）/ 轨迹对（偏好）"
keywords:
  - Robometer
  - reward model
  - trajectory comparison
  - progress prediction
  - preference learning
  - RBM-1M
  - failure detection
  - data retrieval
---

> **迁移与核验说明（2026-08-03）：** 本文档由用户原始阅读笔记迁移至知识库，内容为用户本人笔记，未作改写。论文版本 / arXiv / 官方代码 / 项目主页已在原笔记 YAML 中记录，并于 2026-08-03 联网核验（其中 4 篇 2025-12 / 2026 arXiv 编号均确认可访问）。PDF 仅作索引、不纳入知识库 Git（遵循默认边界：大型文件只记录索引，不直接纳入 Git），本地路径见同目录 `INDEX.md`。原笔记的证据强度标签 `[PAPER]`（论文陈述）/ `[RESULT]`（实验结果）/ `[INFERENCE]`（面向当前项目分析）/ `[OPEN]`（未公开或未验证）予以保留。


# Robometer: Scaling General-Purpose Robotic Reward Models via Trajectory Comparisons 阅读笔记

> **本文档定位：**  
> 这是一份以"用帧级进度 + 轨迹间偏好双目标训练通用机器人奖励模型"为主线的定向精读笔记。Robometer 的核心洞见是：人类用**相对比较（A 比 B 好）**比绝对评分更自然准确，因此把"轨迹对偏好比较"作为辅助监督，使大量无标注的失败/次优轨迹也能参与奖励学习。对当前项目（遥操质量评估）最有价值的是：其**进度曲线 + 偏好排序**可直接做失败检测与数据检索，是 ProgressVLA 进度估计的规模化扩展。

> **论文定位：**  
> Robometer 落在 A-D6（失败检测与失败数据分析），跨 A-D4（任务价值/下游贡献）与 A-D5（进度评估）。它的奖励模型本质上产出"这条轨迹有多好/多差"的 dense reward，与 ProgressVLA 的进度估计同属"用进展判断数据价值"脉络，但 Robometer 用 1M 数据 + 偏好学习把规模与泛化推到通用级别。

> **证据说明：**  
> `[PAPER]` 论文直接陈述；`[RESULT]` 实验结果；`[INFERENCE]` 面向当前项目分析；`[OPEN]` 未公开/未验证。页码对应 arXiv v2（2026-05-13）。

<a id="目录"></a>

## 目录

- [0. 阅读结论速览](#sec-0)
- [1. 论文定位与研究问题](#sec-1)
- [2. 方法：双目标训练](#sec-2)
  - [2.1 帧级进度/成功损失](#sec-2-1)
  - [2.2 轨迹比较偏好损失](#sec-2-2)
  - [2.3 数据增强与训练输入](#sec-2-3)
- [3. RBM-1M 数据集](#sec-3)
- [4. 实验](#sec-4)
- [5. 在 A-D6 中的角色](#sec-5)
- [6. 局限](#sec-6)
- [7. 与其他方法的关系](#sec-7)
- [8. 对当前项目的启示](#sec-8)
- [9. 最终摘要](#sec-9)
- [附录 A：术语](#appendix-a)

<a id="sec-0"></a>

## 0. 阅读结论速览

- Robometer = 通用机器人奖励模型，训练用**双目标**：帧级 progress/success 损失（锚定奖励量级）+ 轨迹对偏好损失（跨轨迹全局序约束）。
- 关键洞见：偏好比较只需"哪个更好"的二元判断，可自然扩展到**无标注的失败轨迹**，突破了"绝对进度标签只能用于专家演示"的限制。
- 发布 **RBM-1M**：100 万+ 多具身轨迹，含大量次优/失败数据，21 种机器人。
- 在 RoboRewardBench 偏好准确率、RBM-EVAL 的 VOC（奖励—进度相关性）与 Kendall τ（轨迹排序）上显著优于 GVL/VLAC/RoboDopamine/TOPReward。
- 下游覆盖在线/离线 RL、失败检测、IL 数据检索；6 个 OOD 实机场景验证。

<a id="sec-1"></a>

## 1. 论文定位与研究问题

[PAPER] 现有通用奖励模型只依赖专家演示的绝对进度标签（0→1 线性插值）。问题：(1) 失败轨迹的进度非单调、标注昂贵且歧义大，大量次优数据被丢弃；(2) 仅轨迹内局部监督，缺全局校准。

<a id="sec-2"></a>

## 2. 方法：双目标训练

<a id="sec-2-1"></a>

### 2.1 帧级进度/成功损失

[PAPER] 在专家轨迹上回归每帧 progress 与 success，锚定奖励的量级与语义（成功=1，失败=缺失）。

<a id="sec-2-2"></a>

### 2.2 轨迹比较偏好损失

[PAPER] 对同一任务的一对轨迹预测偏好（哪个更好）。偏好标签由轨迹对自动构造（无需人工），施加全局序约束，使模型学到跨轨迹、跨机器人、跨视角的相对排序。这是利用失败数据的关键——失败轨迹虽无绝对进度，但其"比专家差"的相对关系可自然获得（例如 video rewind 增强生成失败版，模型学"原版优于增强版"）。

<a id="sec-2-3"></a>

### 2.3 数据增强与训练输入

[PAPER] 把专家与次优轨迹配对，训练帧级 progress/success 于专家、偏好于轨迹对；并增强轨迹（video rewind 模拟失败）。模型输入为视频 + 语言指令，输出逐帧 progress/success + 轨迹对偏好。

<a id="sec-3"></a>

## 3. RBM-1M 数据集

[PAPER] 聚合 100 万+ 轨迹，覆盖 21 具身，来源含 Open-X、AGIBotWorld、Epic-Kitchens（人类视频）、RH20T（人—机配对）、LIBERO（仿真）等。刻意强调 viewpoint/scene/embodiment 多样性而非单纯数量；次优/失败数据占相当比例，支撑偏好学习。

<a id="sec-4"></a>

## 4. 实验

[RESULT]
- **Reward alignment (VOC Pearson r)**：RBM-EVAL-ID 0.79–0.80，远超基线（GVL/VLAC 约 0.16）。
- **Trajectory ranking (Kendall τ)**：RBM-EVAL-OOD 0.45，优于基线（最高 0.19）。
- **下游**：模型无关在线 RL、基于模型在线 RL、离线 RL、失败检测、IL 数据检索均受益；6 个机构外 OOD 场景验证泛化。
- 混淆矩阵显示 Robometer 在未见数据上奖励—指令对齐最对角化。

<a id="sec-5"></a>

## 5. 在 A-D6 中的角色

- 提供"**进度 + 偏好**"的通用价值/奖励信号，是 ProgressVLA 进度估计的规模化扩展（1M 数据 + 偏好学习）。
- 可直接做**失败检测**（逐帧 progress 异常低 → 失败）与**数据检索排序**（按奖励筛选/检索训练演示）。
- 与 A-D4 影响函数类方法（CUPID/QoQ）互补：Robometer 是模型无关的密集奖励，影响函数衡量对特定策略回报的因果影响。

<a id="sec-6"></a>

## 6. 局限

- [OPEN] 依赖预训练 VLM 骨干与大规模数据，部署成本不低。
- 偏好标签由轨迹对自动构造，可能含噪声（尤其增强生成的失败是否真实合理）。
- 奖励曲线对长时程、非单调失败的解释仍需人工判读，未给失败定位的时间窗。
- 主要验证在操纵任务，长时程/接触丰富场景的奖励可靠性待查。

<a id="sec-7"></a>

## 7. 与其他方法的关系

- **ProgressVLA（A-D5）**：同属进度估计脉络；Robometer 更通用、规模更大、加偏好学习。
- **CUPID / QoQ / ATHENA（A-D4）**：影响函数衡量任务价值；Robometer 提供密集奖励，可作其上游信号。
- **Foresight（A-D6）**：在线失败监测；Robometer 提供离线奖励/排序，二者可组合（Robometer 打分 + Foresight 时变阈值监测）。

<a id="sec-8"></a>

## 8. 对当前项目的启示

- 当前遥操质检若仅有成功示范、缺失败标签，可借鉴 Robometer 的**偏好比较**思路：用"成功 vs 已知次优/增强失败"的成对比较构建训练信号，无需逐帧失败标注。
- 其逐帧 progress 曲线可直接作为 episode 级质量分 $q(\tau)$ 的连续版本，与 DQAF 的遥测聚合互补。

<a id="sec-9"></a>

## 9. 最终摘要

Robometer 用帧级进度 + 轨迹比较偏好双目标训练通用机器人奖励模型，借助 1M 多具身轨迹（含大量失败/次优）学习到跨轨迹全局排序，在奖励对齐与轨迹排序上大幅超越基线，并赋能 RL、失败检测与数据检索。它是 A-D6 中"进度/偏好价值信号"的代表方法，对当前项目的失败检测与数据排序有直接借鉴价值。

<a id="appendix-a"></a>

## 附录 A：术语

- **Reward Model (RM)**：奖励模型，输出对轨迹/状态的标量评价。
- **Preference learning**：偏好学习，从"轨迹 A 优于 B"的成对比较中学习。
- **VOC (Value Order Correlation)**：奖励与真实进度的 Pearson 相关性。
- **Kendall τ**：轨迹排序的 Kendall 等级相关系数。
- **RBM-1M**：Robometer 的百万级奖励学习数据集。
