---
title: V1 仓库审计清单
type: project
status: active
created: 2026-08-07
updated: 2026-08-07
tags:
  - project-maintenance
  - architecture
  - v2-design
---

# V1_INVENTORY — 现状审计（Phase 0）

> 本文件是 V1 → V2 架构升级的 **Phase 0 产出**：对 `my-embodied-ai-kb` 当前状态的实测记录。
>
> 所有数字来自 2026-08-07 对工作区 `D:\materials\VLA\my-embodied-ai-kb` 的实际扫描（文件遍历 / front matter 解析 / 链接解析 / `git` 查询），不是根据文档描述推断。
>
> 配套文档：[`V2_ARCHITECTURE_PROPOSAL.md`](./V2_ARCHITECTURE_PROPOSAL.md)、[`V1_TO_V2_MAPPING.md`](./V1_TO_V2_MAPPING.md)、[`MIGRATION_PLAN_V2.md`](./MIGRATION_PLAN_V2.md)。

---

## 1. 审计口径

| 项 | 值 |
|---|---|
| 审计日期 | 2026-08-07 |
| 审计对象 | 本地工作区 + Git 仓库 `origin = github.com/19118299622/my-embodied-ai-kb` |
| 分支 | `main`（唯一分支），`HEAD = 46f4768`，与 `origin/main` 同步 |
| 工作区状态 | `git status --short` 为空（clean） |
| 排除范围 | `.git/`、`.workbuddy/`（后者已在 `.gitignore` 中，属项目记忆，不属知识库） |

---

## 2. 结构与规模实测

### 2.1 文件统计

| 指标 | 数量 |
|---|---:|
| Git 追踪文件总数 | 85 |
| Markdown | 74 |
| PDF（Git LFS） | 5 |
| PPTX（Git LFS） | 4 |
| 配置文件（`.gitignore` / `.gitattributes`） | 2 |
| `docs/` 下 Markdown 正文体量 | 约 678 KB（UTF-8 字节） |
| 目录层级最深 | 3 层（`docs/<层>/<簇>/<文件>`） |

### 2.2 目录布局（实际）

```text
my-embodied-ai-kb/
├── README.md              AGENTS.md          RESEARCH_MAP.md
├── CHANGELOG.md           MIGRATION_BACKLOG.md
├── MIGRATION_CHECKLIST.md PROJECT_INVENTORY.md
├── .gitignore             .gitattributes
├── assets/README.md                          （仅说明，无实际资产）
├── templates/             7 个模板
└── docs/
    ├── 01_foundations/    README + 1 个学习簇（reinforcement_learning，2 文件）
    ├── 02_topics/         README + 5 个专题簇（9 文件）
    ├── 03_papers/         README + 20 个论文簇（44 文件，含 5 PDF）
    ├── 04_experiments/    README + 1 个实验
    ├── 05_ideas/          README（**空目录，无任何内容文档**）
    └── 06_reports/        README + 1 个汇报簇（4 PPTX）
```

### 2.3 Git 历史深度（关键发现）

| 指标 | 值 |
|---|---|
| 提交总数 | **2** |
| `1f37b50 initial commit` | 22 个文件 |
| `46f4768 docs: publish ...` | 77 个文件变更，+11316 行 |
| 任意知识文档的独立提交次数 | **≤ 1** |

**含义**：AGENTS.md §9 规定「除非用户明确要求，否则不 commit」，实践中被严格执行——结果是 **Git 目前不构成有效的文档版本史**。所有知识几乎是在一次提交中同时落地的。因此 V2 设计中「Git history 保存 living document 的历史版本」这一假设，在当前仓库上 **尚不成立**，必须由 V2 的提交粒度策略来重新建立。

---

## 3. 元数据现状实测

### 3.1 front matter 覆盖率

| 类别 | 数量 |
|---|---:|
| 含 YAML front matter | 63 / 74 |
| 无 front matter | 11 / 74 |

无 front matter 的 11 个文件全部是**治理层 / 导航层文档**：`README.md`、`AGENTS.md`、`CHANGELOG.md`、`PROJECT_INVENTORY.md`、`assets/README.md`，以及 `docs/0X_*/README.md`（6 个）。

> 这实际上暴露了 V1 的一个隐含分层：**知识文档有元数据，导航与治理文档没有**。V2 应把这条隐含规则显式化，而不是简单地给所有文件补 front matter。

### 3.2 front matter 字段频次

| 字段 | 出现次数 | 说明 |
|---|---:|---|
| `title` / `type` / `status` / `created` / `updated` / `tags` | 各 63 | 6 个核心字段，覆盖率 100%（在有 FM 的文件中） |
| `migrate_source` | 16 | 迁移来源（溯源字段，已自发出现） |
| `short_name` | 16 | 稳定短名（**事实上的人类可读标识符**） |
| `pdf_index` | 15 | 指向仓库外 PDF 的索引路径 |
| `authors` / `venue` / `year` / `paper_link` / `code_link` / `project_link` / `local_file` / `paper_version` / `read_date` / `read_status` / `primary_dimension` / `related_dimensions` / `primary_level` / `keywords` | 各 9 | **仅存在于 `d6_failure_detection/` 的 9 篇笔记** |
| `source_note` | 2 | 零散 |

**关键发现（schema 分叉）**：仓库中存在**两套并行的论文笔记元数据 schema**——

- 通用 schema：6 字段（其余 paper-note 使用）；
- D6 schema：22 字段（含 `paper_version`、`read_date`、`primary_dimension` 等）。

D6 的这套字段更接近 V2 需要的形态（时间语义、来源版本、阅读状态、维度归属都已具备）。它是自发演化出来的，**说明 6 字段 schema 对论文类知识确实不够用**，也说明 V2 的「type 扩展字段」设计有真实需求支撑，而不是凭空增加负担。

### 3.3 受控词表漂移（实测）

| 词表 | README 声明 | 实际使用 | 漂移 |
|---|---:|---:|---|
| `tags` | 14 个白名单 | **39 个不同值** | 25 个越界（`failure-detection`、`vlm`、`ppo`、`grpo`、`gae`、`cross-embodiment`、`flow-matching`、`post-training`、`vla-post-training`、`lingbot`、`action-chunking`、`diffusion-policy`、`autoregressive`、`action-expert`、`kv-cache`、`lego`、`depth`、`spatial-perception`、`3d-reconstruction`、`mapping`、`causal-world-model`、`robot-control`、`foundation-model`、`open-source`、`soft-prompt`、`lora`、`engineering` 等） |
| `type` | 6 个（paper-note / topic / foundation / experiment / idea / report） | **10 个** | 新增 `index`、`paper-index`、`project`、`research-map`；且 `index` 与 `paper-index` 语义重复 |
| `status` | 4 个（seed / active / stable / archived） | **2 个**（seed / active） | 从未有文档进入 `stable` 或 `archived`——状态机的后半段是死的 |

**含义**：V1 已经做过一次「受控词表」尝试，并且在 6 天内就漂移到 2.8 倍。这为 V2 提供了**经验证据**：

1. 靠文档约定（README 白名单）+ 人工自律，无法维持受控词表；
2. 必须要么**机器校验**，要么**降级为非权威字段**；
3. `status` 的四态设计过度——真实使用中只需要区分「在写 / 已稳定」，而「历史 / 被取代 / 有争议」这类语义在 V1 中根本没有承载处（这正是 V2 `knowledge_status` 要补的洞）。

---

## 4. 链接与索引健康度实测

### 4.1 链接统计

| 指标 | 数量 |
|---|---:|
| 内部相对链接 | 239 |
| 外部 URL | 57 |
| **失效内部链接** | **21** |
| **越出仓库边界的相对链接** | **5**（指向 `VLA/数据质量评估/`） |

### 4.2 失效链接清单（21 条）

| 源文件 | 失效目标 | 错误类型 |
|---|---|---|
| `01_foundations/reinforcement_learning/学习规划与进度.md` | `../../docs/03_papers/ript_vla/RIPT-VLA_阅读笔记.md` | 多写了 `docs/` 段 |
| 同上 | `../../docs/03_papers/ript_vla/RIPT-VLA_Rollout存储与OnOffPolicy_问答.md` | 同上 |
| `01_foundations/reinforcement_learning/长期学习笔记.md` | `../../docs/03_papers/ript_vla/RIPT-VLA_阅读笔记.md` | 同上 |
| 同上 | `../../docs/03_papers/ript_vla/RIPT-VLA_GRPO_vs_RLOO_对比分析.md` | 同上 |
| 同上 | `../../docs/03_papers/ript_vla/RIPT-VLA_PPO_clip_epsilon_调优分析.md` | 同上 |
| `02_topics/action_chunk_flow_matching/动作块与流匹配动作生成.md` | `../01_foundations/reinforcement_learning/` | 目录链接，缺少目标文件 |
| `02_topics/embodied_data_quality/实机机器人数据质量评估总览.md` | `../03_papers/d6_failure_detection/INDEX.md` | 缺少 `../` |
| `02_topics/vla_post_training/VLA后训练中的奖励与信用分配.md` | `../docs/03_papers/ript_vla/...`（3 处） | 层级错误 |
| `03_papers/ar_vla/AR-VLA_阅读笔记.md` | `../02_topics/action_chunk_flow_matching/...` | 缺少 `../` |
| `03_papers/a_survey_of_datasets/...` | `../embodied_data_quality/实机机器人数据质量评估总览.md` | 跨层错误 |
| `03_papers/data_assessment_for_embodied_intelligence/...` | 同上 + `DataAssessmentforEmbodiedIntelligence.pdf` | 跨层错误 + 文件名不符（实际为带空格文件名） |
| `03_papers/data_quality_in_imitation_learning/...` | `../embodied_data_quality/...` | 跨层错误 |
| `03_papers/robot_data_curation_.../...` | `../embodied_data_quality/...` | 跨层错误 |
| `04_experiments/PPO_CartPole_可复现实验记录.md` | `../../01_foundations/...`、`../../03_papers/...` | 多写一层 |
| `templates/REPORT_TEMPLATE.md` ×3 | `链接` | 模板占位符（可接受，但会污染校验结果） |

**根因**：手写相对路径 + 没有任何校验。这是纯机械错误，**100% 可自动检测、95% 可自动修复**——是 V2 派生层（`derived/health/`）最容易兑现价值的地方。

### 4.3 越界链接（5 条）

`docs/02_topics/embodied_data_quality/` 下 5 个文件包含：

```markdown
[数据质量评估（VLA/ 原文件夹）](../../../../数据质量评估/)
```

该路径指向仓库外的 `D:\materials\VLA\数据质量评估\`。在本机可解析，**在 GitHub 上必然 404**。

同类问题还有 15 个 `pdf_index` 字段与多个 `INDEX.md` 中写死的绝对路径，例如：

```yaml
本地 PDF 路径（索引，不进 git）：D:/materials/VLA/openvla/2406.09246v3.pdf
```

**含义**：V1 已经做出了「源材料不进库、只留索引」这一正确决策，但**索引的表达方式是机器绑定的绝对/越界路径**，既不可移植也不可校验。V2 的 Source 层要解决的正是这件事。

### 4.4 索引漂移

| 索引 | 声明 | 实际 | 状态 |
|---|---|---|---|
| `PROJECT_INVENTORY.md` | 34 条 | 仓库 85 个文件 | **严重过期**（停留在初始化版本） |
| `docs/04_experiments/README.md` | 「_（暂无）_」 | 实际有 1 个实验文档 | 过期 |
| `docs/05_ideas/README.md` | 「_（暂无）_」 | 确实为空 | 准确 |
| `docs/01/02/03/06 README.md` | 手写条目表 | 与实际基本一致 | 目前准确，但**全靠人工维护** |
| 孤儿文档（0 条入链） | — | 12 个 | 含 4 个数据质量支撑文档（`L1核心精读总结`、`PAPER_MATRIX`、`READING_LOG`、`分层阅读计划`）与 `templates/REPORT_TEMPLATE.md` |

**含义**：所有索引都是**人工维护的派生数据**。它们必然漂移，而且已经漂移。这是 V2「Derived Data 必须可从 canonical 重建」这条原则的最直接动因。

---

## 5. V1 的隐含设计（未写入文档但真实存在）

这些是审计中发现的、**没有被 V1 显式命名但已经在运行**的设计。它们是 V2 最重要的继承对象。

| # | 隐含设计 | 证据 | V2 价值 |
|---|---|---|---|
| I1 | **证据强度标注体系** | D6 的 9 篇笔记全文使用 `[PAPER]` / `[RESULT]` / `[INFERENCE]` / `[OPEN]` 四级标签；AGENTS §5.3 与所有模板要求区分「作者报告 / 独立复现 / 个人判断 / 用户假设」 | **V1 最有价值的发明**。V2 必须把它升格为一等公民：既是写作规范，也是 Agent 的**写保护边界**（带 `[INFERENCE]` 的段落不允许自动改写） |
| I2 | **核验日期（last verified）** | AGENTS §6 要求记录核验日期；`d6` schema 有 `read_date`；多篇文档正文有「资源核验日期：2026-07-31」 | 已有字段，但**是被动的**。V2 加上 `temporal_class` 后即可变成主动的「复验债务」计算 |
| I3 | **迁移溯源头** | 16 个文件有 `migrate_source`；D6/LingBot 笔记正文顶部有「迁移与核验说明」引用块 | 这就是 V2 的 `sources` / `derived_from` 关系的手写版 |
| I4 | **Source / Knowledge 事实分离** | 「PDF 不纳入 git，仅建索引」是被反复执行的默认边界（CHANGELOG 三处备注）；大文件排除规则写在 `MIGRATION_CHECKLIST` §1 | V2 三层分离的 Source 层**已经存在**，只是没有被命名，也没有 registry |
| I5 | **双文档 living 模式** | `01_foundations/reinforcement_learning/` = `学习规划与进度.md`（状态）+ `长期学习笔记.md`（内容），教学节奏写在目录 README | 这已经是一个 **living document 对**：一个是可变状态，一个是累积内容。V2 的 living document 模型应以它为原型 |
| I6 | **五分类处置词表** | `MIGRATION_CHECKLIST` 处置方式 = 迁移 / 改写 / 仅建索引 / 暂缓 / 排除 | V2 的 intake router 应**直接复用这五个词**，不要另发明一套 |
| I7 | **簇（cluster）作为组织单元** | `ript_vla/` 一篇论文 4 个文档、`lingbot/` 一个家族 7 篇、`d6_failure_detection/` 一个维度 9 篇 | 实际组织粒度不是「文档」也不是「目录层」，而是**知识簇**。V2 应显式支持 cluster 概念 |
| I8 | **type 与目录不一致是常态** | `ript_vla/` 下 3 个 `type: topic` 文档位于 `03_papers/`；`lingbot/analysis_summary.md` 同样 | 证明**路径不能作为类型来源**。V2 必须把 type 从路径中解耦 |
| I9 | **INDEX.md 承担双重职责** | 每个 `INDEX.md` 同时是（a）论文来源元数据卡片，（b）目录导航表 | V2 必须拆分：(a) → Source registry，(b) → Derived index |
| I10 | **Agent 默认只 add 不 commit** | AGENTS §9 + CHANGELOG 四处「本次仅 `git add` 追踪，未 `git commit`」 | 权限模型的雏形。V2 的角色化权限矩阵应保留它作为 `default_agent` 的基线 |

---

## 6. 技术债务清单

按「修复成本 × 阻塞 V2 的程度」排序。

| # | 债务 | 严重度 | 阻塞 V2？ | 可自动修复 |
|---|---|---|---|---|
| D1 | **文档没有稳定标识**——身份完全依赖文件路径 | 高 | **是**（阻塞 living document、关系、别名、重命名） | 否（需要一次性 uid 注入） |
| D2 | **Git 无有效文档级历史**（2 次提交承载全部内容） | 高 | **是**（阻塞「Git 作为版本存储」的前提） | 否（只能从 V2 起累积） |
| D3 | 21 条失效内部链接 | 中 | 否 | 是（约 18/21） |
| D4 | 5 条越界链接 + 15 处机器绑定绝对路径 | 中 | **是**（阻塞 Source 层与仓库可移植性） | 半自动（路径→source_id 需要建 registry） |
| D5 | tags 词表漂移 14→39；type 词表漂移 6→10 且有重复项 | 中 | 部分（阻塞 by-tag 检索的可信度） | 是（可归一化 + 校验） |
| D6 | `PROJECT_INVENTORY.md` 严重过期；目录 README 条目表人工维护 | 中 | 否（但每天都在恶化） | 是（改为派生） |
| D7 | 元数据 schema 分叉（6 字段 vs 22 字段） | 中 | 部分 | 半自动 |
| D8 | `RESEARCH_MAP.md` 混合四种职责（地图 / 知识节点 / 索引 / 变更日志） | 中 | **是**（阻塞 maps 与 indexes 的分离） | 否（需人工拆分） |
| D9 | `status` 后两态（stable / archived）从未使用；缺少 historical / superseded / disputed 语义 | 低 | 部分 | 否 |
| D10 | `docs/05_ideas/` 为空；`04_experiments/` 仅 1 篇且结果待回填 | 低 | 否 | 否 |
| D11 | 无 LICENSE、无 CI、无任何校验脚本 | 低 | 否（但 V2 校验层需要落脚点） | 是 |
| D12 | 中文文件名 + Windows + Git：`git` 输出中出现八进制转义路径 | 低 | 否 | 是（`core.quotepath=false`） |
| D13 | 12 个孤儿文档（无入链），其中 4 个是数据质量核心支撑文档 | 低 | 否 | 是（派生索引可兜底） |

---

## 7. V2 可复用组件（继承清单）

**直接继承，不改动：**

- `.gitattributes` 的 LFS 规则（`*.pdf` / `*.pptx` / `*.ppt`）——已验证可用，9 个对象已入 LFS；
- `.gitignore`（含 `.workbuddy/` 排除）；
- 7 个 `templates/`——它们实质上是 **7 个对象类型的 schema 草案**，V2 直接升级为 typed template；
- AGENTS.md §5（写作规范：术语首现、LaTeX、结论强度）——语言层规范与架构无关，全量保留；
- AGENTS.md §6（来源与链接核验优先级）——升级为 Source 层策略；
- AGENTS.md §7（当前研究关注点的分析检查表：数据 / 世界模型 / RL / VLA）——这是 `embodied-ai` 域的**领域专属审阅清单**，V2 中下沉为 domain-level 规则文件；
- AGENTS.md §9（Git 规则）——升级为 `default_agent` 的权限行；
- `MIGRATION_CHECKLIST.md` §1 排除规则与五分类处置词表——直接成为 intake router 的规则表。

**改造后继承：**

- `INDEX.md` 元数据部分 → `sources/registry/<source_id>.yaml`；
- `INDEX.md` 导航部分 + 目录 README 条目表 + `PROJECT_INVENTORY.md` → `derived/indexes/`；
- `RESEARCH_MAP.md` → 拆为 `maps/embodied-ai.md`（地图）+ 开放问题知识节点 + 派生索引 + research-state 快照；
- `migrate_source` / `pdf_index` / `local_file` → `sources[]` 关系 + source registry 引用；
- `[PAPER]/[RESULT]/[INFERENCE]/[OPEN]` → 正文保留原样，同时作为 Agent 写保护标记；
- 双文档学习模式 → living document 的参考实现。

---

## 8. 迁移风险清单

| # | 风险 | 触发条件 | 影响 | 缓解 |
|---|---|---|---|---|
| R1 | **批量注入 uid 产生 63 文件大 diff**，淹没真实内容变更 | M2 阶段 | 用户无法审阅；污染 Git blame | 单独一个 commit；按簇分批；commit message 明确标注「纯元数据，正文零改动」；提供 diff 校验脚本证明正文字节未变 |
| R2 | **`git mv` 之外的文件移动会丢失历史** | 任何目录重排 | 无法追溯 | 强制只用 `git mv`；移动与内容修改**永不同 commit** |
| R3 | **21 条失效链接的「修复」可能指向错误目标** | M3 自动修复 | 静默错链 | 只修复候选唯一的情况；候选 0 或 ≥2 → `review/ambiguous/` |
| R4 | **越界链接改写会切断用户与源材料的关联** | M3/M4 | 用户找不到原始 PDF/代码 | 不删除路径信息，转为 `sources/registry` 条目并保留 `local_hint` 原文 |
| R5 | **V1 与 V2 双结构并存期间「这个知识在哪」失去唯一答案** | M1–M7 全程 | 用户与 Agent 都可能重复建档 | HOME + `derived/indexes/` 从 M1 起**同时覆盖 V1 与 V2 两侧**；新建内容一律进 V2 |
| R6 | **Agent 误判 living document 更新，覆盖用户观点** | 任何 nightly 运行 | 不可逆知识损失（V1 无历史可回滚，见 D2） | 截断守卫 + 观点段落写保护 + 第一阶段 nightly 只写分支不合并 + 全程 dry-run 先行 |
| R7 | **元数据负担超过写作收益，用户放弃填写** | M2 之后 | schema 名存实亡（V1 tags 已有先例） | 必填字段压到 7 个；其余由 Agent 回填；schema 校验失败只警告不阻塞 |
| R8 | **派生数据不确定性导致每晚都有噪声 diff** | derived 重建 | 提交历史被污染，真实变更被淹没 | 排序全部显式；VALIDATE 阶段做「重建两次字节一致」的确定性测试 |
| R9 | **PPTX/PDF 已在 LFS，若迁移路径变更会产生 LFS 指针失配** | 涉及 `03_papers/*/`、`06_reports/` 的移动 | 大文件丢失 | 这两类**第一阶段完全不动**；确需移动时先 `git lfs ls-files` 核对再 `git mv` |
| R10 | **中文路径 + Windows + LFS + 分支切换**的编码/长路径问题 | nightly 分支操作 | 分支切换失败、文件名乱码 | `core.quotepath=false`、`core.longpaths=true`；uid 寻址使文件名不参与逻辑判断 |
| R11 | **对一个 74 文件的仓库引入五层架构，过度工程** | 全程 | 系统维护成本超过知识本身 | MVP 只做 report-only 的健康检查 + schema + intake 骨架；每增加一层必须先出现真实痛点 |

---

## 9. 一句话结论

> V1 不是失败设计——它已经自发长出了 V2 需要的**大部分语义**（证据强度、核验日期、迁移溯源、源/知识分离、簇组织、双文档 living 模式、五分类处置）。
>
> V1 缺的不是概念，而是**执行机制**：没有稳定标识、没有机器校验、没有派生重建、没有版本史、没有权限分级。
>
> 因此 V2 的正确定位是 **给 V1 已有语义配上执行机制**，而不是重新设计一套语义。

---

## 10. 更新记录

- 2026-08-07：建立本审计清单（Phase 0）。基于对工作区与 Git 仓库的实测扫描，记录结构规模、元数据现状、链接健康度、10 项隐含设计、13 项技术债务、可复用组件与 11 项迁移风险。
