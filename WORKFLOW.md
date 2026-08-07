# WORKFLOW.md — Nightly Knowledge Curation 工作流

> 本文件定义 **Agent 每次 inbox curation 的可执行步骤**。它是“工作流说明”，不是调度脚本——Nightly 由**外部定时任务**触发，仓库不内置调度器或状态机（见 [`DECISIONS.md`](./DECISIONS.md) D-008）。
> 所有动作受 [`AGENTS.md`](./AGENTS.md) 权限红线与 [`V2_MVP_SPEC.md`](./V2_MVP_SPEC.md) 约束；写作细节以 [`WRITING_RULES.md`](./WRITING_RULES.md) 为准。
> Agent 在 **nightly 分支**提交，并**绝不自动 merge main**（[`DECISIONS.md`](./DECISIONS.md) D-007）。

---

## 1. 读取顺序（每次 run 开始）

按以下顺序加载协议栈，再处理 `inbox/`：

1. [`AGENTS.md`](./AGENTS.md) — 行为边界、权限红线、Git 安全约束；
2. [`STATE.md`](./STATE.md) — 当前阶段、活跃主题、迁移重点；
3. [`DECISIONS.md`](./DECISIONS.md) — 已确认长期选择；
4. [`WORKFLOW.md`](./WORKFLOW.md) — 本文件（本次动作步骤）；
5. 扫描 `inbox/`；
6. 根据实际任务**按需读取** [`WRITING_RULES.md`](./WRITING_RULES.md)。

## 2. 规则冲突优先级

> **用户本次明确指令 > [`AGENTS.md`](./AGENTS.md) > [`DECISIONS.md`](./DECISIONS.md) > 本文件（WORKFLOW.md） > [`WRITING_RULES.md`](./WRITING_RULES.md) > [`STATE.md`](./STATE.md) 的运行时上下文**

[`STATE.md`](./STATE.md) 仅提供上下文，不得覆盖更高层安全规则。

---

## 3. 一次 run 的步骤

### 3.1 检查 Git 状态

- 确认当前在 **nightly 分支**（若不存在则基于 `main` 新建，如 `nightly/YYYY-MM-DD`）；**不**在 `main` 上直接提交策展结果。
- 运行 `git status --short`，确认工作树没有未预期的改动；不覆盖用户未提交改动。
- **不 push 到 `main`、不 merge 任何分支、不做 `reset --hard` / `clean -fd` / 强制推送。**

### 3.2 扫描 `inbox/`

- 列出 `inbox/` 下所有条目（`.pdf` / `.md` / 其他）；忽略 `inbox/.gitkeep`。
- 对每条记录初始分类候选（见 §4）。

### 3.3 逐条分类与处理

对每条 `inbox/` 条目，判定为以下之一：

| 类别 | 判定 | 处理 |
|---|---|---|
| **Source** | 原始材料（论文 PDF、资料文件） | 登记到 Source 层（§3.7） |
| **新 Knowledge** | 可明确归类的新 Markdown 知识 | 归入正确知识位置（§3.5） |
| **Living Document update** | 明显对应已有文档的增量补充 / 修订 | 安全增量更新（§3.6） |
| **已有 Source 新版本** | 与 `sources/papers.yaml` 中某篇同篇、且版本更高 | Source version update（§3.7） |
| **Ambiguous** | 来源 / 主题 / 语义无法可靠判断 | **留在 `inbox/`**（§3.9） |

### 3.4 查已有知识与 Map

- 用 **GitHub Search / 全文搜索** 与 **README → Domain / Map** 查找是否已有相关文档或 `_map.md`；
- 判断该材料与现有知识的关系（补写 / 新建 / 更新）；
- 找不到可靠归属时，归入 Ambiguous。

### 3.5 何时允许移动 / 归档

- **允许**：材料语义清晰、可归入既有的明确主题或新建一个低风险的明确主题；
- **移动后**：更新必要导航 / 链接（如对应 `_map.md` 的“代表工作 / 相关主题”）；
- **Front Matter**：按 [`WRITING_RULES.md`](./WRITING_RULES.md) 填写极简字段（`id` / `title` / `kind` / `domain` / `created` / `updated`）。

### 3.6 安全处理 Living Document

- **增量优先**：补写 / 修订，而非整篇重写；
- **异常防护**：若出现**异常大量删除、内容长度骤降、关键章节丢失或结构断裂**，**不得自动覆盖**原文档——暂停、在报告中说明、等待用户确认；
- 双文档学习模式（学习规划 + 长期学习笔记）保持各自职责，重要问答 / 误区修正沉淀进长期学习笔记。

### 3.7 处理论文 PDF 与 `papers.yaml`

- 论文 PDF **只读**：原始 PDF 文件**不可被内容级改写**；
- PDF 放入 `sources/papers/`，由 `.gitattributes` 自动走 **Git LFS**；
- 在 `sources/papers.yaml` 登记：稳定 `source id`、正式 `title`、arXiv / DOI（有则）、`latest version`、`published_at`、`version_published_at`、`added_at`、`pdf` 路径，以及可靠获得时的 `authors` / `venue` / `code`；
- **Source version update（唯一允许的 PDF 替换）**：仅当**可靠确认**「同一篇论文且新版本高于当前」时，用新官方 Source 文件替换当前 canonical PDF，并更新 `papers.yaml` 的版本与时间字段；旧版本由 Git history 保存；
- **无法可靠确认同篇 / 版本时，绝不替换**当前 PDF，材料留在 `inbox/` 或作为新条目登记并标注待确认。

### 3.8 生成 `paper-summary`（受约束）

仅当满足以下之一才可生成：

- 用户**明确要求**；或
- 该论文与 [`STATE.md`](./STATE.md) 中**活跃主题高度明确相关**。

要求：

- 文件名以**正式论文标题**为语义来源，仅对 Windows 非法字符做最小替换；
- **必须标记 `Agent-generated` / `user-unverified`**；
- 同一篇论文只维护一个 Markdown 文件，未来随用户精读自然演化为 `paper-note`（[`DECISIONS.md`](./DECISIONS.md) D-010）；
- 该标注只是人类可读谨慎提醒，**不引入全局 `knowledge_status` 状态模型**。

### 3.9 维护已有 Map / 明确双向链接

- 仅对**已经明确存在的重要关系**补充带自然语言说明的双向链接；
- 可自动维护已有 `_map.md` 中的**事实型低风险内容**（导航、横向链接、代表工作清单）；
- **不**仅因章节为空就补写「当前判断 / 开放问题 / 关键关系 / 认知演化」；
- **不**为丰富图结构而自动推断弱关系；认知演化写入须用户明确表达或确认（[`DECISIONS.md`](./DECISIONS.md) D-005）。

### 3.10 最终 diff 检查

- `git diff`（含 staged）确认本次改动：
  - 范围小而清晰；
  - 无“统一格式”驱动的无关大规模重排版；
  - 无用户既有结论被无声覆盖；
  - 无 Agent 推断被写成用户判断。
- 若发现异常截断 / 大规模删除，回到 §3.6 暂停并报告。

### 3.11 提交与报告

- 在 nightly 分支 `git commit`（描述性 message，如 `kb: curate <topic> from inbox` / `sources: register <paper>`）；
- **不 push 到 `main`、不 merge**；是否 push nightly 分支 / 合并由用户决定；
- 在聊天窗口报告：本次处理了哪些条目、归入哪里、生成 / 更新了什么、哪些留在 `inbox/` 及原因、哪些需要用户决策。

---

## 4. 分类决策要点

- **能可靠归类 → 移动 / 归档**；**不能 → 留 `inbox/`**。
- **新主题**：仅在语义清晰、且与现有主题确有区分时新建；不为了“看起来完整”拆分。
- **已有文档更新**：优先增量补写；明显对应某文档的 material 才视为 Living Document update。
- **论文**：先判 Source 登记，再判是否需要 `paper-summary`（§3.8）。

## 5. 红线回顾（不可逾越）

- 永不自动 merge main；
- 原始 Source 文件不可内容级改写；
- 不删除 / 覆盖用户认知；不把推断冒充用户判断；
- 不确定 → 保留 `inbox/` + 聊天报告。

## 6. 本轮范围

本文件只定义工作流；**不实现**调度脚本、状态机、Python curator 或任何自动触发逻辑。真实 nightly 自动化留待后续阶段。
