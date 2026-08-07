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
migrate_source: "VLA/数据质量评估/notes/D6-RoboFAC-A-Comprehensive-Framework-for-Robotic-Failure-Analysis-and-Correction-阅读笔记.md"
title: "RoboFAC: A Comprehensive Framework for Robotic Failure Analysis and Correction"
short_name: "RoboFAC"
authors: "Weifeng Lu, Minghao Ye, Zewei Ye, Ruihan Tao, Shuo Yang, Bo Zhao (SJTU MINT)"
venue: "arXiv preprint"
year: 2025
paper_link: "https://arxiv.org/abs/2505.12224"
code_link: "https://github.com/MINT-SJTU/RoboFAC"
project_link: "https://mint-sjtu.github.io/RoboFAC.io/"
local_file: "RoboFAC-A-Comprehensive-Framework-for-Robotic-Failure-Analysis-and-Correction.pdf"
pdf_index: "VLA/数据质量评估/papers/RoboFAC-A-Comprehensive-Framework-for-Robotic-Failure-Analysis-and-Correction.pdf"
paper_version: "arXiv:2505.12224v4, 2026-03-22"
read_date: 2026-08-03
read_status: "L1-定向精读完成"
primary_dimension: "A-直接质量评估与数据筛选"
related_dimensions: ["A-D6", "B", "A-D5"]
primary_level: "视频 / 失败轨迹 / 8 类 QA"
keywords:
  - RoboFAC
  - failure analysis
  - failure correction
  - failure taxonomy
  - video QA
  - benchmark
  - VLA critic
---

> **迁移与核验说明（2026-08-03）：** 本文档由用户原始阅读笔记迁移至知识库，内容为用户本人笔记，未作改写。论文版本 / arXiv / 官方代码 / 项目主页已在原笔记 YAML 中记录，并于 2026-08-03 联网核验（其中 4 篇 2025-12 / 2026 arXiv 编号均确认可访问）。PDF 仅作索引、不纳入知识库 Git（遵循默认边界：大型文件只记录索引，不直接纳入 Git），本地路径见同目录 `INDEX.md`。原笔记的证据强度标签 `[PAPER]`（论文陈述）/ `[RESULT]`（实验结果）/ `[INFERENCE]`（面向当前项目分析）/ `[OPEN]`（未公开或未验证）予以保留。


# RoboFAC: A Comprehensive Framework for Robotic Failure Analysis and Correction 阅读笔记

> **本文档定位：**  
> 这是一份以"构建大规模机器人失败分析基准 + 微调失败分析/纠正模型"为主线的定向精读笔记。RoboFAC 的核心贡献有三点：(1) **三级失败分类法**（Task/Motion/Execution Planning）；(2) 把失败视频标注为 **8 类问答**（检测/识别/定位/解释/纠正等），并记录**视频—运动学配对**；(3) 微调 Qwen2.5-VL 作外部 critic 接入 VLA 控制管线。对当前项目（遥操质量评估）最有价值的是其**失败分类 schema** 与**视频—运动学配对基准**——可直接作为失败标注体系与轨迹级质量诊断的参考。

> **论文定位：**  
> RoboFAC 落在 A-D6（失败检测与失败数据分析），跨 B（失败数据/基准）。它是 Guardian/AHA/KITE 失败分析的**评测落脚点之一**——提供细粒度标注与分类 schema；其运动学配对对轨迹级质量诊断有额外价值。

> **证据说明：**  
> `[PAPER]` 论文直接陈述；`[RESULT]` 实验结果；`[INFERENCE]` 面向当前项目分析；`[OPEN]` 未公开/未验证。页码对应 arXiv v4（2026-03-22）。

<a id="目录"></a>

## 目录

- [0. 阅读结论速览](#sec-0)
- [1. 论文定位与研究问题](#sec-1)
- [2. 失败分类法](#sec-2)
- [3. 数据集构造](#sec-3)
  - [3.1 仿真与真实采集](#sec-3-1)
  - [3.2 八类 QA 标注](#sec-3-2)
- [4. RoboFAC 模型与部署](#sec-4)
- [5. 实验](#sec-5)
- [6. 在 A-D6 中的角色](#sec-6)
- [7. 局限](#sec-7)
- [8. 与其他方法的关系](#sec-8)
- [9. 对当前项目的启示](#sec-9)
- [10. 最终摘要](#sec-10)
- [附录 A：术语](#appendix-a)

<a id="sec-0"></a>

## 0. 阅读结论速览

- RoboFAC 提出**三级失败分类**：Task Planning / Motion Planning / Execution Control，各含具体错误类型（如步骤遗漏、错物、位姿偏差、抓取错误、时序错误）。
- 构建 **RoboFAC 数据集**：9,440 失败 + 1,282 成功轨迹、78,623 QA、16 任务、53 场景（仿真 + 真实 SO-100 遥操作）。
- 把失败视频标注为 **8 类问答**：任务识别、任务规划、失败检测、失败识别、失败定位、失败解释、高层纠正、低层纠正。
- 微调 Qwen2.5-VL 得到 RoboFAC 模型，在基准上超 GPT-4o 34.1%；接入真实 VLA 管线作外部 critic，4 真实任务平均相对提升 29.1%。

<a id="sec-1"></a>

## 1. 论文定位与研究问题

[PAPER] 现有 VLA 多在成功演示上训练、缺失败恢复能力；缺覆盖广、标注细、含真实数据的失败分析基准。RoboFAC 旨在补全这一缺口。

<a id="sec-2"></a>

## 2. 失败分类法

[PAPER] 受经典机器人文献启发，按任务层次结构定义三级失败：
- **Task Planning Error**：任务分解或语言 grounding 错误（步骤遗漏、错选物体）。
- **Motion Planning Error**：空间推理/指令到位姿映射错误（位置偏差、朝向偏差）。
- **Execution Control Error**：执行中的物理不精确/延迟/动态失配（抓取错误、时序错误）。

每级错误原子化、任务无关，可在操作中一致观测。

<a id="sec-3"></a>

## 3. 数据集构造

<a id="sec-3-1"></a>

### 3.1 仿真与真实采集

[PAPER] 仿真用 MiniSkill + YCB 物体 + ReplicaCAD/AI2-THOR 场景，定义专家策略后替换子阶段为错误代码生成失败；真实用 SO-100 遥操作采集 6 任务（含 2 个仿真没有的）。记录视频 + 文本描述（失败子阶段、分类、详细解释、扰动位姿）。

<a id="sec-3-2"></a>

### 3.2 八类 QA 标注

[PAPER] 标注为 8 类视频问答：任务识别、任务规划、失败检测、失败识别、失败定位、失败解释、高层纠正、低层纠正。前 5 类真值可直接抽取；后 3 类用 GPT-4o 生成 + 人工校验。最终 70K 仿真 + 8K 真实 QA。

<a id="sec-4"></a>

## 4. RoboFAC 模型与部署

[PAPER] 微调 Qwen2.5-VL 得到 RoboFAC 模型，给定失败视频输出任务理解 + 失败分析 + 失败纠正。部署为真实 VLA 控制管线的**外部 critic**，在线检测并给纠正指令。

<a id="sec-5"></a>

## 5. 实验

[RESULT]
- 数据集：9,440 失败 + 1,282 成功轨迹、78,623 QA、16 任务、53 场景。
- RoboFAC 模型在评测基准上超 GPT-4o 34.1%。
- 接入真实 VLA 管线作 critic，4 真实任务平均相对提升 29.1%。

<a id="sec-6"></a>

## 6. 在 A-D6 中的角色

- 提供**失败分类 schema + 视频—运动学配对基准 + 失败定位任务**的完整工件。
- 是 KITE/Guardian/AHA 失败分析的评测落脚点之一；其运动学配对（.h5  kinematics）对轨迹级质量诊断有额外价值（可比对失败轨迹的关节/夹爪状态）。

<a id="sec-7"></a>

## 7. 局限

- [OPEN] 仿真失败由代码扰动生成，分布偏理想，真实长尾失败覆盖有限。
- 真实仅 6 任务、480 失败轨迹，规模偏小。
- QA 参考答案部分由 GPT-4o 生成，需人工把关质量与偏差。
- 未给失败检测在真实部署中的实时延迟与误报成本。

<a id="sec-8"></a>

## 8. 与其他方法的关系

- **Guardian / AHA（A-D6）**：端到端 VLM 失败模型；RoboFAC 提供它们评测所需的细粒度基准与分类。
- **KITE（A-D6）**：在 RoboFAC 基准上验证其 training-free 前端。
- **REFLECT（A-D6）**：多模态摘要→LLM 解释；RoboFAC 提供结构化纠正 QA。

<a id="sec-9"></a>

## 9. 对当前项目的启示

- 失败标注体系可直接借用 RoboFAC 的三级分类（Task/Motion/Execution Planning），比 Guardian 的规划/执行二级更细。
- 视频—运动学配对思路值得借鉴：失败轨迹不仅要标视觉，还应配对关节/夹爪状态，便于轨迹级质量诊断。

<a id="sec-10"></a>

## 10. 最终摘要

RoboFAC 构建了大规模、细标注的机器人失败分析基准（三级分类 + 8 类 QA + 视频—运动学配对），并微调模型作 VLA 外部 critic 实现失败纠正，显著提升真实任务表现。它是 A-D6 中"失败分析基准与分类 schema"的代表方法，对当前项目的失败标注体系有直接借鉴价值。

<a id="appendix-a"></a>

## 附录 A：术语

- **RoboFAC dataset**：9,440 失败 + 1,282 成功轨迹、78,623 QA 的失败分析基准。
- **Failure taxonomy**：三级失败分类（Task/Motion/Execution Planning）。
- **8-type QA**：任务识别/规划、失败检测/识别/定位/解释、高层/低层纠正。
- **External critic**：外部 critic，接入 VLA 管线作在线失败检测与纠正。
- **MiniSkill / SO-100**：仿真环境与真实遥操作臂。
