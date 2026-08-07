# STATE.md — 轻量运行时上下文

> 本文件只保存**易变、轻量**的当前运行上下文。
> 它**不存放长期规则、不存放知识正文、不复制 [`AGENTS.md`](./AGENTS.md) / [`DECISIONS.md`](./DECISIONS.md) 的内容**。
> 长期规则以 [`AGENTS.md`](./AGENTS.md) 与 [`DECISIONS.md`](./DECISIONS.md) 为准；本文件的运行时上下文**绝不能覆盖**其中的权限红线与 Git 安全约束。
> 无法可靠确认的内容不在此臆测。

---

## 当前阶段

- **V2 MVP — Pilot 2：论文 Source → Registry → paper-summary 已完成**
- 本 Pilot 验证了完整的「用户投放 PDF 到 `inbox/` → Agent 按协议策展」日常链路：从 PDF 原文 + 可靠公开来源核验论文身份 / 稳定外部身份（arXiv ID、DOI）/ 版本 / 作者并确认其为最新公开版本；将 PDF **真实移动**（非复制）至 `sources/papers/` 并经 Git LFS 管理，文件名以正式标题为语义来源、仅对 Windows 非法字符做最小替换；在 `sources/papers.yaml` **新增且仅新增**一条 Source registry（稳定 source id `arxiv-2602.19313`）；生成一篇受约束 `paper-summary`（Front Matter `sources:` 引用该 source id，标记 `Agent-generated` / `user-unverified`，不写 `read_at`）。
- 唯一处理论文：**TOPReward: Token Probabilities as Hidden Zero-Shot Rewards for Robotics**（arXiv:2602.19313，最新 v2）。本 Pilot 显式授权建立了 `kb/machine-learning/reinforcement-learning/reward-design/` Subtopic 承载该 `paper-summary`；未创建其它 Domain / Area / Topic，未创建 `_map.md`。
- **`_map.md` 不是目录模板**：只有当某个 Topic 已形成值得长期表达的独立认知结构时，才在该目录创建 `_map.md`；本 Pilot 不创建地图。
- 尚未进入：其它 `docs/` 知识迁移、完整 Nightly 自动化、任何复杂基础设施、除本次显式授权外的结构扩张。
- **下一步**：等待用户确认本次结果，准备下一 Pilot（下一 Pilot 的具体范围由用户确认，不在此预定义）。

## 当前活跃主题

以下方向在仓库中仍有可见的持续推进（保守列举，来源于 [`RESEARCH_MAP.md`](./RESEARCH_MAP.md) 与 `docs/` 现有文档）：

- 实机机器人数据质量评估；
- 视觉—语言—动作模型（VLA）训练、后训练与实时控制执行验证；
- 世界模型；
- 强化学习与 VLA 后训练；
- 跨具身 VLA；
- 执行验证与可靠性。

> 具身智能是深度主线；新领域（机器学习、计算机系统、Linux、Git / 软件工程、数学、研究方法、Agent Engineering 等）将在实际出现时进入。

## 当前 Living Documents（可靠确认）

仅列出能从现有仓库**可靠确认**的 Living Document：

- `kb/machine-learning/reinforcement-learning/`
  - `学习规划与进度.md`（id: `rl-basics-plan`）+ `长期学习笔记.md`（id: `rl-basics-note`）：双文档学习模式实例，持续 canonical 维护（由 V1 真实迁移）
- `RESEARCH_MAP.md`（V1 legacy 全局研究地图；仍是有价值的当前研究主线快照）

> 其它 `docs/` 下的阅读计划 / 总结等材料按其在 V1 legacy 中的真实角色看待，不擅自定义为 canonical Living Document 一组；
> V2 的细粒度认知地图（`_map.md`）仅在某个 Topic 形成值得长期表达的独立认知结构时创建，届时在此补充。

## 当前迁移重点

- **V1 使用时迁移**：`docs/` 为 V1 legacy 知识，仅在被真实引用或用户明确要求时才迁入新结构；不强制全量迁移。
- 本轮已真实迁移强化学习基础双文档（`docs/01_foundations/reinforcement_learning/` → `kb/machine-learning/reinforcement-learning/`）；其余 `docs/` 内容仍按「使用时迁移」原则；本轮未注入额外 stable id、未改动其它知识正文。
