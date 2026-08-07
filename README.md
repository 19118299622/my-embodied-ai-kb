# my-embodied-ai-kb

> 中文定位：**具身智能研究实验室**
>
> 这是一个持续演化的个人研究项目，用于组织具身智能、机器人学习、视觉—语言—动作模型、世界模型、强化学习和数据质量评估相关的学习、调研、实验与研究想法。

## 项目定位

本项目**不是一份已经完整收录过去全部知识的“具身智能知识库”**，也不要求在启用时一次性迁移所有历史材料。

它更接近一个长期运行的个人研究环境：

- 过去知识按需要逐步迁移；
- 当前研究形成结构化专题；
- 新论文优先建立独立笔记；
- 相互关联的工作再沉淀为专题综述；
- 实验、失败记录和研究想法持续回流；
- Agent 依据 [`AGENTS.md`](./AGENTS.md) 协助维护。

当前内容只是项目的初始版本，而不是对个人既有知识的完整声明。

## 当前入口

| 文件 | 作用 |
|---|---|
| [`RESEARCH_MAP.md`](./RESEARCH_MAP.md) | 当前研究版图、主线关系与开放问题 |
| [`MIGRATION_BACKLOG.md`](./MIGRATION_BACKLOG.md) | 历史学习材料的渐进迁移清单 |
| [`AGENTS.md`](./AGENTS.md) | Agent 的维护规则、写作规范与 Git 约束 |
| [`CHANGELOG.md`](./CHANGELOG.md) | 项目结构和重要内容变更记录 |
| [`docs/02_topics/README.md`](./docs/02_topics/README.md) | 专题文档索引 |
| [`docs/02_topics/vla_runtime_verification_online_learning/2026-07-31_VLA实时控制执行验证与在线学习.md`](./docs/02_topics/vla_runtime_verification_online_learning/2026-07-31_VLA实时控制执行验证与在线学习.md) | 当前首个前沿专题观察 |

## 目录结构

```text
my-embodied-ai-kb/
├── README.md
├── AGENTS.md
├── RESEARCH_MAP.md
├── MIGRATION_BACKLOG.md
├── CHANGELOG.md
├── .gitignore
├── .gitattributes
│
├── docs/
│   ├── 01_foundations/    # 系统学习与基础理论
│   ├── 02_topics/         # 跨论文专题综述与研究主线
│   ├── 03_papers/         # 单篇论文阅读笔记
│   ├── 04_experiments/    # 实验方案、复现、结果与失败记录
│   ├── 05_ideas/          # 未验证想法、研究问题与假设
│   └── 06_reports/        # 阶段报告、汇报材料与对外交付
│
├── templates/             # 统一文档模板
└── assets/                # 图片、图表和小型附件说明
```

## 文档分工

### `01_foundations`

用于长期、系统性学习。默认采用“双文档学习模式”：

1. 学习规划与进度；
2. 长期学习笔记。

每一讲结束后先进入问答区，重要澄清再回写笔记。

### `02_topics`

用于跨论文、跨实验的专题综合。专题文档回答：

- 这个方向解决什么问题？
- 方法之间如何演进？
- 共同假设和主要分歧是什么？
- 哪些结论已经较稳固？
- 哪些问题仍然开放？
- 对当前研究有什么可执行的启发？

### `03_papers`

用于单篇论文阅读笔记。论文笔记优先保留：

- 核心问题；
- 方法与数学形式；
- 实验是否真正支撑结论；
- 局限与失败模式；
- 与已有工作的关系；
- 对当前研究的参考价值。

### `04_experiments`

用于可复现实验记录，包括：

- 实验问题；
- 假设；
- 数据与环境；
- 评价指标；
- 实验配置；
- 结果；
- 失败原因；
- 下一步。

### `05_ideas`

用于保存尚未成熟的想法。允许零散，但必须逐步区分：

- 合理洞见；
- 隐含前提；
- 可能反例；
- 可验证预测；
- 最小实验。

### `06_reports`

用于阶段汇报、调研报告与对外交付。这里可以引用其他目录内容，但不应成为唯一知识来源。

## 新内容放在哪里

优先级如下：

1. **新的论文**：先进入 `docs/03_papers/`；
2. **新的研究问题或方法类别**：新建 `docs/02_topics/` 专题；
3. **与既有专题高度连续的补充**：更新对应专题；
4. **尚未验证的个人判断**：进入 `docs/05_ideas/`；
5. **实际运行、复现或消融**：进入 `docs/04_experiments/`；
6. **旧材料迁移**：依据 [`MIGRATION_BACKLOG.md`](./MIGRATION_BACKLOG.md) 渐进处理。

默认原则：

> **优先新增独立文档，次选扩写已有文档。**

不要为了“看起来完整”而把多个方向压缩到一个超长文件中。

## 文档状态

建议在 Markdown 文件顶部使用 YAML 元数据：

```yaml
---
title: 文档标题
type: paper-note | topic | foundation | experiment | idea | report
status: seed | active | stable | archived
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags:
  - embodied-ai
---
```

状态含义：

- `seed`：只有初始框架或零散内容；
- `active`：正在持续维护；
- `stable`：结构和主要结论已相对稳定；
- `archived`：保留历史，但不再作为当前结论。

## 文件命名

### 长期维护文档

不使用日期前缀：

```text
动作块执行与闭环反馈.md
世界模型数据协议.md
```

### 阶段性观察与报告

使用日期前缀：

```text
2026-07-31_VLA实时控制执行验证与在线学习.md
```

### 单篇论文笔记

优先使用稳定短标题：

```text
CheckVLA_阅读笔记.md
Robot_Data_Curation_with_Mutual_Information_Estimators_阅读笔记.md
```

### 目录放置约定

- **`01_foundations`**：保持「双文档学习模式」——每个主题一个子目录，内含 `学习规划与进度.md` + `长期学习笔记.md`。
- **`02_topics`**：每个专题一个子目录（英文 slug）；主文档为长期维护的总览时用**稳定名、无日期**（如 `实机机器人数据质量评估总览.md`），若为阶段性快照/观察则用 `YYYY-MM-DD_` 前缀；专题配图放 `assets/<topic-slug>/`。
- **`03_papers`**：每篇论文一个子目录（英文 slug），内含 `<slug>_阅读笔记.md` + 同名 PDF（经 Git LFS 管理）+ 必要配图。
- **`04_experiments` / `05_ideas` / `06_reports`**：单文件即可，按上述日期/稳定名规则命名。

### 文档状态初始值

- 已带完整内容且经核验 → `active`；
- 仅骨架、待复验或迁移中 → `seed`。

### 标签受控词表

`tags` 从以下白名单选取，新增需先确认：

```text
embodied-ai, vla, world-model, reinforcement-learning, imitation-learning,
data-quality, robot-data, runtime-verification, foundation, survey,
experiment, research-idea, learning-plan, learning-notes
```

## 公式和图表

行内公式：

```latex
$Q(s,a)$
```

块公式：

```latex
$$
Q(s,a)=r+\gamma\max_{a'}Q(s',a')
$$
```

流程图优先使用 Mermaid。涉及复杂图像、实验结果图或论文原图时，放入 `assets/` 并使用相对路径引用。

## 历史材料迁移策略

不建议一次性搬运所有旧笔记。采用“使用时迁移”：

1. 当前研究引用到旧知识；
2. 检查旧材料是否仍然准确；
3. 提炼稳定结论；
4. 迁入合适目录；
5. 在原材料中保留迁移说明或来源；
6. 更新 `MIGRATION_BACKLOG.md`。

这样可以避免把过时、重复或上下文不完整的内容直接复制进新项目。

## Git 初始化

将文件夹放到本地目标位置后：

```bash
cd my-embodied-ai-kb
git init
git add .
git commit -m "chore: initialize my-embodied-ai-kb"
git branch -M main
```

创建远程仓库后：

```bash
git remote add origin <YOUR_REMOTE_URL>
git push -u origin main
```

建议先保持私有仓库。论文 PDF、模型权重、数据集和大型图片不要直接提交到普通 Git 历史；确实需要版本控制时使用 Git LFS 或外部存储。

## 推荐提交格式

```text
docs: add CheckVLA paper note
docs: update VLA runtime verification topic
research: add action-consequence consistency hypothesis
experiment: record world-model data scoring ablation
chore: reorganize topic index
```

## 当前版本

- 项目状态：`seed`
- 初始化日期：2026-07-31
- 当前重点：
  1. 实机机器人数据质量评估；
  2. 世界模型辅助数据分析与执行验证；
  3. 视觉—语言—动作模型后训练；
  4. 机器人强化学习与数据闭环。
