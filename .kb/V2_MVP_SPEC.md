---
title: V2 MVP 需求规格（轻量、Agent-first、Git-native）
type: spec
status: active
created: 2026-08-07
updated: 2026-08-07
tags:
  - v2-design
  - mvp
  - requirements
---

# V2_MVP_SPEC — 轻量 V2 MVP 需求基线

> ## ✅ 当前权威需求基线（2026-08-07 重新对齐）
>
> 本文档是 **V2 的当前权威需求基线**。
>
> 早期完整架构探索 [`.kb/history/V2_ARCHITECTURE_PROPOSAL.md`](./history/V2_ARCHITECTURE_PROPOSAL.md)、[`.kb/history/V1_TO_V2_MAPPING.md`](./history/V1_TO_V2_MAPPING.md)、[`.kb/history/MIGRATION_PLAN_V2.md`](./history/MIGRATION_PLAN_V2.md) 属于「未来参考」，若与本文冲突，**一律以本文为准**。
>
> 本文只固化**需求与边界**，不规定实现细节；本轮不创建任何实际仓库结构、协议文件或脚本（见 §13、§14）。

---

## 0. 目录

- [1. 背景与本轮范围](#1-背景与本轮范围)
- [2. 核心目标与日常循环](#2-核心目标与日常循环)
- [3. 知识目录结构（领域/问题优先）](#3-知识目录结构领域问题优先)
- [4. 旧 V1 迁移策略](#4-旧-v1-迁移策略)
- [5. 极简 Front Matter 规范](#5-极简-front-matter-规范)
- [6. 持续学习双文档 Living Document](#6-持续学习双文档-living-document)
- [7. Source 与 Knowledge 分离（论文）](#7-source-与-knowledge-分离论文)
- [8. 论文 Markdown 命名与 paper-summary](#8-论文-markdown-命名与-paper-summary)
- [9. `_map.md` 认知结构约束](#9-_mapmd-认知结构约束)
- [10. Relation MVP](#10-relation-mvp)
- [11. 主要找回机制](#11-主要找回机制)
- [12. 仓库协议栈与人类入口](#12-仓库协议栈与人类入口)
- [13. Markdown 写作规则继承](#13-markdown-写作规则继承)
- [14. Nightly 机制](#14-nightly-机制)
- [15. MVP 明确不实现的能力](#15-mvp-明确不实现的能力)
- [16. 与旧架构文档的关系](#16-与旧架构文档的关系)
- [17. 更新记录](#17-更新记录)

---

## 1. 背景与本轮范围

现有 [`.kb/history/V1_INVENTORY.md`](./history/V1_INVENTORY.md)、[`.kb/history/V2_ARCHITECTURE_PROPOSAL.md`](./history/V2_ARCHITECTURE_PROPOSAL.md)、[`.kb/history/V1_TO_V2_MAPPING.md`](./history/V1_TO_V2_MAPPING.md)、[`.kb/history/MIGRATION_PLAN_V2.md`](./history/MIGRATION_PLAN_V2.md) 是一次**完整但明显偏重**的架构探索。经过重新需求对齐，正式目标已收敛为一个 **Agent-first、Git-native、面向长期个人技术知识沉淀的轻量 V2 MVP**：

- **不做**数据库、向量库、Web UI、复杂 schema、通用 sidecar、复杂身份/状态/审查系统；
- **只做**基于 Git 与 Markdown 的、由 Agent 按仓库自身协议维护的知识沉淀循环；
- 现有完整架构**不再作为实施规格**，仅保留为未来参考与决策演化证据。

`V1_INVENTORY.md` 作为**事实审计基线**保持不变，不因新 MVP 修改其事实内容。

本轮范围**仅限**：新增本需求规格 + 对三篇旧设计文档做最小 authority 重定位。**不进入实际仓库改造或实现阶段。**

---

## 2. 核心目标与日常循环

用户白天将 `.pdf` / `.md` 手动放入 `inbox/`，由 Agent 按仓库自身协议完成以下动作：

1. **Source 登记**：识别来源（论文/资料），登记到 Source 层；
2. **Knowledge 归档**：将材料归入正确的知识目录（领域/问题优先，见 §3）；
3. **Living Document 安全更新**：在既有知识文档上做受约束的增量更新（见 §6、§9）；
4. **受约束的论文快速总结**：仅对符合 §8 条件的论文生成 `paper-summary`，并明确标记未经验证；
5. **`_map.md` 与明确链接维护**：填充已有 Map、补齐导航与横向链接（见 §9、§10）；
6. **提交 nightly branch 并在聊天窗口报告**本次策展结果。

**无法可靠判断的材料**：原样留在 `inbox/`，不强行分类、不臆测、不丢弃。Agent 应说明为何无法判断，交由用户后续处理。

---

## 3. 知识目录结构（领域/问题优先）

采用 **Domain（领域）→ Area（子领域）→ Topic（主题）→ 必要时 Subtopic（子主题）** 的**领域/问题优先**结构。

- **不再以 `papers/` `experiments/` `ideas/` 等文档类型作为主目录**；文档类型仅通过 Front Matter 的 `kind` 作为元数据（见 §5），不决定目录层级。
- 领域（Domain）是受控的顶层划分，反映长期知识主干与广度分支（例：具身智能/机器人为主干，机器学习、计算机系统、数学、研究方法、智能体工程等为广度分支）。
- Area / Topic / Subtopic 按实际问题与知识结构自然展开，不强制固定层数；Subtopic 仅在主题确实过大、需要再分时使用。
- 目录命名以稳定语义为准，避免日期前缀作为主目录名（阶段性观察/报告文件可用日期前缀，见 §13）。

---

## 4. 旧 V1 迁移策略

旧 V1 内容**按实际重新使用时才真正迁移**（「使用时迁移」），不进行一次性全量迁移：

- 当当前研究引用到某条旧知识、或用户明确要求时，才检查其是否仍然准确、提炼稳定结论、迁入新结构；
- 迁移动作以最小事务进行，不批量重排 `docs/`；
- 迁移过程中保留来源说明，必要时在原材料中标注迁移去向；
- **「真实迁移」的含义**：指 canonical 知识文件**实际移动到**新的 Domain / Area / Topic 结构，并更新必要链接；**不因为迁移而长期保留一份 V1 canonical 副本**。旧路径 / 旧版本由 **Git history** 保存；除非未来有明确需求，否则**不建立重复副本或复杂 redirect 系统**。
- `MIGRATION_BACKLOG.md` / `MIGRATION_CHECKLIST.md` 仍作为历史材料渐进迁移清单保留（与本轮结构升级是两条独立线）。

---

## 5. 极简 Front Matter 规范

知识 Markdown 使用**极简 Front Matter**，核心字段：

```yaml
---
id: <stable-id>
title: 文档标题
kind: <文档类型>
domain: <领域>
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

- 按需增加：`sources:`（来源引用）、`read_at:`（阅读/精读时间）等。
- **`kind` 只承担 Writing Rules 路由**，不决定目录结构。
- 首版 `kind` 采用**当前 MVP 的轻量受控集合**（以下 9 个足够首版使用）：
  - `learning-plan`（学习规划与进度）
  - `learning-note`（长期学习笔记）
  - `paper-summary`（受约束的论文快速总结，Agent 生成、未验证）
  - `paper-note`（用户精读后的论文笔记）
  - `topic-note`（主题/专题笔记）
  - `experiment`（实验记录）
  - `idea`（研究想法）
  - `report`（阶段报告/汇报）
  - `map`（认知结构地图 `_map.md`）
- **`kind` 为受控集合，非自由扩展**：若出现现有 `kind` 无法合理表达的真实需求，Agent **只能在聊天报告中提出新增建议**；新增 `kind` **须经用户确认并记录到 `DECISIONS.md`**，**不引入 schema YAML** 来管控取值。
- **`id` 是稳定身份标识，作为第一版主要身份机制**（不使用 uid / identity_key / 多层信任级联等重型身份体系，见 §15）。

---

## 6. 持续学习双文档 Living Document

持续学习保留「**学习规划与进度 + 长期学习笔记**」双文档 Living Document 模式（每个主题一个子目录，内含两份文档）。

- **`stable id` 作为第一版主要身份机制**：文档身份以稳定 `id` 为主，不引入五层身份信任级联。
- **异常防护**：对 Living Document 的更新中，若出现**异常大量删除**或**疑似截断**（如内容长度骤降、关键章节丢失、结构断裂），**不得自动覆盖**原文档；应暂停、向用户报告并等待确认，避免无声丢失用户既有知识。
- 增量更新优先补写/修订，而非整篇重写。

---

## 7. Source 与 Knowledge 分离（论文）

**Source 与 Knowledge 分离**：原始材料（论文 PDF、资料）与由此产出的知识文档分别管理。

- 论文 PDF **使用 Git LFS**，集中于 `sources/papers/`。
- 统一一个**轻量 `papers.yaml`** 记录每篇论文的：
  - 身份（稳定 key）；
  - arXiv / DOI；
  - 最新版本号；
  - `published_at`：论文**首次发布**时间；
  - `version_published_at`：当前所选版本的**版本发布**时间；
  - `added_at`：该论文被知识库**摄入**的时间；
  - PDF 路径；
  - `authors` / `venue` / `code` 等**可选信息**：仅在可靠获得时记录，**不扩展为复杂的 Source registry**。
- **同篇论文当前工作树只保留最新 PDF**；旧版本由 **Git history** 保存，不额外保留多份。
- **Source 原始文件不可被 Agent 改写**（论文 PDF、资料原文件保持只读）；但 Agent **可以读取 / 解析 Source、提取可靠 metadata**（如从 PDF 抽取标题、作者、arXiv / DOI、版本号），并**基于论文原文生成受约束的 Knowledge 文档或 `paper-summary`**（见 §8）。此读取 / 解析 / 生成能力不与「不加工原始内容」冲突——被保护的是 **Source 原始文件本身不被改写**，而非禁止 Agent 从中提取信息与产出 Knowledge。

---

## 8. 论文 Markdown 命名与 paper-summary

- 论文 Markdown 文件名以**正式论文标题**为语义来源，仅对 Windows 非法字符做**最小替换**（如 `\:*?"<>|` 等替换为安全字符），不做语义化 slug 重命名。
- **受约束的论文快速总结**：Agent 仅可对以下论文生成 `paper-summary`：
  - 用户**明确要求**；
  - 或该论文与 `STATE.md` 中**活跃主题高度明确相关**。
- 生成的 `paper-summary` **必须标记 `Agent-generated` / `user-unverified`**（明确未经验证）。
- **该声明是人类可读的谨慎标注**，仅用于提醒读者此文档尚未经用户精读验证；**这不意味着重新引入全局 `knowledge_status` 状态模型**（见 §15）。
- **同一文件随用户精读演化成 `paper-note`**：同一篇论文只维护一个 Markdown 文件，从 `paper-summary` 自然成长为 `paper-note`，**不得另建重复笔记**。

---

## 9. `_map.md` 认知结构约束

`_map.md` 是**人类认知结构的核心**，承载：

- 主题问题；
- 当前知识结构；
- 关键关系；
- 代表工作；
- 当前判断；
- 开放问题；
- 认知演化；
- 相关主题。

**章节按语义使用，而非强制填满所有章节**；允许 map 只覆盖当前真正存在的部分。

Agent 对 `_map.md` 的权限：

- ✅ 可自动维护已有结构中的**事实型、低风险内容**：明确导航信息、横向链接、代表工作清单等；
- ✅ 可补齐明确导航与横向链接；
- ❌ **不允许仅因为某章节为空就补写认知内容**（如「当前判断」「开放问题」「关键关系」等需要判断或理解的内容）；
- ❌ **未经用户确认，不得**：
  - 新建 / 拆分 / 合并重要知识节点；
  - 改变认知框架；
  - 重写「当前判断」；
  - 编造「认知演化」。
- **「认知演化」写入条件**：只有当**用户已经明确表达或确认认知变化**时，才能正式写入 `认知演化`；Git history / 既有记录**只能作为证据**，不能单独授权 Agent 推断用户认知变化；若 Agent 发现疑似认知转折，只能在**聊天报告中建议**，由用户决定是否写入。

---

## 10. Relation MVP

Relation MVP **只使用 Markdown 的「相关知识 / 相关主题」段落及自然语言解释**。

- **不实施**完整 relation ontology、受控关系词表、机器可读 graph、全局图谱。
- 横向关系通过文档内「相关主题 / 相关知识」列表 + 自然语言说明表达，辅以明确的链接（见 §11）。
- **普通知识文档不强制创建「相关知识 / 相关主题」章节**，不制造空章节；章节按语义使用（见 §9 的填空约束）。
- 对**已经明确存在的重要关系**，Agent 可以补充带有自然语言说明的**双向链接**；但**不得为了丰富图结构而自动推断弱关系**。

---

## 11. 主要找回机制

知识的主要找回路径为：

1. **GitHub Search**（代码/全文搜索）；
2. **README / HOME → Domain / Map**：从人类入口进入领域与认知地图；
3. **Knowledge + 明确的横向链接**：通过文档间链接跳转；
4. **Map 认知演化 / Git history**：通过认知地图的演化记录与 Git 历史追溯来龙去脉。

不依赖数据库索引或全局 graph 做主找回。

---

## 12. 仓库协议栈与人类入口

仓库把人类阅读入口与 Agent 控制面分开；根目录保留 `README.md` 与 `AGENTS.md`，其余协议文件集中在隐藏目录 `.kb/`：

| 文件 | 职责 |
|---|---|
| `README.md` | **人类 HOME**：项目入口、当前重点、目录导航 |
| `AGENTS.md` | **Agent 持久行为边界**：仅负责持久的行为边界、权限红线与 Git 安全约束；**不含写作规范**（写作规范归 `WRITING_RULES.md`）。该边界持久 / 稳定，但**经用户确认可演化** |
| `.kb/DECISIONS.md` | **已确认长期选择**：记录经过确认的方向性/架构性决策（持久 / 稳定，经用户确认可演化） |
| `.kb/WORKFLOW.md` | **每次 inbox curation 动作**：定义白天投放 → 夜间策展的具体步骤 |
| `.kb/WRITING_RULES.md` | **正式文档输出规范入口**：写作规则的唯一权威真源。当前 `README.md` / `AGENTS.md` 中的写作约定**仅作为未来建立本文件时的迁移来源**，不得继续作为并列规范真源 |
| `.kb/STATE.md` | **仅保存轻量当前状态**：当前阶段、活跃主题、持续维护文档、当前迁移重点 |

`.kb/STATE.md` 只承载**易变、轻量**的当前状态，不存放长期规则或知识正文。

---

## 13. Markdown 写作规则继承

Markdown 写作规则的**正式权威真源为 `WRITING_RULES.md`**（协议栈见 §12）。当前 `README.md` / `AGENTS.md` 中已有的写作约定**只作为未来建立 `WRITING_RULES.md` 时的迁移来源**，**不能继续作为与 `WRITING_RULES.md` 并列的规范真源**；在 `WRITING_RULES.md` 建立前，这些既有约定作为临时参考沿用，但任何新建的 `WRITING_RULES.md` 条款优先。继承的约定包括：

- **公式**：行内 `$...$`、块 `$$...$$`（兼容标准 Markdown 的 LaTeX）；
- **术语**：首次出现使用「中文名称（英文全称，必要时缩写）」，后文方用缩写；
- **长文档**：按既有规范维护目录与导航；
- **结论强度标记**：按语义使用 `[PAPER]`（论文作者报告）/ `[RESULT]`（已有结果）/ `[INFERENCE]`（推断）/ `[OPEN]`（开放/未验证），**而非机械地逐段添加**。

`.kb/WRITING_RULES.md` 的关键约束（写入该文件时的要求）：

- **按文档语义 / `kind` / 本次任务选择性应用**；
- **不得机械套模板**；
- **已有 canonical 结构优先**（尊重文档既有结构，不强行重排）；
- **禁止无关的大规模格式化**（不改变语义的重排版一律不做）。

---

## 14. Nightly 机制

- Nightly 由**外部 Agent 定时任务**触发，仓库**只提供 workflow**（即 `.kb/WORKFLOW.md` 中定义的可执行步骤），不内置复杂调度或状态机。
- 最终的触发 Prompt 应能缩短为：
  > 「按仓库 `AGENTS.md` / `.kb/WORKFLOW.md` 执行 nightly knowledge curation」
- Agent 可以在 **nightly branch** 上提交（commit）本次策展结果；
- **第一版绝不自动 merge main**：nightly 分支只作为待用户审阅的工作分支，合并动作由用户执行。

### 14.1 Nightly Agent 读取顺序

每次 nightly 触发时，Agent 按以下顺序读取协议栈：

1. `AGENTS.md`（行为边界、权限红线、Git 安全约束）；
2. `.kb/STATE.md`（当前阶段、活跃主题、持续维护文档、迁移重点）；
3. `.kb/DECISIONS.md`（已确认长期选择）；
4. `.kb/WORKFLOW.md`（本次 inbox curation 动作步骤）；
5. 扫描 `inbox/`；
6. 根据实际任务**按需读取** `.kb/WRITING_RULES.md`（文档输出规范）。

### 14.2 规则冲突优先级

当不同协议文件出现冲突时，按以下优先级裁定（从高到低）：

> **用户本次明确指令 > `AGENTS.md` > `.kb/DECISIONS.md` > `.kb/WORKFLOW.md` > `.kb/WRITING_RULES.md` > `.kb/STATE.md` 的运行时上下文**

- `.kb/STATE.md` 仅提供**运行时上下文**（当前阶段 / 主题 / 重点），**绝不能覆盖更高层的安全规则**（尤其 `AGENTS.md` 的权限红线与 Git 安全约束）。
- 用户在当次聊天中的明确指令优先级最高。

---

## 15. MVP 明确不实现的能力

以下能力在 MVP 中明确**不实现**（属旧完整架构探索，留作未来参考）：

- 数据库 / 向量库 / 检索引擎；
- Web UI；
- 复杂 schema YAML（重型元数据 schema）；
- 通用 sidecar 文件体系；
- 五层 identity trust（L1–L5 信任级联）；
- `document_state` / `knowledge_status` 双轴状态模型；
- temporal SLA / 复验调度；
- 复杂 review 子系统；
- 完整 relation ontology / global graph；
- 大量 derived indexes（派生索引）；
- `timeline` / `recent` 等聚合视图；
- Operational History 持久化层；
- `.curator` runtime state；
- 复杂 nightly 状态机；
- Python curator / `check_health` 脚本；
- 自动 merge（任何形式）；
- 一次性 V1 全量迁移。

---

## 16. 与旧架构文档的关系

| 文件 | 当前定位 |
|---|---|
| `.kb/history/V1_INVENTORY.md` | **事实审计基线**，保持不变，不因其后 MVP 而修改事实内容 |
| `.kb/history/V2_ARCHITECTURE_PROPOSAL.md` | 早期完整架构探索 / 未来参考；冲突以本文为准 |
| `.kb/history/V1_TO_V2_MAPPING.md` | 早期完整架构映射 / 未来参考；冲突以本文为准 |
| `.kb/history/MIGRATION_PLAN_V2.md` | 早期完整架构迁移计划；其 **M0–M7 为 superseded implementation proposal**，不再作为当前 roadmap |

本文是需求基线，**不要求**为迎合它而改写上述三篇历史设计正文——它们保留架构探索价值与决策演化证据。

---

## 17. 更新记录

- 2026-08-07：初始创建。将重新对齐后的轻量 V2 MVP 需求基线固化；对三篇旧设计文档做最小 authority 重定位（冲突以本文件为准，M0–M7 标记 superseded）。本轮仅做需求收敛，不进入实现。
- 2026-08-07（协议收尾）：对独立审查指出的 9 处歧义做最小修正——①Source 原始文件只读但允许读取/解析/提取 metadata 并产出 Knowledge/paper-summary；②`_map.md` 收紧为仅自动维护事实型低风险内容、认知演化须用户明确表达/确认、疑似转折仅建议；③AGENTS 只管持久行为边界/权限红线/Git 安全（不含写作规范）、WRITING_RULES 为唯一权威真源、README/AGENTS 既有约定仅作迁移来源、「长期不变」改「持久可演化」；④补入 Nightly 读取顺序与冲突优先级（STATE 仅上下文、不覆盖高层安全规则）；⑤Relation MVP 不强制空章节、不自动推断弱关系；⑥kind 改为轻量受控集合、新增须用户确认入 DECISIONS；⑦papers.yaml 区分 published_at/version_published_at/added_at、可选字段不扩 registry；⑧明确「真实迁移」= 实际移动 canonical 文件、不长期保留 V1 副本；⑨paper-summary 标注不重新引入 knowledge_status。本轮仍只改规格，未进入实现。
