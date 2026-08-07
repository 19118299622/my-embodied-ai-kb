# my-embodied-ai-kb

> **个人技术知识库（Personal Technical Knowledge Base）**
>
> 一个长期运行、Agent-first、Git-native 的个人技术知识沉淀系统。
> 当前以**具身智能 / 机器人**为深度主线，但知识域可扩展到机器学习、计算机系统、Linux、Git / 软件工程、数学、研究方法、Agent Engineering 等。

## 这是什么

这不是一份已经完整收录过去全部知识的“知识库”，也不要求在启用时一次性迁移所有历史材料。它更接近一个持续运行的知识环境：

- 用户白天把 `.pdf` / `.md` 放进 `inbox/`；
- Agent 按仓库自身协议完成 Source 登记、Knowledge 归档、Living Document 安全更新、受约束的论文快速总结、已有 `_map.md` 与明确链接维护；
- Agent 在 nightly 分支提交，并在聊天窗口报告；
- 无法可靠判断的材料原样留在 `inbox/`，由用户后续处理。

知识随真实使用逐步积累与演化，而非一次性搬运。

## 当前重点

仓库当前仍在持续维护的主线（保守列举，详见 [`RESEARCH_MAP.md`](./RESEARCH_MAP.md)）：

1. 实机机器人数据质量评估；
2. 视觉—语言—动作模型（VLA）训练、后训练与实时控制执行验证；
3. 世界模型；
4. 强化学习与 VLA 后训练；
5. 执行验证与可靠性；
6. 数据生成与跨具身迁移。

> 具身智能是当前的深度主线，但不是唯一主题。新领域会在实际出现时自然进入知识库。

## 主要入口

### 协议栈（Agent 与用户共同遵守的根目录约定）

| 文件 | 职责 |
|---|---|
| [`AGENTS.md`](./AGENTS.md) | Agent 持久行为边界、权限红线、Git 安全约束 |
| [`DECISIONS.md`](./DECISIONS.md) | 已确认的长期方向性 / 架构性决策 |
| [`STATE.md`](./STATE.md) | 轻量运行时上下文：当前阶段、活跃主题、迁移重点 |
| [`WORKFLOW.md`](./WORKFLOW.md) | 每次 inbox curation 的具体步骤 |
| [`WRITING_RULES.md`](./WRITING_RULES.md) | 文档输出规范（唯一权威真源） |
| 本文 `README.md` | 人类 HOME：项目入口、当前重点、导航 |

冲突时以 [`AGENTS.md`](./AGENTS.md) 中的权限红线为最高安全约束；完整优先级见 [`WORKFLOW.md`](./WORKFLOW.md)。

### 知识入口

- **`docs/` —— V1 legacy 知识（渐进迁移中）**
  现有大量已沉淀的论文笔记、专题、实验与学习文档仍在 `docs/` 下，采用旧的 `01_foundations / 02_topics / 03_papers / 04_experiments / 05_ideas / 06_reports` 文档类型目录结构。**这只是 V1 遗留结构，不是 V2 目标结构。** 这些材料按「使用时迁移」原则，在未来被真实引用或用户要求时才迁入新的 V2 结构。
- **`RESEARCH_MAP.md` —— 当前研究版图快照（V1 legacy 全局地图）**
  仍是有价值的当前研究主线、专题关系与开放问题快照；V2 的细粒度认知地图（`_map.md`）是**内容驱动**而非目录模板：只有当某个 Topic 已形成值得长期表达的独立认知结构时才创建，**并非每个领域都必然创建**。
- **V2 新知识结构（尚未生成）**
  新的知识将采用 **Domain → Area → Topic → 必要时 Subtopic** 的领域 / 问题优先结构（详见 [`V2_MVP_SPEC.md`](./V2_MVP_SPEC.md)）。真正的 `kb/` Topic 目录留到下一轮选取实际 pilot 内容时按内容自然生成，**本轮刻意不创建空骨架**。

### 投放与来源

- **`inbox/`** —— 用户白天手动投放 `.pdf` / `.md` 的入口；Agent 在 nightly curation 中处理。
- **`sources/papers/` + `sources/papers.yaml`** —— Source 层：论文 PDF（经 Git LFS 管理）与轻量身份 / 版本 registry。原始 Source 文件只读，不被改写。

## 简短使用说明

1. 把新的论文 PDF 或 Markdown 笔记丢进 `inbox/`；
2. Agent 按 [`WORKFLOW.md`](./WORKFLOW.md) 在 nightly 分支完成策展，并在聊天中报告；
3. 你审阅 nightly 分支、确认后合并；
4. **不要把 Agent 当作知识结构的最终决策者**——Agent 只在协议约束内整理、提议与执行，知识的归类、结构取舍与最终采纳始终由你决定；不确定时材料留在 `inbox/`，由你后续处理。

PDF、大型图片等二进制走 Git LFS（`assets/` 与 `sources/` 中的 `*.pdf` / `*.pptx` 已由 `.gitattributes` 配置）。

---

旧 V1 架构探索与早期设计详见 [`V1_INVENTORY.md`](./V1_INVENTORY.md)、[`V2_ARCHITECTURE_PROPOSAL.md`](./V2_ARCHITECTURE_PROPOSAL.md)、[`V1_TO_V2_MAPPING.md`](./V1_TO_V2_MAPPING.md)、[`MIGRATION_PLAN_V2.md`](./MIGRATION_PLAN_V2.md)；当前权威需求基线为 [`V2_MVP_SPEC.md`](./V2_MVP_SPEC.md)。
