# DECISIONS.md — 已确认的长期决策

> 本文件只固化 **已经在 [`V2_MVP_SPEC.md`](./V2_MVP_SPEC.md) 中真正确认**的方向性 / 架构性长期决策。
> 它不是需求规格的全文复制，也不是实现细节清单。新增决策须经用户确认后在此追加（同时更新下方编号）。
> 任何未在此确认、且不属于上述规格的内容，都不得被当作“已决定的方向”。

---

## D-001 · 知识目录采用领域 / 问题优先结构

- **decision**：知识目录使用 `Domain → Area → Topic → 必要时 Subtopic` 的领域 / 问题优先结构；文档类型仅通过 Front Matter 的 `kind` 作为元数据，不决定目录层级。
- **rationale**：按主题与问题组织比按「论文 / 实验 / 想法」等文档类型更贴合长期个人认知；文档类型用 `kind` 标记即可。
- **status**：confirmed

## D-002 · V1 采用「使用时真实迁移」

- **decision**：旧 V1 内容仅在被当前研究真实引用或用户明确要求时才迁移；「真实迁移」= canonical 知识文件实际移动到新结构并更新必要链接，**不长期保留 V1 canonical 副本**，旧路径 / 旧版本由 Git history 保存；不明确需求时不建重复副本或复杂 redirect 系统。
- **rationale**：避免一次性全量搬运过时、重复或上下文不全的内容，降低迁移风险。
- **status**：confirmed

## D-003 · Source 与 Knowledge 分离 + 轻量 `papers.yaml`

- **decision**：原始 Source（论文 PDF 等）与产出的 Knowledge 文档分离；PDF 走 Git LFS 集中于 `sources/papers/`；用轻量 `papers.yaml` 记录身份 / 版本 / 时间 / 路径等，**不建 per-paper sidecar、不扩展为复杂 Source registry**。
- **rationale**：既保住原始证据可读性，又让知识文档独立于二进制；单一轻量 registry 足够。
- **status**：confirmed

## D-004 · `stable id` 作为第一版主要身份机制

- **decision**：知识文档以稳定 `id` 字段为主要身份标识；**不引入** uid / identity_key / 多层信任级联等重型身份体系。
- **rationale**：MVP 阶段身份需求简单，重型身份体系收益低、复杂度高。
- **status**：confirmed

## D-005 · `_map.md` 权限边界（认知演化须用户确认）

- **decision**：`_map.md` 是人类认知结构核心；Agent 仅可维护事实型低风险内容（导航、横向链接、代表工作清单）；「认知演化」「当前判断」等只有在**用户明确表达或确认认知变化**时才写入；Git history / 既有记录仅作证据，不能单独授权 Agent 推断；疑似认知转折只能在聊天报告中建议。
- **rationale**：防止 Agent 把推断冒充用户认知、无声篡改认知结构。
- **status**：confirmed

## D-006 · Relation 仅用 Markdown 自然语言（无 ontology / graph）

- **decision**：横向关系仅用文档内「相关主题 / 相关知识」+ 自然语言说明 + 明确链接表达；普通文档不强制空章节，不为了丰富图结构自动推断弱关系。
- **rationale**：MVP 不实现完整 relation ontology / global graph，降低维护负担。
- **status**：confirmed

## D-007 · 永不自动 merge main

- **decision**：nightly 分支只作为待用户审阅的工作分支；**第一版绝不自动合并 main**，合并动作由用户执行。
- **rationale**：知识变更需人类把关，避免无声覆盖用户既有内容。
- **status**：confirmed

## D-008 · Agent-first，Automation-later

- **decision**：知识策展由 Agent 按仓库协议执行；Nightly 由**外部定时任务**触发，仓库只提供 workflow（[`WORKFLOW.md`](./WORKFLOW.md)），不内置复杂调度 / 状态机；最终触发 Prompt 可缩短为「按 `AGENTS.md` / `WORKFLOW.md` 执行 nightly knowledge curation」。
- **rationale**：先把协议与人类协作跑通，自动化调度后置。
- **status**：confirmed

## D-009 · 论文只保留最新 Source 版本

- **decision**：同篇论文当前工作树只保留最新 PDF；旧版本由 Git history 保存；若**可靠确认**「同一论文且新版本更高」，可用新官方 Source 文件替换当前 canonical PDF 并更新 `papers.yaml`（Source version update）；无法可靠确认同篇 / 版本时**不得替换**。
- **rationale**：避免工作树堆积多份旧 PDF，同时保留历史版本可追溯。
- **status**：confirmed

## D-010 · `paper-summary` → `paper-note` 单文件演化

- **decision**：同一篇论文只维护一个 Markdown 文件，从 `paper-summary` 自然成长为 `paper-note`；**不另建重复笔记**。
- **rationale**：避免同一论文多份笔记分裂、结论不一致。
- **status**：confirmed

## D-011 · `kind` 为轻量受控集合（不引入 schema YAML）

- **decision**：`kind` 采用当前 MVP 受控集合（learning-plan / learning-note / paper-summary / paper-note / topic-note / experiment / idea / report / map）；新增 `kind` 须**用户确认并记录到本文件**，不引入 schema YAML 管控取值。
- **rationale**：现有 9 个足够首版使用，避免重型 schema。
- **status**：confirmed

## D-012 · inbox-first intake（用户投放，Agent 策展）

- **decision**：用户白天把 `.pdf` / `.md` 放入 `inbox/`，Agent 按 [`WORKFLOW.md`](./WORKFLOW.md) 夜间策展；无法可靠判断的材料原样留在 `inbox/`。
- **rationale**：明确人机分工，降低误操作与无声丢失。
- **status**：confirmed
