---
title: 迁移清单（Phase 1 详细编目）
type: project
status: active
created: 2026-07-31
updated: 2026-07-31
tags:
  - project-maintenance
  - migration
  - checklist
---

# 迁移清单（Phase 1 详细编目）

> 本文件是「第一阶段：建立迁移清单」的产出，与 [`MIGRATION_BACKLOG.md`](./MIGRATION_BACKLOG.md)（优先级路线图）和 [`RESEARCH_MAP.md`](./RESEARCH_MAP.md)（研究地图）联动。
>
> **本阶段原则**：先不移动或删除原资料，仅为每个候选材料建立记录。处理方式分为：迁移 / 改写 / 仅建索引 / 暂缓 / 排除。

## 0. 字段说明

| 字段 | 含义 |
|---|---|
| 原始路径 | `VLA/` 父文件夹下的源位置（不含 `my-embodied-ai-kb`） |
| 资料类型 | 纯论文 PDF / 笔记目录 / 混合（PDF+代码+权重）/ 代码仓库 / 非 md（docx/pptx） |
| 研究主题 | 归属的研究方向 |
| 是否已有总结 | 源内是否已有中文/英文 Markdown 分析笔记 |
| 是否需要联网核验 | 论文版本、项目主页、官方代码、实验数字、版权状态是否需联网确认 |
| 目标目录 | 计划落入的 `docs/` 子目录 |
| 处理方式 | 迁移 / 改写 / 仅建索引 / 暂缓 / 排除 |
| 来源和版权风险 | 论文 PDF 版权、代码许可证、公司材料、大文件等风险标记 |

## 1. 排除规则（本阶段即生效）

以下类别**不纳入知识库**，仅在清单中标记：

1. `.git`、`.codex`、`.agents`、`.vscode` 等版本/工具元数据目录；
2. `.pytest_cache`、`.ruff_cache`、`tmp`、`__pycache__`、`__MACOSX__` 等临时/缓存目录；
3. 数据集、模型权重（`.pt`/`.ckpt`/`.safetensors`/`.bin`）、训练与评测输出（`runs/`、`eval_logs/`）；
4. 大型 PDF（>10 MB）、压缩包（`.zip`）、书籍；
5. 与研究系统无关的公司材料。

**已从 `VLA/` 父文件夹识别的具体排除项**：

- 压缩包（约 950 MB 总计）：`数据质量评估/grasp_2_20260704.zip`(303M)、`RL/ray-master.zip`(176M)、`Diffusion_related/BC_and_RL/robust-rearrangement-main.zip`(173M)、`RL/spinningup-simplified-main.zip`(95M)、`数据质量评估/episode级数据质量评估规划.zip`(63M)、`Diffusion_related/Residual_Learning/vq_bet_official-main.zip`(31M) 等；
- 模型权重：`VLA_Adapter/VLA-Adapter-main/pretrained_models/prism-qwen25-...`(12M)；
- 训练/评测输出：`RL/spinningup-simplified-main/runs/`、`RL/PPO/PPO_Implementation_Details/runs/`、`VLA_Adapter/VLA-Adapter-main/eval_logs/`(19M)；
- 缓存：`数据质量评估/.pytest_cache`、`.ruff_cache`、`tmp/`(2.4G)、15 处 `__pycache__`、`Diffusion_related/BC_and_RL/__MACOSX__`；
- 大型 PDF（>10MB，共 29 个）：`Robot_Relate_Book/*机器人学*.pdf`(132M)、`ROS机械臂开发与实践.pdf`(90M)、`未分类/Track2Act.pdf`(57M)、`lingbot/LingBot_World.pdf`(48M)、`RL_and_VLA/SLIM.pdf`(39M)、`openVLA-oft/2502.19645v2.pdf`(27M)、`AR_VLA/2603.10126v2.pdf`(21M) 等；
- 公司材料：`公司任务_介绍PPT_12月中旬`(3.1M)；
- 残留/空目录：`.codewhale`、`978-3-031-73116-7 (1).pdf.crdownload`、`Embodied_AI_Research_Map`（空）、`新建文件夹`（空）。

---

## 2. Phase 2 已迁移（记录留存）

> 试点已完成，以下条目状态为「已完成」。源材料保留在 `VLA/数据质量评估/`，知识库内以相对链接引用其代码与数据。

| 原始路径 | 资料类型 | 研究主题 | 是否已有总结 | 目标目录 | 处理方式 | 来源和版权风险 |
|---|---|---|---|---|---|---|
| `VLA/数据质量评估/docs/具身数据集质量评估-调研地图.md` | 笔记目录 | 实机数据质量评估 | 是 | `docs/02_topics/embodied_data_quality/` | 迁移（改写对齐模板） | 自用笔记，低风险 |
| `VLA/数据质量评估/docs/L1核心精读总结.md` 等 4 篇支撑 | 笔记 | 数据质量阅读 | 是 | `docs/02_topics/embodied_data_quality/` | 迁移 | 低风险 |
| `VLA/数据质量评估/docs/Data Quality in Imitation Learning-阅读笔记.md` | 笔记+PDF | 数据质量论文 | 是 | `docs/03_papers/data_quality_in_imitation_learning/` | 迁移（PDF 走 LFS） | 论文版权，PDF 仅本地 LFS |
| `VLA/数据质量评估/docs/阅读笔记-DataAssessmentforEmbodiedIntelligence.md` | 笔记+PDF | 数据质量论文 | 是 | `docs/03_papers/data_assessment_for_embodied_intelligence/` | 迁移（PDF 走 LFS） | 论文版权 |
| `VLA/数据质量评估/notes/RobotDataCuration..._reading.md` | 笔记+PDF | 互信息数据筛选 | 是 | `docs/03_papers/robot_data_curation_with_mutual_information_estimators/` | 迁移（PDF 走 LFS） | 论文版权 |
| `VLA/数据质量评估/notes/ASurveyofDatasets_reading.md` | 笔记+PDF | VLA 数据综述 | 是 | `docs/03_papers/a_survey_of_datasets/` | 迁移（PDF 走 LFS） | 论文版权 |
| `VLA/数据质量评估/PAPER_MATRIX.md`、`READING_LOG.md` | 笔记 | 论文矩阵/阅读日志 | 是 | `docs/02_topics/embodied_data_quality/` | 迁移 | 低风险 |
| `VLA/数据质量评估/report/phase1/*.pptx`（4 份） | 非 md（PPTX） | 阶段汇报 | 是（汇报） | `docs/06_reports/数据质量评估进度汇报/` | 仅建索引（PPTX 走 LFS，不作为核心知识源） | 内部汇报，注意不外传 |

---

## 3. Phase 3 候选：强化学习与 VLA 后训练

重点来源：`RL/`、`RL_and_VLA/`、`RIPT-VLA/`、`SimpleVLA_RL/`、`Diffusion_related/`、`ACT/`、`action_chunk/`。
目标产出：强化学习基础、PPO/价值函数/信用分配、VLA 后训练专题、动作块与流匹配动作生成、可复现实验记录。

| 原始路径 | 资料类型 | 研究主题 | 是否已有总结 | 需联网核验 | 目标目录 | 处理方式 | 来源和版权风险 |
|---|---|---|---|---|---|---|---|
| `VLA/RL/` | 混合（PDF+代码包+zip+runs） | RL 基础 / PPO / GRPO / SAC | 否（无 md） | 是（PPO/GRPO 论文版本） | `docs/01_foundations/`（RL 基础、PPO、价值函数、信用分配）+ `docs/04_experiments/`（可复现实验） | 提炼（建笔记，不复制代码）；排除 `*.zip`、`runs/`、`__pycache__` | 论文 PDF 版权；spinningup MIT / ray Apache，代码仅关联引用 |
| `VLA/RL_and_VLA/` | 纯论文（5 PDF） | VLA 后训练（RL 用于 VLA） | 否 | 是 | `docs/03_papers/`（每篇一笔记）+ `docs/02_topics/`（VLA 后训练专题） | 仅建索引（PDF）；后续建笔记 | 论文版权；`SLIM.pdf`(39M) 仅索引 |
| `VLA/RIPT-VLA/` | 笔记目录（4 md + PDF） | VLA 交互式后训练 / GRPO vs RLOO / PPO ε | 是（4 篇中文分析） | 是（论文版本、官方代码、实验数字） | `docs/03_papers/ript_vla/` + `docs/02_topics/` | 改写（对齐模板，补核验） | 论文版权；代码许可待查 |
| `VLA/SimpleVLA_RL/` | 纯论文（1 PDF） | VLA 后训练 | 否 | 是 | `docs/03_papers/simplevla_rl/` | 仅建索引；后续建笔记 | 论文版权 |
| `VLA/Diffusion_related/` | 混合（9 子目录 PDF+代码 zip） | 流匹配、扩散策略、DPPO/DDPO | 否（仅翻译版 PDF） | 是 | `docs/01_foundations/`（流匹配动作生成）+ `docs/03_papers/`（DPPO 等）+ `docs/02_topics/` | 提炼（PDF 结论建笔记）；排除 `*.zip`、`__MACOSX__`、`CoRL_supplemental.mp4`、大 PDF | 论文版权；代码许可待查 |
| `VLA/ACT/` | 非 md（docx + pptx）+ PDF | 动作块 / ACT | 部分（docx 梳理） | 是 | `docs/02_topics/`（动作块与流匹配）+ `docs/03_papers/act/` | 改写（docx→md 笔记）；PPTX 仅建索引 | 论文版权 |
| `VLA/action_chunk/` | 纯论文（1 PDF） | 动作块 | 否 | 是 | `docs/03_papers/action_chunk/` + `docs/02_topics/` | 仅建索引；后续建笔记 | 论文版权 |

---

## 4. Phase 4 候选：建立 VLA 论文库

按「一篇论文一个笔记」迁移。现有 AR-VLA、RIPT-VLA、X-VLA、LingBot 分析可作初始材料，但需重检论文版本 / 主页 / 官方代码 / 实验数字 / 结论边界。

| 原始路径 | 资料类型 | 研究主题 | 是否已有总结 | 需联网核验 | 目标目录 | 处理方式 | 来源和版权风险 |
|---|---|---|---|---|---|---|---|
| `VLA/openvla/` | 纯论文（PDF） | VLA 基础模型 | 否（有 lerobot 底稿） | 是 | `docs/03_papers/openvla/` | 迁移（建笔记，可复用 lerobot 分析） | 论文版权 |
| `VLA/openVLA-oft/` | 纯论文（PDF, 27M） | VLA 微调 | 否 | 是 | `docs/03_papers/openvla_oft/` | 迁移；PDF 仅建索引（27M） | 论文版权 |
| `VLA/pi0/` | 纯论文（PDF） | VLA（π0） | 否（有 lerobot 底稿） | 是 | `docs/03_papers/pi0/` | 迁移（建笔记） | 论文版权 |
| `VLA/pi05/` | 纯论文（PDF） | VLA（π0.5） | 否 | 是 | `docs/03_papers/pi05/` | 迁移（建笔记） | 论文版权 |
| `VLA/pio_fast/` | 纯论文（PDF） | VLA 动作表示（π0-FAST） | 否 | 是 | `docs/03_papers/pi0_fast/` | 迁移（建笔记） | 论文版权 |
| `VLA/RT-1/` | 纯论文（PDF） | VLA 先驱 | 否 | 是 | `docs/03_papers/rt1/` | 迁移（建笔记） | 论文版权 |
| `VLA/SmolVLA/` | 纯论文（PDF） | VLA（SmolVLA） | 否（有 lerobot 底稿） | 是 | `docs/03_papers/smolvla/` | 迁移（建笔记） | 论文版权 |
| `VLA/x-vla/` | 笔记目录（md + PDF） | 跨具身 VLA（X-VLA） | 是（`X-VLA_论文深度分析.md`） | 是 | `docs/03_papers/x_vla/` + `docs/02_topics/`（跨具身） | 改写（对齐模板） | 论文版权；同时是跨具身主题核心笔记 |
| `VLA/AR_VLA/` | 混合（2 md + PDF + 代码子仓） | 自回归 VLA（AR-VLA） | 是（2 篇中文） | 是（论文版本、官方代码、实验数字） | `docs/03_papers/ar_vla/` + `docs/02_topics/` | 改写（2 md 迁移，PDF 走 LFS）；排除 `AR-VLA-lerobot/.git` | 论文版权；代码 INSAIT 许可待查 |
| `VLA/VLA_Adapter/` | 混合（含权重/日志/zip） | VLA 适配器 | 否 | 是 | `docs/03_papers/vla_adapter/` | 仅建索引（PDF + README）；排除 权重(12M)/`*.zip`/`eval_logs/`/`__pycache__` | 论文版权；权重/代码许可待查 |
| `VLA/TriVLA/` | 纯论文（PDF） | VLA（TriVLA） | 否 | 是 | `docs/03_papers/trivla/` | 迁移（建笔记） | 论文版权 |
| `VLA/UD-VLA/` | 纯论文（PDF） | VLA（UD-VLA） | 否 | 是 | `docs/03_papers/ud_vla/` | 迁移（建笔记） | 论文版权 |
| `VLA/lingbot/` | 混合（7 md 分析 + PDF + 代码子仓） | LingBot 系列（VLA/World/VA/Depth/Map/Qwen-VLA） | 是（7 篇） | 是 | `docs/03_papers/lingbot_*/` + `docs/02_topics/`（世界模型/跨具身） | 改写（7 md 分流到 Phase 4 与 Phase 5）；排除 `lingbot-va/.git`、`LingBot_World.pdf`(48M) 仅索引 | 论文版权；代码许可待查 |
| `VLA/lerobot/docs/policy_analysis/`（23 篇 md） | 笔记（代码级分析） | 多模型策略分析（覆盖 ACT/pi0/SmolVLA/openvla/x-vla 等） | 是 | 部分 | `docs/02_topics/`（索引）或作为上述笔记补充底稿 | 改写/仅建索引（交叉引用，不重复建笔记） | lerobot Apache-2.0；自用笔记低风险 |

---

## 5. Phase 5 候选：世界模型 / 数据生成 / 跨具身

来源：`DreamVLA`、`LingBot-World`、`Pegasus` 类数据生成、`CG-World` 类世界模型数据协议、跨具身 VLA、仿真/反事实/数据增强。拆成多个专题，不合并成「世界模型大杂烩」。

| 原始路径 | 资料类型 | 研究主题 | 是否已有总结 | 需联网核验 | 目标目录 | 处理方式 | 来源和版权风险 |
|---|---|---|---|---|---|---|---|
| `VLA/DreamVLA/` | 纯论文（PDF） | 世界模型 + VLA（DreamVLA） | 否 | 是 | `docs/03_papers/dreamvla/` + `docs/02_topics/`（世界模型） | 迁移（建笔记） | 论文版权 |
| `VLA/lingbot/analysis_LingBot_World.md` | 笔记 | 开源世界模型（LingBot-World） | 是 | 是 | `docs/02_topics/`（世界模型）+ `docs/03_papers/` | 随 lingbot 一并改写 | 论文版权 |
| `VLA/lingbot/analysis_LingBot_VA.md` 等 | 笔记 | 因果世界建模 / 深度建模 / 3D 重建 | 是 | 是 | `docs/02_topics/`（世界模型/感知） | 随 lingbot 一并改写 | 论文版权 |
| `VLA/x-vla/`（跨具身部分） | 笔记 | 跨具身 VLA | 是 | 是 | `docs/02_topics/`（跨具身） | 随 x-vla 一并改写 | 论文版权 |
| `VLA/lingbot/analysis_Qwen_VLA.md` | 笔记 | 跨任务/环境/本体统一建模 | 是 | 是 | `docs/02_topics/`（跨具身） | 随 lingbot 一并改写 | 论文版权 |
| `VLA/数据质量评估/papers/`（RoboGen、Open X-Embodiment、MinInter） | 大型 PDF | 生成式仿真 / 跨具身数据 / 数据增强 | 否（配套笔记已在 Phase 2） | 是 | `docs/02_topics/`（数据生成与跨具身）+ `docs/03_papers/` | 仅建索引（PDF 不纳入 git）；提炼到专题 | 论文版权 |
| `VLA/Bimanual_robot_arm/`（MimicGen、DexMimicGen、RDT_1B） | 混合 | 合成演示数据生成 / 扩散 Transformer | 否 | 是 | `docs/02_topics/`（数据生成）+ `docs/03_papers/rdt_1b/` | 仅建索引 / 暂缓；排除 `*.zip`、权重 | 论文版权；RDT 代码许可待查 |
| `VLA/libero/`（LIBERO benchmark） | 纯论文（PDF） | 仿真基准 | 否 | 是 | `docs/03_papers/libero/` 或 `docs/02_topics/` 索引 | 仅建索引 / 暂缓 | 论文版权 |
| `Pegasus*`（数据生成） | 未发现 | 数据生成 | — | — | — | 暂缓（待获取） | — |
| `CG-World*`（世界模型数据协议） | 未发现 | 世界模型数据协议 | — | — | — | 暂缓（待获取） | — |

---

## 6. 跨阶段复用资产（避免重复）

- **`VLA/lerobot/docs/policy_analysis/`（23 篇）**：覆盖 Phase 4 大半模型的代码级分析（ACT/pi0/SmolVLA/openvla/x-vla/SAC/RDT 等）。编目时做交叉引用，作为对应论文笔记的底稿，不重复新建。
- **`VLA/数据质量评估/papers/`**：含 RoboGen(34M)、Open X-Embodiment、MinInter 等大型 PDF，配套笔记已在 Phase 2 落地。本库只建索引（路径 + 一句话），PDF 不纳入 git。
- **`VLA/AR_VLA/AR-VLA-lerobot/`**：仅 6 个源文件的轻量代码仓，作为 AR-VLA 笔记的「官方代码」核验参考，迁移时不带 `.git`。

## 7. 状态汇总

| 阶段 | 状态 | 说明 |
|---|---|---|
| Phase 1 建立迁移清单 | 进行中（本文件） | 已编目 Phase 2-5 全部候选 + 排除项 |
| Phase 2 数据质量评估 | 已完成 | 专题 + 4 论文笔记 + 矩阵/日志 + 4 PPTX 索引 |
| Phase 3 RL 与 VLA 后训练 | 已完成（3a–3e + 动作块与流匹配专题） | RL 基础 / RIPT-VLA / VLA 后训练专题 / 可复现实验 / RL_and_VLA·SimpleVLA_RL 索引 / 动作块与流匹配专题 |
| Phase 4 VLA 论文库 | 已完成 | AR-VLA/X-VLA 笔记 + LingBot 7 篇笔记 + 9 纯 PDF 模型索引（OpenVLA/π0/RT-1/SmolVLA/TriVLA/UD-VLA/VLA-Adapter/OpenVLA-OFT/π0.5）；PDF 仅索引不进 git |
| Phase 5 世界模型/数据生成/跨具身 | 进行中（种子级） | 世界模型+跨具身种子专题；LingBot-World/VA、DreamVLA、TriVLA、Foresight、X-VLA、Qwen-VLA、VLA_Aerial 已挂接；Pegasus、CG-World 未找到，暂缓 |

> 默认边界：原始资料不删除、不移动；优先迁移 Markdown 与可验证结论；大型文件只记录索引，不直接纳入 Git；代码仓库只在确实需要复现实验时建立关联。
