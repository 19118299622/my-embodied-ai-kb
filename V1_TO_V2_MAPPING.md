---
title: V1 → V2 映射表
type: project
status: active
created: 2026-08-07
updated: 2026-08-07
tags:
  - project-maintenance
  - architecture
  - v2-design
---

# V1_TO_V2_MAPPING — 逐项映射（Phase 1b）

> 本文件是 V1 → V2 架构升级的 **Phase 1b 产出**：把 `V1_INVENTORY.md` 审计出的**每一个真实存在的对象**（路径、字段、取值、隐含概念、技术债务），映射到 `V2_ARCHITECTURE_PROPOSAL.md` 定义的目标形态。
>
> 本文件**只描述映射，不执行迁移**。执行节奏见 [`MIGRATION_PLAN_V2.md`](./MIGRATION_PLAN_V2.md)。
>
> 配套文档：[`V1_INVENTORY.md`](./V1_INVENTORY.md)、[`V2_ARCHITECTURE_PROPOSAL.md`](./V2_ARCHITECTURE_PROPOSAL.md)。

---

## 0. 图例（必须先读）

### 0.1 迁移策略代码

| 代码 | 含义 | 对文件的实际操作 |
|---|---|---|
| `KEEP` | 原地保留，不动 | 无 |
| `ANNOTATE` | 原地注入元数据，**正文一个字节都不改** | 仅在 front matter 增加 `uid` / 新增 sidecar |
| `COPY` | 复制到 V2 位置，V1 原文件保留 | 新增文件；V1 侧不变 |
| `MOVE` | `git mv` 到 V2 位置 | 路径变更；旧路径记入 `previous_paths` |
| `SPLIT` | 一个 V1 文件拆成多个 V2 对象 | 新建多个文件 + 原文件降级为导航页或快照 |
| `DERIVE` | 改为由脚本生成，人工版本停止维护 | 新增 `derived/…`；V1 人工版本保留但标注 deprecated |
| `FREEZE` | 冻结为不可变快照 | 复制到 `reports/`，标 `report.snapshot` |
| `ARCHIVE` | 移入归档区，只读 | 移动到 `archive/` 或加 `status: historical` |
| `DEFER` | 本轮不决定，留待用户裁决 | 无 |

### 0.2 自动化等级

| 等级 | 含义 | 判定依据 |
|---|---|---|
| **A3** | Agent 可完全自动执行 | 纯派生产物，删了能一条命令重建，错了零损失 |
| **A2** | Agent 自动执行，但只落到 nightly 分支等待人工 review | 可逆、机械、有确定性规则，但触碰了 canonical 文件 |
| **A1** | Agent 只能**提议**，人工确认后才执行 | 需要语义判断，或涉及知识内容的取舍 |
| **A0** | **禁止自动**，仅人工 | 涉及用户观点、结论强度、领域划分、删除 |

> 判定原则（与 `V2_ARCHITECTURE_PROPOSAL.md` §15 权限矩阵一致）：
> **凡是「做错了用户不看 diff 就发现不了」的操作，一律 ≤ A1。**

### 0.3 风险标度

`低` = 错了改回来成本 < 5 分钟；`中` = 需要人工复核一批文件；`高` = 可能丢失用户知识或历史。

---

## 1. 顶层文件映射

| V1 路径 | V2 目标 | 策略 | 风险 | 自动化 | 说明 |
|---|---|---|---|---|---|
| `README.md` | `README.md`（重写为 HOME，见 V2 §11.2） | `SPLIT` | 中 | **A0** | V1 的 README 混合了「入口 + 目录说明 + tags 白名单 + 使用约定」。白名单迁往 `schema/`，使用约定迁往 `AGENTS.md`，只留导航。**这是用户的门面文件，不允许自动改写。** |
| `AGENTS.md` | `AGENTS.md` v2（角色化权限矩阵 + 五条红线） | `SPLIT` | 中 | **A0** | §1–§8 的写作规范全部保留；§9 Git 规则升级为 V2 §15 权限矩阵；新增 intake 与 nightly 章节。V1 §7「当前研究关注点」下沉为 `maps/embodied-ai.md` 的「当前焦点」区 |
| `CHANGELOG.md` | `CHANGELOG.md` | `KEEP` | 低 | **A1**（仅追加） | 保持人工语义变更日志。**不要**被 nightly 报告污染——机器运行记录进 `reports/nightly/`（Operational History，非 Derived） |
| `RESEARCH_MAP.md` | 分解为 4 类产物 | `SPLIT` | **高** | **A0** | 详见 §7，全文档最需要人工判断的一项 |
| `PROJECT_INVENTORY.md` | `derived/indexes/inventory.md` | `DERIVE` | 低 | **A3** | V1 声明 34 条 / 实际 85 个文件，已严重过期（D6）。转派生后过期问题结构性消失。原文件保留一轮并加 deprecated 提示，确认派生版可用后删除 |
| `MIGRATION_BACKLOG.md` | `KEEP` → 完成后 `ARCHIVE` | `DEFER` | 低 | **A0** | 用户明确要求不删。它记录的是**尚未完成的历史材料迁移**（V1→KB），与本轮的 V1→V2 结构迁移是两件事，不可混为一谈 |
| `MIGRATION_CHECKLIST.md` | §1 排除规则 → `schema/intake-rules.yaml`；五分类处置词表 → intake router；文件本体 `KEEP` | `COPY` + `KEEP` | 低 | **A1** | 这是 V1 最被低估的资产：它已经是一份 intake 规则说明书（I6）。V2 直接复用它的五个处置词，不另发明 |
| `.gitattributes` | 追加 `*.meta.yaml linguist-generated=true`、`derived/** linguist-generated=true` | 追加 | 低 | **A1** | LFS 规则原样继承（已验证可用，9 个对象在 LFS） |
| `.gitignore` | 追加 `sources/files/`（若采用「大文件不入库」策略） | 追加 | 低 | **A1** | `.workbuddy/` 排除规则保持 |
| `assets/README.md` | `KEEP` | `KEEP` | 低 | **A3** | 目前为空壳，无迁移价值 |
| `V1_INVENTORY.md` / `V2_ARCHITECTURE_PROPOSAL.md` / `V1_TO_V2_MAPPING.md` / `MIGRATION_PLAN_V2.md`（本轮 4 份） | 最终归入 `kb/agent-engineering/` 或 `docs/_meta/` | `DEFER` | 低 | **A0** | 它们本身就是 `agent-engineering` 领域的知识对象，是 V2 「元领域」的第一批内容。放置位置待用户定 |

---

## 2. 目录层映射（docs/ 六层）

### 2.1 总规则

```text
V1: docs/<NN_层名>/<簇>/<文件>.md          路径 = 分类
V2: kb/<domain>/<role>/<slug>.md           元数据 = 分类，路径只是默认落点
```

**关键变化**：V1 中「路径即类型」，而实测已证明二者早就不一致（I8：`ript_vla/` 下有 3 个 `type: topic` 文档却位于 `03_papers/`）。V2 中 `type` 与 `domain` 来自 front matter，路径仅为人类浏览的**默认约定**，不再是类型的真源。

### 2.2 逐层映射

| V1 目录 | 文件数 | V2 目标 | 默认 type | 策略 | 风险 | 自动化 |
|---|---:|---|---|---|---|---|
| `docs/01_foundations/` | 3 | `kb/embodied-ai/concepts/`（RL 基础也可归 `kb/machine-learning/concepts/`） | `note.concept` | `MOVE`（M6 之后） | 中 | **A1** |
| `docs/02_topics/` | 10 | `kb/embodied-ai/topics/` | `synthesis.topic` | `MOVE` | 中 | **A1** |
| `docs/03_papers/` | 44 | `kb/embodied-ai/literature/` + `sources/registry/` | `note.literature` | `MOVE` + `SPLIT` | **高** | **A1**（拆分部分 A0） |
| `docs/04_experiments/` | 2 | `kb/embodied-ai/experiments/` | `record.experiment` | `MOVE` | 低 | **A1** |
| `docs/05_ideas/` | 1（仅 README，空） | `kb/embodied-ai/ideas/` | `hypothesis.idea` / `hypothesis.question` | 新建 | 低 | **A2** |
| `docs/06_reports/` | 5 | `kb/embodied-ai/reports/` + `sources/files/` | `report.snapshot` | `MOVE` + `SPLIT` | 中 | **A1** |
| `docs/0X_*/README.md`（6 个） | 6 | 由 `derived/indexes/by-domain.md` 取代；目录 README 降级为「一句话说明 + 指向派生索引」 | — | `DERIVE` | 低 | **A2** |
| `templates/`（7 个） | 7 | `templates/` 升级为按 object type 命名 | — | `COPY`（新增 typed 版本） | 低 | **A1** |

> **本阶段红线**：以上 `MOVE` 全部标注为 M6 之后（`MIGRATION_PLAN_V2.md` §4）。第一阶段 `docs/` 只允许两类零风险改动：**注入 uid** 与 **修链接**。

---

## 3. 逐簇映射（cluster-level）

V2 把「簇」作为迁移的**最小事务单元**（V2 §3.2）：一次迁一整簇，绝不拆散。以下是 V1 全部 12 个真实簇。

| # | V1 簇 | 内容 | V2 目标 | cluster id | 策略 | 风险 | 自动化 |
|---:|---|---|---|---|---|---|---|
| C1 | `03_papers/ript_vla/` | 1 笔记 + 3 深入分析 + 1 PDF(LFS) | 笔记 → `literature/ript-vla.md`；3 篇分析 → `topics/`（它们 `type: topic`）；PDF → `sources/files/` + `registry/arxiv-2505.17016.yaml` | `embodied-ai/ript-vla` | `MOVE`+`SPLIT` | 中 | **A1** |
| C2 | `03_papers/d6_failure_detection/` | 9 篇笔记 + 1 INDEX | 9 篇 → `literature/`；INDEX 拆为 9 条 source registry + 1 条派生索引 | `embodied-ai/failure-detection` | `MOVE`+`SPLIT` | 中 | **A1** |
| C3 | `03_papers/lingbot/` | 7 篇 analysis + 1 INDEX | 6 篇 analysis → `literature/`；`analysis_summary.md`（`type: topic`）→ `topics/`；INDEX 同上拆分 | `embodied-ai/lingbot` | `MOVE`+`SPLIT` | 中 | **A1** |
| C4 | `02_topics/embodied_data_quality/` | 总览 + 分层阅读计划 + L1精读总结 + PAPER_MATRIX + READING_LOG | 总览 → `synthesis.topic`；分层阅读计划 → `plan.learning`；L1 总结 → `synthesis.topic`；PAPER_MATRIX → **`DERIVE`**（矩阵可由 literature 元数据生成）；READING_LOG → **`log.reading`**（新 role） | `embodied-ai/data-quality` | `MOVE`+`DERIVE` | **高** | **A0**（PAPER_MATRIX 派生化需人工确认列定义） |
| C5 | `01_foundations/reinforcement_learning/` | 学习规划与进度 + 长期学习笔记 | 规划 → `plan.learning`（Agent 可改进度字段行）；笔记 → `note.concept`（living，Agent 不可改正文） | `machine-learning/rl-basics` 或 `embodied-ai/rl-basics` | `MOVE` | 中 | **A0**（领域归属需用户裁决） |
| C6 | 13 个单 `INDEX.md` 论文目录（`dreamvla` `openvla` `openvla_oft` `pi0` `pi05` `rl_and_vla` `rt1` `simplevla_rl` `smolvla` `trivla` `ud_vla` `vla_adapter` `vla_aerial`） | 每个只有一个 INDEX.md，无正文笔记 | 每个 → `sources/registry/<id>.yaml`（**它们本质是来源卡片，不是知识**） | 各自独立 | `SPLIT` | 中 | **A1** |
| C7 | 4 个「PDF + 笔记」论文对（`a_survey_of_datasets` `data_assessment_for_embodied_intelligence` `data_quality_in_imitation_learning` `robot_data_curation_…`） | 笔记 + PDF(LFS) | 笔记 → `literature/`；PDF → `sources/files/`（保持 LFS）；新增 registry | 各自独立 | `MOVE` | 中 | **A1** |
| C8 | 3 个单笔记论文（`ar_vla` `x_vla` + `d6` 内独立项） | 仅笔记 | → `literature/` | 各自独立 | `MOVE` | 低 | **A2** |
| C9 | `02_topics/` 其余 4 个单文档专题（`action_chunk_flow_matching` `vla_post_training` `vla_runtime_verification_online_learning` `world_model`） | 各 1 篇 | → `topics/` | 各自独立 | `MOVE` | 低 | **A2** |
| C10 | `04_experiments/PPO_CartPole_可复现实验记录.md` | 1 篇（结果待回填） | → `experiments/` | `machine-learning/ppo-cartpole` | `MOVE` | 低 | **A2** |
| C11 | `06_reports/数据质量评估进度汇报/` | 4 个 PPTX(LFS) | PPTX → `sources/files/`；**新建** 1 个 `report.snapshot` 笔记描述汇报内容与时点 | `embodied-ai/data-quality` | `MOVE`+新建 | 中 | **A1** |
| C12 | `templates/`（7 件套） | 模板 | → typed templates（见 §6） | — | `COPY` | 低 | **A1** |

> **注意 C6**：13 个只有 `INDEX.md` 的目录是 V1 最容易被误读的部分。它们不是「未完成的论文笔记」，而是**已完成的来源登记**——用户读了论文、记了元数据、决定暂不写笔记。V2 必须把它们识别为 Source 层对象，**而不是当成空文档去催促补全**。

---

## 4. 特殊对象映射

### 4.1 `INDEX.md`（15 个）——V1 最纠缠的对象

V1 的 `INDEX.md` 同时承担两种互斥职责（I9 / D8 同源问题）：

| 职责 | 内容示例 | V2 去向 | 自动化 |
|---|---|---|---|
| (a) 来源元数据卡片 | 论文标题、arXiv 链接、作者、项目主页、官方代码、本地 PDF 路径、核验日期 | `sources/registry/<source-id>.yaml` | **A1**（字段抽取机械，但 `official_repo` 是否官方需人工确认——AGENTS §6.2） |
| (b) 目录导航表 | 「本目录包含哪些文件」 | `derived/indexes/by-cluster.md` | **A3** |

映射规则：

```yaml
# V1: docs/03_papers/openvla/INDEX.md 的字段
论文链接          → source.canonical_url
本地 PDF 路径      → source.local_hint  (portable: false)
核验日期          → source.last_verified_at
官方代码          → source.official_repo  (need_human_confirm: true)
目录文件列表       → 丢弃，改由派生索引生成
```

**风险**：15 个 INDEX 中的 `本地 PDF 路径` 全部是 `D:/materials/VLA/...` 形式的绝对路径（D4）。V2 明确**不假装解决**：标 `portable: false`，优先补 `canonical_url`，让健康报告统计「不可移植来源」的数量（V2 §22 风险 11）。

### 4.2 PDF / PPTX（9 个 LFS 对象）

| V1 位置 | V2 目标 | 策略 | 风险 | 自动化 |
|---|---|---|---|---|
| `03_papers/<簇>/*.pdf`（5 个） | `sources/files/<source-id>.pdf` + `sources/registry/<source-id>.yaml` | `MOVE` | **高** | **A0** |
| `06_reports/数据质量评估进度汇报/*.pptx`（4 个） | `sources/files/` 或保持原地 | `DEFER` | **高** | **A0** |

> **为什么是 A0**：移动 LFS 对象涉及指针文件与 LFS 存储的一致性，且 Windows + 中文文件名 + LFS 的组合已被列为工程风险（V2 §22 风险 10）。**nightly 永远不得触碰 LFS 文件**（V2 §17.4）。这一步必须人工、单独一次提交、迁完立刻验证 `git lfs ls-files`。

### 4.3 目录 `README.md`（6 个）

| 组成部分 | V2 去向 | 自动化 |
|---|---|---|
| 「本目录是什么」一句话 | 保留在 `kb/<domain>/_domain.md` | **A0** |
| 手写文件条目表 | `derived/indexes/by-domain.md` | **A3** |
| 命名约定 / 写作要求 | `AGENTS.md` v2 或 `schema/` | **A0** |

`docs/04_experiments/README.md` 声明「（暂无）」但实际有 1 篇（D6）——这是人工维护索引必然漂移的直接证据，转派生后该类错误在结构上不可能再发生。

### 4.4 `templates/`（7 → typed templates）

| V1 模板 | V2 object type | 主要升级 |
|---|---|---|
| `PAPER_NOTE_TEMPLATE.md` | `note.literature` | 合并 D6 的 22 字段 schema；增加 `uid` 占位、`source_id`、`temporal_class` |
| `TOPIC_TEMPLATE.md` | `synthesis.topic` | 增加 `relations` 区块 |
| `EXPERIMENT_TEMPLATE.md` | `record.experiment` | 增加 `code_commit` / `dataset_version` 必填 |
| `RESEARCH_QUESTION_TEMPLATE.md` | `hypothesis.question` | **本模板已存在但从未被使用**——V2 的 `RESEARCH_MAP §4` 拆分正好填满它 |
| `LEARNING_PLAN_TEMPLATE.md` | `plan.learning` | 增加 `stages[]` / `current_stage` 结构化字段（Agent 唯一可写的正文行） |
| `LONG_TERM_NOTES_TEMPLATE.md` | `note.concept` | 明确 living 语义与追加区块位置 |
| `REPORT_TEMPLATE.md` | `report.snapshot` | 增加 `frozen_at`；清理 3 处 `链接` 占位符（它们污染链接校验，D3） |

**7 个模板实质上是 7 份 object type schema 草案**——V2 的类型系统不是新发明，是把它们形式化。

---

## 5. 元数据字段映射（field-by-field）

### 5.1 通用字段（63 个文件都有）

| V1 字段 | V2 位置 | V2 字段名 | 变化 | 自动化 |
|---|---|---|---|---|
| `title` | front matter | `title` | 无 | — |
| `type` | front matter | `type` | **取值改变**，见 §5.3 | **A1** |
| `status` | front matter | `document_state` + `knowledge_status`（两个正交字段，见 §5.4） | **拆分 + 取值改变**，见 §5.4 | **A1** |
| `created` | front matter | `created` | 无（V1 兼容） | — |
| `updated` | front matter | `updated` | 语义收紧：仅正文字节变化时更新 | **A2** |
| `tags` | front matter | `tags` | 归一化，见 §5.5 | **A1** |
| — | front matter | **`uid`**（新增，必填） | 一次性注入 | **A2**（注入机械；但批次由人定） |
| — | front matter | **`domain`**（新增，必填） | 首批全部 `embodied-ai`，C5 除外 | **A1** |

### 5.2 局部字段

| V1 字段 | 出现 | V2 位置 | V2 形态 | 自动化 |
|---|---:|---|---|---|
| `migrate_source` | 16 | sidecar | `previous_paths[]` + `relations: derived_from` | **A2** |
| `short_name` | 16 | front matter | `aliases[]` 的第一项 + 参与 `identity_key` 生成 | **A2** |
| `pdf_index` | 15 | source registry | `local_hint` + `portable: false` | **A1** |
| `source_note` | 2 | sidecar | 并入 source registry `notes` | **A2** |
| `authors` `venue` `year` `paper_link` `paper_version` | 各 9 | **source registry** | 来源属性，不属于笔记 | **A1** |
| `code_link` `project_link` `local_file` | 各 9 | **source registry** | 同上；`code_link` 需人工确认是否官方 | **A0**（官方性判定） |
| `read_date` `read_status` `primary_level` | 各 9 | front matter（类型扩展） | 阅读行为属于笔记，保留 | **A2** |
| `primary_dimension` `related_dimensions` | 各 9 | front matter | 保留；同时生成 `relations: applies_to` 指向维度节点 | **A1** |
| `keywords` | 9 | front matter | **合并进 `tags`**（消除双词表） | **A1** |

> **重要判据**：字段迁移的分界线是「**这是关于来源的，还是关于我的阅读的？**」——`authors`/`venue`/`year` 属于论文本身（Source），`read_date`/`read_status`/`primary_level` 属于我的阅读行为（Knowledge）。V1 把两者混在同一份 front matter 里，这正是 D7 schema 分叉的根因。

### 5.3 `type` 取值映射

| V1 值 | 出现位置 | V2 值 | 说明 | 自动化 |
|---|---|---|---|---|
| `paper-note` | 论文笔记 | `note.literature` | 直接映射 | **A2** |
| `topic` | 专题 | `synthesis.topic` | 直接映射 | **A2** |
| `foundation` | 基础 | `note.concept` | 直接映射 | **A2** |
| `experiment` | 实验 | `record.experiment` | 直接映射 | **A2** |
| `idea` | 想法 | `hypothesis.idea` | 直接映射（V1 无实例） | **A2** |
| `report` | 汇报 | `report.snapshot` | 直接映射 | **A2** |
| `index` | INDEX.md | **拆分**：来源部分 → `source.record`；导航部分 → `index.*`（派生） | 语义重复项，见 §4.1 | **A1** |
| `paper-index` | INDEX.md | 同上，**与 `index` 合并** | V1 的重复定义在此消除 | **A1** |
| `project` | 治理文档 | `note.concept`（domain: `agent-engineering`）或无 type（治理文档不进 kb） | 待定 | **A0** |
| `research-map` | `RESEARCH_MAP.md` | `map.moc` | 见 §7 | **A0** |
| — | `READING_LOG.md` | **`log.reading`**（新增） | V1 无处安放，混在 topic 里（I7） | **A1** |
| — | `RESEARCH_MAP §4` 的 5 个开放问题 | **`hypothesis.question`**（新增） | 见 §7 | **A0** |
| — | `分层阅读计划.md` / `学习规划与进度.md` | **`plan.learning`** | V1 用 `topic`/`foundation` 凑合 | **A1** |

### 5.4 `status` 取值映射（V1 单一 `status` → V2 双正交轴，保守迁移）

V1 的单一 `status` 同时承载「文档生命周期」与「知识认识论状态」，导致两者都无法被清晰使用（D9）。V2 拆成两个正交字段。

> **保守迁移铁律（rev2 / R2-7）**：自动迁移**不得**自动提升知识的认识论可信度。`V1 active` 表示「文档当前处于活跃维护状态」，**不等于** `knowledge_status: current`（「里面承载的知识已被认为当前有效」）。凡无法证明 `knowledge_status` 的，**默认 `unverified`**，且该判断需要语义判断，**不得列为纯机械 A2 自动迁移**（降为 A1）。

**轴一 `document_state`（生命周期）——可机械映射**

| V1 值 | 实际使用 | V2 `document_state` | 自动化 |
|---|---|---|---|
| `stable` | **0 次** | `stable`（保留——它本就属于生命周期轴，只是 V1 错放在 status 里） | — |
| `archived` | **0 次** | `archived`（保留，语义=停止主动维护） | — |
| `active` | 大量 | `active` | **A2**（机械） |
| `seed` | 大量 | `draft`（初稿未成熟，符合 V1 `seed` 本意） | **A2**（机械） |
| — | — | `frozen`（新增） | — |

**轴二 `knowledge_status`（认识论）——需语义判断，默认 `unverified`**

| V1 值 | V2 `knowledge_status` | 自动化 | 说明 |
|---|---|---|---|
| `seed` | `unverified` | **A1**（语义判断） | 初稿天然未核验 |
| `active` | **默认 `unverified`**；仅当文档已有明确证据（已 review / 已核验 / 显式标 current）才升为 `current` | **A1**（语义判断） | **`active` ≠ `current`**——活跃维护不代表知识已被确认有效 |
| `stable` | `unverified`（除非有证据） | **A1**（语义判断） | `stable` 描述内容成熟度，不自动等于认知有效 |
| `archived` | `unverified`（除非有证据） | **A1**（语义判断） | **`archived` ≠ `historical`**：文档可能只是「不再维护」，但其中知识仍可 `current`；二者正交，不得由 archived 推断 |
| — | `disputed`（新增） | **A1**（Agent 只能提议） | |
| — | `superseded`（新增，需 `superseded_by`） | **A0** | |
| — | `historical`（新增） | **A0** | |

> 两轴正交：`active` 文档可同时是 `unverified`（刚写完还没核验）或 `current`（已确认）。V1 的 `stable` / `archived` **不再废弃**，而是归入 `document_state`——这是 rev1 对 P0-1 的修正；rev2 进一步规定 `knowledge_status` 不得被自动推断为 `current`（R2-7）。

### 5.5 `tags` 归一化（39 → 受控集）

| 处理 | V1 值 | V2 去向 | 自动化 |
|---|---|---|---|
| **升格为 domain** | `embodied-ai` | `domain: embodied-ai`（不再是 tag） | **A1** |
| **升格为 cluster** | `lingbot`、`d6`、`ript-vla` 类模型/论文名 | `cluster` 字段 | **A1** |
| **保留为 tag** | `vla`、`reinforcement-learning`、`world-model`、`data-quality`、`imitation-learning`、`flow-matching`、`action-chunking`、`post-training`、`failure-detection` 等技术概念 | `tags[]` | **A2** |
| **合并同义** | `vla-post-training` → `post-training`；`robot-control` → `control` | 归一化表 `schema/tag-aliases.yaml` | **A1** |
| **降级为正文** | `open-source`、`engineering`、`lego` 等一次性描述 | 删除 tag，信息保留在正文 | **A0**（删任何东西都要人确认） |
| **来自 `keywords` 字段的 9 组** | 并入 tags 后统一归一化 | 同上 | **A1** |

> **V2 的态度转变**：V1 试过「README 白名单 + 人工自律」，6 天漂到 2.8 倍（D5）。V2 不再假设自律有效——
> **`domain` / `type` / `relations` 是闭集（机器校验，Agent 无权新增）；`tags` 是开集（允许自由生长，但只用于探索，不作为权威检索维度）。**
> 这个分工比「再列一次白名单」更可能存活。

---

## 6. 概念映射（V1 隐含设计 → V2 显式机制）

| V1 隐含设计 | V1 形态 | V2 机制 | 是否保留原语义 | 自动化 |
|---|---|---|---|---|
| I1 证据强度标注 `[PAPER]/[RESULT]/[INFERENCE]/[OPEN]` | 写作规范（D6 全文使用） | **升格为 Agent 写保护边界**：带 `[INFERENCE]` / `[OPEN]` 的段落进入不可自动改写集合（V2 §8.4） | **完全保留并强化** | **A0**（保护规则本身不可自动放宽） |
| I2 核验日期 | 被动字段 `read_date` / 正文「核验日期：」 | `last_verified_at` + `temporal_class` → `verification_due_at = last_verified_at + SLA`（确定性）→ `derived/health/stale.md` 记录 `stale/not-stale` | 保留并变为主动；不依赖隐式系统时间 | **A3**（计算 due_at，确定性） |
| I3 迁移溯源 | `migrate_source` 字段 + 正文引用块 | `previous_paths[]` + `relations: derived_from` | 保留 | **A2** |
| I4 Source / Knowledge 分离 | 「PDF 不进 git，只建索引」的默认实践 | `sources/registry` + `sources/files` 显式两层 | 保留并命名 | **A1** |
| I5 双文档 living 模式 | `学习规划与进度.md` + `长期学习笔记.md` | 正式化为 `plan.learning` + `note.concept` 配对（V2 §8.5） | **保留，且被指定为 living document 的原型** | **A1** |
| I6 五分类处置词表 | `MIGRATION_CHECKLIST`：迁移/改写/仅建索引/暂缓/排除 | **intake router 直接复用这五个词**，不另发明 | 完全保留 | **A2**（分诊建议） |
| I7 簇作为组织单元 | 目录自发形成 | `cluster` / `cluster_role` 一等字段；重复检测与迁移事务均以簇为单位 | 保留并显式化 | **A1** |
| I8 type 与路径解耦 | 事实上已不一致 | type 来自 front matter，路径仅为默认落点 | 承认现实 | **A2** |
| I9 INDEX.md 双重职责 | 一个文件两种用途 | **强制拆分**：source registry + derived index | 拆分 | **A1** |
| I10 Agent 只 add 不 commit | `AGENTS §9` | `default_agent` 角色基线；`nightly_curator` 可在独立分支 commit 但**永不 merge** | 保留并细化 | **A0**（红线） |

---

## 7. `RESEARCH_MAP.md` 逐节映射（明确要求项）

这是全仓库单点信息密度最高、也最需要人工拆分的文件（D8）。以下把 V2 §13.2 的方案落到逐节。

| V1 节 | 内容 | V2 目标 | 产物类型 | 策略 | 风险 | 自动化 |
|---|---|---|---|---|---|---|
| §1 总体研究闭环（Mermaid） | 数据→训练→部署→验证→回流的认知框架 | `maps/embodied-ai.md` 顶部 | `map.moc` | `MOVE` | 低 | **A0** |
| §2.1–2.7 七条主线标题与定位 | 主线名 + 一句话 | `maps/embodied-ai.md` 主体 | `map.moc` | `MOVE` | 低 | **A0** |
| §2.x「核心问题」清单 | 每条主线下 3–6 个研究问题 | **每个问题一个 `hypothesis.question` 节点** → `kb/embodied-ai/ideas/` | `hypothesis.question` | `SPLIT` | **高** | **A0** |
| §2.x「代表工作（已迁移）」链接列表 | 手写的论文链接表 | **删除人工版本** → `derived/indexes/by-domain.md` + 各 topic 的 `relations` | `index` | `DERIVE` | 中 | **A3**（生成）/ **A1**（删原表） |
| §2.x「当前优先级：最高/高/中」 | 优先级判断 | `maps/embodied-ai.md` 的「当前焦点」区 | `map.moc` | `MOVE` | 低 | **A0** |
| §3 当前专题关系（Mermaid） | 人画的专题依赖图 | `maps/embodied-ai.md`，**并与 `derived/graph/graph.json` 交叉校验** | `map.moc` + 校验 | `MOVE` | 中 | **A0**（图）/ **A3**（校验报告） |
| §4 五个开放问题（含公式、子问题、验证思路） | 全仓库信息密度最高的部分 | **五个独立 `hypothesis.question` 节点**，各含 `falsifier` / `minimal_experiment` / `confidence` | `hypothesis.question` | `SPLIT` | **高** | **A0** |
| §5 近期建议产出（16 项） | 混合了待办与已完成 | 待办 → `plan` 或 issue；已完成 → **删除**（`CHANGELOG` + `derived/recent.md` 已覆盖） | 拆分 | `SPLIT` | 中 | **A1**（分类建议）/ **A0**（删除） |
| §6 维护规则 | 「谁在什么时候更新这张图」 | `AGENTS.md` v2 | 治理 | `MOVE` | 低 | **A0** |
| 整体（2026-08-07 时点） | 某一时刻的研究状态全景 | 冻结副本 `kb/embodied-ai/reports/2026-08-07-research-state.md` | `report.snapshot` | `FREEZE` | 低 | **A1** |
| 文件本体 | — | 分解完成后降级为**短导航页**，指向 `maps/` 与各 question 节点 | — | `KEEP`（内容替换） | 中 | **A0** |

> **为什么 §4 必须拆**：现在这五个开放问题**无法被单独引用、无法记录进展、无法标记「已回答」、无法与论文建立 `supports` / `contradicts` 关系**。它们是用户思考的最高价值产物，却被埋在一个地图文件的第四节里。拆成节点后，`derived/graph/` 能立刻回答「哪些论文与我的开放问题 3 相关」。
>
> **为什么全部 A0**：这一节的每一行都是用户的研究判断。任何自动拆分都可能改变问题的措辞与边界，而措辞就是研究问题本身。

---

## 8. 链接与路径映射

| V1 问题 | 数量 | V2 处理 | 策略 | 风险 | 自动化 |
|---|---:|---|---|---|---|
| 失效内部链接（多写 `docs/` 段、缺 `../`、跨层错误） | 21（含 3 个模板占位符） | `tools/check_health.py` 检出 → 相对路径重算 | 修复 | 低 | **A2**（18 条可自动，3 个模板占位符改为非链接文本） |
| 越界链接 `../../../../数据质量评估/` | 5 | 转为 `sources/registry` 中的 `local_hint`，正文链接改为指向 source 记录 | `SPLIT` | 中 | **A1** |
| `pdf_index` 绝对路径 `D:/materials/VLA/...` | 15 | `source.local_hint` + `portable: false`；优先补 `canonical_url` | 迁移 | 中 | **A1** |
| 未来的路径变更断链 | — | `uid` + `previous_paths` + `derived/indexes/uid-map.json` 双向表；健康检查自动修复指向旧路径的链接 | 结构性解决 | 低 | **A2** |
| 中文文件名八进制转义（Windows + Git） | 全库 | `core.quotepath=false`、`core.longpaths=true`；V2 内部一律用 `uid` 寻址 | 配置 | 低 | **A3** |
| 12 个孤儿文档（0 入链） | 12 | `derived/indexes/` 兜底覆盖 + `derived/health/orphans.md` 持续报告 | 派生兜底 | 低 | **A3** |

---

## 9. 技术债务 → V2 解法对照

| # | V1 债务 | V2 解法 | 何时解决 | 自动化 |
|---|---|---|---|---|
| D1 | 无稳定标识 | `uid`（§7 V2）一次性注入 | M2 试点 / M4 全量 | **A2** |
| D2 | Git 无文档级历史 | **不可追溯性无法修复**；只能从 V2 起累积。故第一阶段绝不自动写 canonical | 从 V2 起 | **A0** |
| D3 | 21 条断链 | `check_health.py` + 自动修复 | **MVP 即可** | **A2** |
| D4 | 越界 / 绝对路径 | Source registry + `portable: false` 统计 | M3 | **A1** |
| D5 | 词表漂移 | 闭集机器校验（domain/type/relations）+ 开集降权（tags） | M1 | **A2** |
| D6 | 索引过期 | 全部索引改为派生 | **MVP 即可** | **A3** |
| D7 | schema 分叉 | 类型扩展字段：D6 的 22 字段成为 `note.literature` 的正式扩展 | M1 | **A1** |
| D8 | `RESEARCH_MAP` 混合四职责 | §7 分解方案 | M5 | **A0** |
| D9 | status 后两态未用（且 lifecycle 与 epistemology 混在一起） | `document_state` × `knowledge_status` 双正交轴 | M1 | **A1** |
| D10 | `05_ideas/` 为空 | `RESEARCH_MAP §4` 拆分后立刻有 5 个真实节点 | M5 | **A0** |
| D11 | 无校验脚本 | `tools/check_health.py` | **MVP 即可** | **A3** |
| D12 | 中文路径工程摩擦 | Git 配置 + uid 寻址 | MVP | **A3** |
| D13 | 12 个孤儿 | 派生索引兜底 | MVP | **A3** |

> **注意 D2 的性质**：它是**已发生且不可逆**的。V1 只有 2 次提交，意味着「用 Git 历史回滚某个文档的错误自动修改」这条安全网**在 V1 上根本不存在**。这直接决定了 V2 第一阶段必须 dry-run：在没有历史可回滚的情况下，任何自动写入都是单向操作。

---

## 10. 明确不迁移 / 不改动清单

| 对象 | 决定 | 理由 |
|---|---|---|
| `docs/` 下任何文件的**位置** | 第一阶段不动 | 用户明确禁止；且 uid 注入后再移动风险更低 |
| `docs/` 下任何文件的**正文** | 永不自动改 | 不变量 #2 |
| `MIGRATION_BACKLOG.md` / `MIGRATION_CHECKLIST.md` | 不删 | 用户明确要求；且它们是 intake 规则的来源 |
| Git 历史 | 不重写 | 红线 |
| 9 个 LFS 对象 | 第一阶段不动 | 工程风险高，需单独人工事务 |
| 仓库名 | 不改 | 用户明确说明 |
| `CHANGELOG.md` 的历史条目 | 不动 | 人工语义日志 |
| 4 个 PPTX 的内容 | 不解析、不摘要 | 属 Source 层，Agent 只登记不加工 |
| `.workbuddy/` | 不纳入知识库 | 项目记忆，已在 `.gitignore` |

---

## 11. 自动化裁定汇总

| 等级 | 涉及项数（本表） | 典型代表 | 共同特征 |
|---|---:|---|---|
| **A3 完全自动** | 9 | 派生索引、健康报告、复验债务计算、孤儿检测、Git 配置 | 纯派生，可重建，零损失 |
| **A2 自动 + 分支等待 review** | 21 | uid 注入、type 机械映射、断链修复、`updated` 维护 | 机械规则，可逆，但触碰 canonical（注：`status` 的认知轴 `knowledge_status` 需 A1 语义判断，不计入纯机械） |
| **A1 提议 + 人工确认** | 27 | 领域归属、tag 归一化、簇迁移、source registry 抽取、schema 扩展 | 需要语义判断 |
| **A0 禁止自动** | 19 | `RESEARCH_MAP` 拆分、观点删除、`superseded` 标记、LFS 移动、README/AGENTS 重写、领域新增 | 涉及用户知识判断或不可逆操作 |

**读法**：A0 + A1 合计 46 项，占绝大多数——这不是保守，而是承认一个事实：**这个仓库里 74 个文件的价值不在文件本身，而在用户对它们的判断。判断不能被自动化。**

---

## 12. 更新记录

- 2026-08-07：建立 V1 → V2 映射表（Phase 1b）。覆盖顶层文件 13 项、目录层 8 项、知识簇 12 个、特殊对象 4 类、元数据字段 23 个、取值映射 3 组（type 12 / status 7 / tags 6）、隐含设计 10 项、`RESEARCH_MAP` 逐节 11 项、链接问题 6 类、技术债务 13 项，并为每项给出迁移策略、风险与自动化等级。**本轮为设计文档，未执行任何迁移。**
- 2026-08-07（rev1）：**同步 V2 架构提案的协议缺陷修正**。§5.1 将 `status` 映射目标改为 `document_state` + `knowledge_status` 双正交字段；§5.4 重写为双轴取值映射（`stable`/`archived` 不再废弃，归入 `document_state`）；§9 D9 同步；§1 CHANGELOG 行将 nightly 报告位置改为 `reports/nightly/`（非 Derived）。**未执行任何迁移。**
- 2026-08-07（rev2）：**同步 V2 架构提案的协议边界收尾（R2 系列）**。§5.4 改为**保守迁移**：`seed→draft×unverified`、`active→active×默认 unverified`（不自动=`current`）、`stable`/`archived` 的 `knowledge_status` 默认 `unverified`；明确 `knowledge_status` 赋值需语义判断、降为 A1（非纯机械 A2）；§6 I2 健康指标改为确定性的 `verification_due_at=last_verified+SLA`（不依赖隐式系统时间）；§11 同步说明 `status` 认知轴非纯机械。**未执行任何迁移。**
