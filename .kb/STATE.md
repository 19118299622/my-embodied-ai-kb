# STATE.md — 轻量运行时上下文

> 本文件只保存**易变、轻量**的当前运行上下文。
> 它**不存放长期规则、不存放知识正文、不复制 [`AGENTS.md`](../AGENTS.md) / [`DECISIONS.md`](./DECISIONS.md) 的内容**。
> 长期规则以 [`AGENTS.md`](../AGENTS.md) 与 [`DECISIONS.md`](./DECISIONS.md) 为准；本文件的运行时上下文**绝不能覆盖**其中的权限红线与 Git 安全约束。
> 无法可靠确认的内容不在此臆测。

---

## 当前阶段

- **V2 Canonical Knowledge Migration Phase 1 正在进行**：当前先迁移具身智能 → Robot Data → Data Quality 主干；Data Quality 第一批核心专题文档已完成 V1 → V2 canonical 迁移，后续主干按用户确认分批处理。
- **V2 MVP Repository Surface Cleanup 已完成**：人类阅读面与 Agent 内部控制面已分离。
- 已验证三条 MVP 能力链：
  - Living Document：V1 → V2 真实迁移；
  - Paper：Source → Registry → `paper-summary`。
  - Nightly Curator Pilot 1：Mixed Markdown Routing（手动触发，完成 Living Document / existing-topic Knowledge / ambiguous hold）。
- 尚未进入自动定时 Nightly Curator 或长期连续运行阶段；下一阶段由用户确认。

## 当前活跃主题

以下方向在仓库中仍有可见的持续推进（保守列举，来源于 [`RESEARCH_MAP.md`](../docs/RESEARCH_MAP.md) 与 `docs/` 现有文档）：

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
- [`docs/RESEARCH_MAP.md`](../docs/RESEARCH_MAP.md)（V1 legacy 全局研究地图；仍是有价值的当前研究主线快照）

> 其它 `docs/` 下的阅读计划 / 总结等材料按其在 V1 legacy 中的真实角色看待，不擅自定义为 canonical Living Document 一组；
> V2 的细粒度认知地图（`_map.md`）仅在某个 Topic 形成值得长期表达的独立认知结构时创建，届时在此补充。

## 当前迁移重点

- **分批主干迁移**：用户已明确确认先迁移 Data Quality 主树；本阶段仅处理可可靠归入该 Topic 的核心专题文档，不进行 V1 全量迁移。
- 已迁移的 Data Quality 主题文档进入 `kb/embodied-ai/robot-data/data-quality/`；相关 V1 论文笔记暂留 `docs/03_papers/`，等待 Source registry 与 PDF 边界单独处理。
- 强化学习基础双文档已完成 V1 → V2 真实迁移；未进入本阶段范围的 `docs/` 内容继续遵循「使用时迁移」原则。
