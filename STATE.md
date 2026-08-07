# STATE.md — 轻量运行时上下文

> 本文件只保存**易变、轻量**的当前运行上下文。
> 它**不存放长期规则、不存放知识正文、不复制 [`AGENTS.md`](./AGENTS.md) / [`DECISIONS.md`](./DECISIONS.md) 的内容**。
> 长期规则以 [`AGENTS.md`](./AGENTS.md) 与 [`DECISIONS.md`](./DECISIONS.md) 为准；本文件的运行时上下文**绝不能覆盖**其中的权限红线与 Git 安全约束。
> 无法可靠确认的内容不在此臆测。

---

## 当前阶段

- **V2 MVP — Protocol Foundation（协议基础）**
- 本轮已建立最小仓库协议与 intake / source 骨架；**即将进入 pilot**：选取实际内容，按 `Domain → Area → Topic` 自然生成首个 `kb/` 知识目录与对应 `_map.md`。
- 尚未进入：知识迁移执行、完整 Nightly 自动化、任何复杂基础设施。

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

- `docs/01_foundations/reinforcement_learning/`
  - `学习规划与进度.md` + `长期学习笔记.md`（双文档学习模式实例）
- `docs/02_topics/embodied_data_quality/`
  - `分层阅读计划.md` + `L1核心精读总结.md`（专题阅读双文档）
- `RESEARCH_MAP.md`（V1 legacy 全局研究地图；V2 细粒度 `_map.md` 尚未生成）

> V2 的细粒度认知地图（`_map.md`）将在 `kb/` pilot 时按领域创建，届时在此补充。

## 当前迁移重点

- **V1 使用时迁移**：`docs/` 为 V1 legacy 知识，仅在被真实引用或用户明确要求时才迁入新结构；不强制全量迁移。
- 本轮未迁移任何 `docs/` 内容；未注入 `stable id`；未改动已有知识正文。
