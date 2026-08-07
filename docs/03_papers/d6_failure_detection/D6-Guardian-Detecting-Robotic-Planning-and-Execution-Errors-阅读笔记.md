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
migrate_source: "VLA/数据质量评估/notes/D6-Guardian-Detecting-Robotic-Planning-and-Execution-Errors-阅读笔记.md"
title: "Guardian: Detecting Robotic Planning and Execution Errors with Vision-Language Models"
short_name: "Guardian"
authors: "Paul Pacaud, Ricardo Garcia, Shizhe Chen, Cordelia Schmid"
venue: "arXiv preprint (submitted to IEEE)"
year: 2025
paper_link: "https://arxiv.org/abs/2512.01946"
code_link: "https://www.di.ens.fr/willow/research/guardian/"
project_link: "https://www.di.ens.fr/willow/research/guardian/"
local_file: "Guardian-Detecting-Robotic-Planning-and-Execution-Errors-with-Vision-Language-Models.pdf"
pdf_index: "VLA/数据质量评估/papers/Guardian-Detecting-Robotic-Planning-and-Execution-Errors-with-Vision-Language-Models.pdf"
paper_version: "arXiv:2512.01946v3, 2026-03-30"
read_date: 2026-08-03
read_status: "L1-定向精读完成"
primary_dimension: "A-直接质量评估与数据筛选"
related_dimensions: ["A-D6", "B", "A-D5", "外部验证"]
primary_level: "episode / 多视角帧 / 子任务描述"
keywords:
  - Guardian
  - failure detection
  - vision-language model
  - failure synthesis
  - planning error
  - execution error
  - UR5-Fail
  - RoboFail
  - chain-of-thought
---

> **迁移与核验说明（2026-08-03）：** 本文档由用户原始阅读笔记迁移至知识库，内容为用户本人笔记，未作改写。论文版本 / arXiv / 官方代码 / 项目主页已在原笔记 YAML 中记录，并于 2026-08-03 联网核验（其中 4 篇 2025-12 / 2026 arXiv 编号均确认可访问）。PDF 仅作索引、不纳入知识库 Git（遵循默认边界：大型文件只记录索引，不直接纳入 Git），本地路径见同目录 `INDEX.md`。原笔记的证据强度标签 `[PAPER]`（论文陈述）/ `[RESULT]`（实验结果）/ `[INFERENCE]`（面向当前项目分析）/ `[OPEN]`（未公开或未验证）予以保留。


# Guardian: Detecting Robotic Planning and Execution Errors with Vision-Language Models 阅读笔记

> **本文档定位：**  
> 这是一份以"用视觉—语言模型做机器人规划/执行失败检测与原因推理"为主线的定向精读笔记。Guardian 的核心贡献有两点：(1) 提出**程序化失败合成**管线，对成功轨迹做扰动自动生成带细粒度类别与推理链的失败样本；(2) 基于合成数据微调 InternVL3-8B 得到 Guardian，用多视角图像推理规划/执行错误。对当前项目（实机遥操质量评估）最有价值的是：其**三级失败分类 schema**、**失败数据可控制合成**范式，以及"多视角 + 子任务描述 → VLM 推理失败"的接口形态。

> **论文定位：**  
> Guardian 落在 A-D6（失败检测与失败数据分析）。它与 AHA 同属"失败数据合成 + VLM 失败推理"脉络，但 Guardian 偏**细粒度分类 + 显式 CoT 推理链**，AHA 偏自由文本解释；二者失败类别体系可互引。Guardian 同时发布真实双臂策略 rollout 自动标注的 UR5-Fail，比单视角遥操作标注的 RoboFail 更贴近真实失败分布。

> **证据说明：**  
> `[PAPER]` 表示论文直接陈述；`[RESULT]` 表示论文实验或图表结果；`[INFERENCE]` 表示根据论文方法面向当前项目作出的分析；`[OPEN]` 表示论文没有公开或尚未验证的问题。页码对应 arXiv v3（2026-03-30）。

<a id="目录"></a>

## 目录

- [0. 阅读结论速览](#sec-0)
- [1. 论文定位、研究问题与证据边界](#sec-1)
  - [1.1 核心问题](#sec-1-1)
  - [1.2 为什么值得放入失败检测调研](#sec-1-2)
  - [1.3 论文没有提供什么](#sec-1-3)
- [2. 方法一：程序化失败合成管线](#sec-2)
  - [2.1 成功轨迹扰动](#sec-2-1)
  - [2.2 三类失败基准构造](#sec-2-2)
  - [2.3 细粒度类别与推理链](#sec-2-3)
- [3. 方法二：Guardian 模型](#sec-3)
  - [3.1 问题形式化](#sec-3-1)
  - [3.2 多视角输入与 CoT](#sec-3-2)
- [4. 实验](#sec-4)
  - [4.1 基准与基线](#sec-4-1)
  - [4.2 主要结果](#sec-4-2)
  - [4.3 下游集成](#sec-4-3)
- [5. 在 A-D6 中的角色](#sec-5)
- [6. 局限与开放问题](#sec-6)
- [7. 与其他失败检测方法的关系](#sec-7)
- [8. 对当前项目的启示](#sec-8)
- [9. 最终摘要](#sec-9)
- [附录 A：专业术语与缩写](#appendix-a)

<a id="sec-0"></a>

## 0. 阅读结论速览

- Guardian 把机器人失败检测形式化为 **VQA 任务**：输入多视角图像 + 任务指令 + 子任务描述，输出失败二元判定 + 细粒度失败类别 + 逐步推理链。
- 关键创新是**失败数据可控制合成**——对成功演示做程序化扰动，生成规划/执行失败样本及推理链，突破了真实失败数据稀缺的瓶颈。
- 发布 **RLBench-Fail / BridgeDataV2-Fail / UR5-Fail** 三个基准，其中 UR5-Fail 来自真实双臂策略 rollout，比单视角遥操作标注更真实。
- 在四个基准（含 ID/OOD）达到 SOTA；集成进 3D-LOTUS++ 后提升任务成功率。
- 对当前项目：其失败分类 schema（规划 5 类 / 执行若干类）可直接作为失败标注体系；合成管线可迁移到"用成功演示生成失败样本做质检模型训练"。

<a id="sec-1"></a>

## 1. 论文定位、研究问题与证据边界

<a id="sec-1-1"></a>

### 1.1 核心问题

稳健的机器人操纵需要可靠的失败检测与恢复。VLM 虽展现潜力，但其准确率与泛化受限于**失败数据稀缺**。[PAPER] 现有失败数据集要么规模小、要么仅仿真、要么只覆盖执行错误、且缺细粒度失败原因与推理链。

<a id="sec-1-2"></a>

### 1.2 为什么值得放入失败检测调研

- 它给出了一套**可复用的失败类别体系**与**失败数据合成方法**，正是 A-D6 需要的"失败 = 低质量"判定所需的基础设施。
- 其多视角 + 子任务描述的输入形态，与地面遥操采集的多相机 + 子任务计划数据高度同构，可直接借鉴用于 episode 级失败标注。

<a id="sec-1-3"></a>

### 1.3 论文没有提供什么

- 未给出失败检测的**误报成本 / 阈值**分析（在遥操质检中误报会打断操作者，成本不低）。
- 合成失败的**分布偏移**未充分量化——扰动生成的失败是否覆盖真实长尾失败未知。

<a id="sec-2"></a>

## 2. 方法一：程序化失败合成管线

<a id="sec-2-1"></a>

### 2.1 成功轨迹扰动

[PAPER] 对成功轨迹做程序化扰动，自动生成多样化规划/执行失败。与 AHA 的 FailGen 同源思路，但 Guardian 强调同时产出**二元标签 + 细粒度失败类别 + 逐步推理链**（仿真与真实均适用）。

<a id="sec-2-2"></a>

### 2.2 三类失败基准构造

[PAPER] 由此构造三个新基准：

| 基准 | 规模（执行 / 规划） | 来源 |
|---|---|---|
| RLBench-Fail | 14K / 7K | RLBench 仿真 |
| BridgeDataV2-Fail | 10K / 6K | BridgeDataV2 真实数据（policy-driven） |
| UR5-Fail | 570 / 370 | 真实 UR5 双臂 + 三相机，3D-LOTUS++ 策略 rollout |

UR5-Fail 用 34 个任务、三视角图像，由自主策略 rollout 产生更真实的失败（区别于 RoboFail 的单视角遥操作）。[INFERENCE] 这与当前项目"真实双臂遥操失败难采集"的痛点直接相关——用策略 rollout 自动标注是低成本获取真实失败标签的可行路径。

<a id="sec-2-3"></a>

### 2.3 细粒度类别与推理链

[PAPER] 规划失败分 5 类（如步骤遗漏、错选物体、位置错误等），执行失败若干类（抓取错误、时序错误等）。每条样本附平均 118 token 的推理链，由大推理模型（InternVL3-38B）基于初始图文 + 失败原因生成。

<a id="sec-3"></a>

## 3. 方法二：Guardian 模型

<a id="sec-3-1"></a>

### 3.1 问题形式化

[PAPER] 规划验证：给定任务指令 T、计划 P 与初始视觉上下文 I_start，VLM 输出 (B_plan, C_plan)——是否有效 + 失败类别。执行验证：对每子步前后图像判断完成与否并归类。

<a id="sec-3-2"></a>

### 3.2 多视角输入与 CoT

[PAPER] Guardian 微调自 InternVL3-8B，融合高分辨率多视角图像，通过显式 CoT 推理多类失败。相比 AHA（拼接多视角为单图压缩输入）、Sentinel/Cosmos-Reason1（单视角或仅 CoT），Guardian 统一了规划与执行的验证，且对分离的多视角输入做推理。

<a id="sec-4"></a>

## 4. 实验

<a id="sec-4-1"></a>

### 4.1 基准与基线

[PAPER] 对比通用大模型（Qwen3-VL-235B、GPT4.1）与专用小模型（CLIP+MLP、Sentinel、Cosmos-Reason1、AHA-13B、InternVL3-8B）。

<a id="sec-4-2"></a>

### 4.2 主要结果

[RESULT] 在 ID 与 OOD 四个基准上，Guardian-8B-Thinking 在规划/执行二元准确率上多数超越或匹配更大的 Qwen3-VL / GPT4.1；例如 RLBench-Fail 执行 0.83 / 规划 0.87，UR5-Fail 执行 0.86 / 规划 0.70。比从 InternVL3-8B 基座提升约 23%。

<a id="sec-4-3"></a>

### 4.3 下游集成

[RESULT] 作为即插即用验证模块集成进 3D-LOTUS++ 操纵系统，在仿真与真实机器人任务中提升任务完成率。

<a id="sec-5"></a>

## 5. 在 A-D6 中的角色

- 提供"**失败分类 + 原因解释**"的可复用监督信号与失败数据合成范式。
- 其失败类别体系可作失败标注 schema；合成管线可作"用成功演示生成失败样本"的模板。
- 与 AHA 互补：AHA 自由文本解释，Guardian 细粒度分类 + CoT；二者可互为训练数据增强来源。

<a id="sec-6"></a>

## 6. 局限与开放问题

- [OPEN] 合成失败分布与真实长尾失败的偏移未量化。
- 推理链由大模型生成，需人工校验质量与偏差。
- UR5-Fail 规模有限（测试各 140），真实泛化证据仍偏弱。
- 误报 / 漏报成本未讨论——在采集期质检中这直接决定可用性。

<a id="sec-7"></a>

## 7. 与其他失败检测方法的关系

- **AHA（ICLR 2025）**：同源失败合成 + VLM 路线；AHA 自由文本、Guardian 细粒度分类 + CoT。
- **RoboFAC**：提供三级失败分类 schema 与视频—运动学基准，是 Guardian 的评测落脚点之一。
- **KITE**：训练无关前端，把视频转成结构化证据 token 再喂 VLM；可与 Guardian 后端解耦组合。
- **Foresight / REFLECT**：在线监测 / 多模态摘要路线，与 Guardian 的离线分类互补。

<a id="sec-8"></a>

## 8. 对当前项目的启示

- 失败标注不必纯人工：可用"成功演示 + 程序化扰动"合成失败样本，再用真实 rollout 校准，降低标注成本。
- 多相机 + 子任务计划 + 语言指令的输入形态，与当前遥操采集同构，Guardian 的接口设计可直接复用。
- 建议把 Guardian 的规划/执行二级分类纳入 episode 级质检的失败标签体系。

<a id="sec-9"></a>

## 9. 最终摘要

Guardian 用程序化扰动成功轨迹合成带细粒度类别与推理链的失败数据，并据此微调多视角 VLM 实现规划/执行错误检测与原因推理，在多个基准达 SOTA。它是 A-D6 中"失败数据合成 + 失败分类"的代表方法，对当前项目的失败标注体系与低成本失败数据获取有直接借鉴价值。

<a id="appendix-a"></a>

## 附录 A：专业术语与缩写

- **VLM (Vision-Language Model)**：视觉—语言模型。
- **CoT (Chain-of-Thought)**：思维链，逐步推理输出。
- **UR5-Fail / RoboFail**：真实机器人失败检测基准。
- **3D-LOTUS++**：论文使用的视觉—语言操纵策略。
- **OOD (Out-of-Distribution)**：分布外，指测试集含未见任务/环境。
