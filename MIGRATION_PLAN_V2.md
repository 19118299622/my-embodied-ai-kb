---
title: V2 迁移计划
type: project
status: active
created: 2026-08-07
updated: 2026-08-07
tags:
  - project-maintenance
  - architecture
  - v2-design
---

# MIGRATION_PLAN_V2 — 迁移计划（Phase 1c）

> 本文件是 V1 → V2 架构升级的 **Phase 1c 产出**：把 [`V1_TO_V2_MAPPING.md`](./V1_TO_V2_MAPPING.md) 的映射关系排成**可执行、可验证、可回滚**的阶段序列。
>
> **本轮到此为止。执行需要用户批准（见 §7）。**
>
> 配套文档：[`V1_INVENTORY.md`](./V1_INVENTORY.md)、[`V2_ARCHITECTURE_PROPOSAL.md`](./V2_ARCHITECTURE_PROPOSAL.md)。

---

## 1. 计划的五条约束

这五条是本计划的边界条件，任何阶段都不得违反。

| # | 约束 | 落地方式 |
|---|---|---|
| 1 | **不一次性重排 `docs/`** | M0–M5 全程 `docs/` 零移动；M6 之后才逐簇迁移，且以簇为最小事务 |
| 2 | **V1 与 V2 短期共存** | `docs/` 与 `kb/` 并行；派生索引从第一天同时覆盖两侧，用户只看一个索引 |
| 3 | **不破坏 Git 历史** | 只增量提交；不 rebase、不 amend、不 force push、不改写历史 |
| 4 | **不删 `MIGRATION_BACKLOG.md` / `MIGRATION_CHECKLIST.md`** | 全程保留；`MIGRATION_CHECKLIST §1` 反而升格为 `schema/intake-rules.yaml` 的来源 |
| 5 | **知识判断的最终控制权属于用户** | 所有 A0/A1 项必须人工确认；nightly 分支**永不自动合并 main**（但 `human` 可随时手动合并 nightly 分支——合并动作由用户执行） |

> 补充约束（来自 V1_INVENTORY D2）：**V1 只有 2 次提交，文档级历史事实上不存在**。这意味着「出错了用 Git 回滚单个文档」这条安全网在 V1 内容上是失效的。因此**越早期的阶段，越必须只读**。

---

## 2. 先不迁的内容（明确清单）

| 对象 | 数量 | 不迁到什么时候 | 理由 |
|---|---:|---|---|
| `docs/` 下所有文件的**物理位置** | 74 md | M6 之后，且逐簇 | 用户明确禁止；uid 注入后再移动风险显著降低 |
| `docs/` 下所有文件的**正文** | 74 md | **永不自动改** | 不变量 #2；正文改动一律人工 |
| 9 个 LFS 对象（5 PDF + 4 PPTX） | 9 | M7，单独人工事务 | Windows + 中文名 + LFS 组合的工程风险；nightly 永不触碰 LFS |
| `RESEARCH_MAP.md` | 1 | M6 | 全库信息密度最高，必须人工拆分；且拆分后的 question 节点需要 `kb/` 已就绪 |
| `README.md` / `AGENTS.md` 的重写 | 2 | M1（AGENTS）/ M6（README） | AGENTS 要先立规矩所以早；README 是门面，等结构稳定后再改 |
| `MIGRATION_BACKLOG.md` / `MIGRATION_CHECKLIST.md` | 2 | 不迁（可能永久保留） | 用户要求；且记录的是**另一条**迁移线（历史材料 → 知识库），与本轮结构升级不是一回事 |
| 4 个 PPTX 的**内容解析** | 4 | 不做 | 属 Source 层，Agent 只登记不加工 |
| 仓库改名 | — | 不做 | 用户明确说明 |
| `docs/05_ideas/`（空目录） | — | M6 自然填充 | `RESEARCH_MAP §4` 拆分后会产生 5 个真实节点 |
| 全量 uid 注入（63 个文件） | 63 | M4 | M3 先用 1 个簇验证方案 |
| 全量 sidecar | — | M4 之后按需 | sidecar 只在文档被真正处理过时才生成 |

---

## 3. 试点选择

### 3.1 试点标准

一个好的迁移试点必须同时满足：

1. **样本小**（diff 可人工逐行看完）；
2. **形态杂**（能暴露方案缺陷，而不是挑最容易的）；
3. **痛点真**（迁完立刻有可见收益，而不是纯仪式）；
4. **失败可弃**（错了整簇 revert，不影响其他内容）。

### 3.2 主试点：`docs/02_topics/embodied_data_quality/`（5 个文档）

| 文件 | 现状 | 试点验证什么 |
|---|---|---|
| `实机机器人数据质量评估总览.md` | `type: topic`，是簇的主文档 | `synthesis.topic` + `cluster_role: primary` |
| `分层阅读计划.md` | 用 `topic` 凑合表达「计划」 | **新类型 `plan.learning`** 是否成立 |
| `L1核心精读总结.md` | 跨论文综合 | 与 `literature` 的边界是否清晰 |
| `PAPER_MATRIX.md` | 手工维护的论文矩阵 | **能否由元数据派生**（这是 V2 最核心的赌注） |
| `READING_LOG.md` | append-only 时间流 | **新角色 `log.reading`** 是否成立 |

它同时命中：**5 条越界链接**（全部在这个簇）、**4 个孤儿文档**（全部在这个簇）、**双文档 living 模式的变体**、**手工索引可否派生**。

> 换句话说：这 5 个文件是 V1 全部结构性问题的**最小完整样本**。跑通它，V2 的核心假设基本都被验证过一次。

### 3.3 次试点：`docs/03_papers/d6_failure_detection/`（9 篇 + 1 INDEX，M4 启动）

理由：

- 它是唯一使用 **22 字段 schema** 的簇——是 `note.literature` 类型扩展字段的现成样本；
- 它全文使用 **`[PAPER]/[RESULT]/[INFERENCE]/[OPEN]` 证据标注**——是「Agent 写保护边界」的最佳验证对象；
- 它的 `INDEX.md` 元数据最丰富——是 **source registry 抽取**的最佳样本；
- 9 篇同源笔记——是**重复检测必须以簇为单位**这条规则的压力测试。

### 3.4 明确不选做试点的

| 候选 | 为什么不选 |
|---|---|
| `ript_vla/` | 含 LFS PDF，第一阶段不碰 LFS |
| `lingbot/` | 7 篇 analysis 的 type 归属本身就有争议（笔记 vs 专题），会把「类型判定」和「迁移机制」两个变量混在一起 |
| `01_foundations/reinforcement_learning/` | 领域归属未定（`embodied-ai` 还是 `machine-learning`），是 §7 的待确认问题之一 |
| `06_reports/` | 全是 LFS PPTX |

---

## 4. 迁移阶段（M0 → M7）

### 阶段总览

```text
M0  基线冻结          只读        0 个文件被改
M1  地基（schema）    纯新增      0 个已有文件被改
M2  只读健康层        只读        0 个已有文件被改
M3  主试点            首次触碰    5 个文件的 front matter（正文零改动）
M4  Source 层         中等        15 个 INDEX + 9 篇 D6
M5  dry-run 策展      只读        0 个文件被改（观察 ≥ 2 周）
M6  分解与派生化      重          RESEARCH_MAP / README / 索引
M7  逐簇迁入 kb/      长期        使用时迁移，无终点压力
```

---

### M0 — 基线冻结

| 项 | 内容 |
|---|---|
| **目标** | 建立一个「出事能回到这里」的确定性锚点 |
| **交付物** | 1) 打 tag `v1-final`；2) 记录 `git rev-parse HEAD`；3) 设置 `core.quotepath=false`、`core.longpaths=true`；4) `git lfs ls-files` 输出存档 |
| **触碰的已有文件** | **0** |
| **Agent 可做** | 全部（A3，纯只读 + 打 tag） |
| **必须人工** | 确认 tag 名称 |
| **退出条件** | tag 存在且 `git status` clean |
| **回滚** | 不适用 |
| **diff 预算** | 0 行 |

> 这一步只花几分钟，但它是后面所有「可回滚」承诺的物理基础。

---

### M1 — 地基：schema 与治理

| 项 | 内容 |
|---|---|
| **目标** | 在任何 Agent 动手之前先把规矩写死 |
| **交付物** | `schema/domains.yaml`、`object-types.yaml`、`relations.yaml`、`metadata.schema.yaml`、`intake-rules.yaml`（继承 `MIGRATION_CHECKLIST §1`）、`tag-aliases.yaml`；`AGENTS.md` v2（权限矩阵 + 五条红线 + intake/nightly 章节）；`inbox/` 与 `review/{ambiguous,conflicts,duplicates}` 骨架 + README |
| **触碰的已有文件** | 1（`AGENTS.md`） |
| **Agent 可做** | 起草 schema（A1，需人工审核取值集合） |
| **必须人工** | `domains.yaml` 的领域清单；`AGENTS.md` 的红线条款；`object-types.yaml` 的类型闭集 |
| **退出条件** | 用户逐条确认 6 个领域、12 个对象类型、10 个关系词；`inbox/` 可以开始丢东西 |
| **回滚** | 全部是新增文件，`git rm` 即可；`AGENTS.md` 单文件 revert |
| **diff 预算** | 新增 ~400 行，修改 ~150 行 |

**风险**：类型闭集定早了会束缚，定晚了会漂移。缓解——闭集允许**由用户扩充**，只是 Agent 无权扩充。

---

### M2 — 只读健康层（**Foundation MVP = Checkpoint A，建议的 MVP 终点**）

| 项 | 内容 |
|---|---|
| **目标** | 让 74 个文件的健康状况一次性可见，立刻兑现价值 |
| **交付物** | `tools/check_health.py`（链接检查 / schema 检查 / 孤儿检查 / 复验超期 / 重复候选）；`derived/health/{links,schema,orphans,stale,duplicates}.md` 首份报告；`derived/indexes/` 首批派生索引（覆盖 `docs/` 全部 74 个文件） |
| **触碰的已有文件** | **0** |
| **Agent 可做** | 全部（A3——派生产物，删了能重建） |
| **必须人工** | 无（但需要看一遍报告，判断误报率） |
| **退出条件** | 报告能复现 V1_INVENTORY 的 21 条断链、12 个孤儿、词表漂移；连跑两次输出字节一致（确定性测试） |
| **回滚** | `rm -rf derived/` |
| **diff 预算** | 新增 ~600 行（其中大部分是生成内容） |

> **这是投入产出比最高的一步**：不改任何内容，就把 21 条断链、5 条越界链接、39 个 tag 的漂移、12 个孤儿、`PROJECT_INVENTORY` 的过期全部变成可见清单。
>
> **建议：MVP 就停在这里，先用两周。** 如果两周后用户没打开过 `derived/health/`，说明整个 V2 的价值假设需要重估（V2 §22 风险 1 的早期信号）。

---

### M3 — 主试点：`embodied_data_quality` 簇

| 项 | 内容 |
|---|---|
| **目标** | 用 5 个文件验证 uid + sidecar + 新类型三件事 |
| **交付物** | 5 个文件的 front matter 注入 `uid` / `domain` / `type`(新值) / `document_state`(新值) / `knowledge_status`(新值) / `cluster`；5 个 `.meta.yaml` sidecar（仅 durable 字段）；`PAPER_MATRIX.md` 的**派生版原型**（与手工版并列，供对比）；`.gitattributes` 增加 `*.meta.yaml linguist-generated=true` |
| **触碰的已有文件** | 5（**仅 front matter，正文字节零改动**） |
| **Agent 可做** | uid 生成、sidecar 生成、type/status 机械映射（A2 → 落 nightly 分支等审） |
| **必须人工** | `plan.learning` / `log.reading` 两个新类型是否成立；`PAPER_MATRIX` 是否接受派生版取代手工版（A0） |
| **退出条件** | 1) `git diff` 证明 5 个文件正文字节未变（用 hash 校验）；2) 派生版 PAPER_MATRIX 信息不少于手工版；3) 5 条越界链接被 registry 记录 |
| **回滚** | `git revert` 单个提交；sidecar 直接删除 |
| **diff 预算** | 修改 5 个文件各 ~6 行 front matter；新增 5 个 sidecar |

**这一阶段的关键检验**：如果 `PAPER_MATRIX.md` **无法**由元数据完整派生（例如它含有手工写的对比判断），那么 V2「索引全部派生」的假设就需要修正——**这类矩阵应归为 `synthesis.topic` 而非 index**。这个发现比迁移本身更重要。

---

### M4 — Source 层与次试点

| 项 | 内容 |
|---|---|
| **目标** | 把 15 个 `INDEX.md` 的双重职责拆开，解决绝对路径问题 |
| **交付物** | `sources/registry/*.yaml`（约 20 条：15 个 INDEX + 5 个 PDF 论文）；`sources/files/` 目录（**暂不移入文件**）；D6 簇 9 篇的 uid 注入 + 22 字段 schema 正式化；`derived/health/portability.md`（统计不可移植来源） |
| **触碰的已有文件** | 15 个 INDEX（标注 deprecated，暂不删）+ 9 篇 D6 的 front matter |
| **Agent 可做** | 字段抽取、registry 生成（A1——需人工确认） |
| **必须人工** | **`official_repo` 是否真的是官方仓库**（AGENTS §6.2 明确禁止按名称相似度猜测）；每条 registry 的 `canonical_url` 核验 |
| **退出条件** | 20 条 registry 全部人工过目；`portable: false` 的数量被明确记录（预期 15） |
| **回滚** | registry 是纯新增，删除即可；INDEX 的 deprecated 标注单独 revert |
| **diff 预算** | 新增 ~500 行 yaml；修改 24 个文件各 ~3 行 |

---

### M5 — dry-run 策展（**Curator Dry-run MVP，观察期**）

| 项 | 内容 |
|---|---|
| **目标** | 在赋予任何写权限之前，先看它判断得准不准 |
| **交付物** | `tools/curate.py --dry-run`：走完 SCAN → IDENTIFY → CLASSIFY → ASSOCIATE → DETECT_UPDATE → ROUTE，**只出报告，不写 `kb/`**；`reports/nightly/YYYY-MM-DD.md`（Operational History，非 Derived） |
| **触碰的已有文件** | **0** |
| **Agent 可做** | 全部（只读） |
| **必须人工** | 每天扫一眼分诊报告，标记误判 |
| **退出条件** | **连续 ≥ 2 周**；L1–L3（可自动识别）占比 ≥ 70%；L4/L5（需人工）中的误判率可接受；`review/` 没有堆积 |
| **回滚** | 不适用（只读） |
| **diff 预算** | 每天新增 1 个报告文件 |

**测试用例（必须覆盖）**：

| 场景 | 期望行为 |
|---|---|
| 同一篇 `长期学习笔记.md` 连续 3 天丢进 `inbox/`，文件名带不同日期前缀 | 三次都识别为**同一 logical document 的更新**（L2/L3 命中），不新建文件 |
| 一篇全新论文笔记 | L6 → 新建，分配新 uid |
| 同簇的第 10 篇 D6 笔记 | 不被判为重复（重复检测以簇为单位） |
| arXiv v1 已存在，丢进 v2 | 追加 version，uid 不变，**不判 duplicate** |
| 一份无法归入 6 个领域的内容 | 进 `review/ambiguous/`，**不自动开新领域** |
| 一份内容比库中版本短 60% | 触发截断守卫 → `review/conflicts/`，不落地 |

---

### M6 — 分解与派生化

| 项 | 内容 |
|---|---|
| **目标** | 拆 `RESEARCH_MAP.md`，把人工索引全面换成派生 |
| **交付物** | `maps/embodied-ai.md`；`kb/embodied-ai/ideas/` 下 5 个 `hypothesis.question` 节点 + 各主线的 question 节点；`kb/embodied-ai/reports/2026-08-07-research-state.md`（冻结快照）；`README.md` v2（HOME）；6 个目录 README 降级；`PROJECT_INVENTORY.md` 转派生 |
| **触碰的已有文件** | `RESEARCH_MAP.md`、`README.md`、6 个目录 README、`PROJECT_INVENTORY.md` |
| **Agent 可做** | 生成派生索引（A3）；提议 question 节点的**结构**（A1） |
| **必须人工** | **五个开放问题的措辞、边界、falsifier 全部人工撰写**（A0）；`maps/` 的认知结构（A0）；README 重写（A0） |
| **退出条件** | 人画的专题关系图与 `derived/graph/` 无矛盾（或矛盾已被解释）；`RESEARCH_MAP.md` 变成短导航页且无信息丢失 |
| **回滚** | 单文件 revert；question 节点删除 |
| **diff 预算** | 新增 ~800 行；修改 ~400 行 |

> **这是全计划中唯一一个「人工工作量 > Agent 工作量」的阶段**，也是价值最高的阶段。不要试图加速它。

---

### M7 — 逐簇迁入 `kb/`（长期、无终点压力）

| 项 | 内容 |
|---|---|
| **目标** | 按「使用时迁移」原则，把 `docs/` 内容逐簇搬进 `kb/` |
| **触发条件** | 某个簇被**真实使用**（被引用、被更新、被扩写）时才迁移它——继承 V1 的原则 |
| **每次事务** | 一个完整的簇：`git mv` + `previous_paths` 记录 + 入链修复 + 派生索引重建，**一次提交** |
| **LFS 对象** | 单独人工事务，迁完立刻 `git lfs ls-files` 验证 |
| **必须人工** | 领域归属（尤其 C5 的 RL 基础归 `embodied-ai` 还是 `machine-learning`）；`type` 与目录不一致的文档（I8）的最终类型 |
| **退出条件** | **没有硬性终点**。`docs/` 可以长期存在；只要派生索引同时覆盖两侧，用户就永远只需要看一个索引 |
| **回滚** | 每簇一个提交，单独 revert |

---

## 5. 自动化裁定（执行时的操作分级）

### 5.1 可以让 Agent 自动做的（A2/A3）

| 操作 | 等级 | 前置条件 |
|---|---|---|
| 生成 / 重建 `derived/**` 全部内容 | A3 | 确定性测试通过 |
| 链接健康检查、孤儿检测、复验债务计算 | A3 | — |
| 修复指向 `previous_paths` 的旧链接 | A2 | uid-map 已建立 |
| 修复 21 条机械断链（相对路径重算） | A2 | 人工确认修复规则一次，之后批量 |
| 注入 `uid`（front matter 追加一行） | A2 | 提供「正文字节未变」的 hash 证据 |
| 生成 / 更新 sidecar | A2 | — |
| `type` / `status` 的机械值映射 | A2 | 映射表已确认（`V1_TO_V2_MAPPING §5.3/5.4`） |
| 在 `inbox/` 中识别 L1–L3 身份命中 | A2 | dry-run 期误判率可接受 |
| 追加型 living 更新（V2 §8.2 类别 A） | A2 | M5 观察期结束后才开启 |
| 生成夜间报告（`reports/nightly/`，非 Derived） | A3 | — |

### 5.2 必须人工审核的（A0/A1）

| 操作 | 等级 | 为什么 |
|---|---|---|
| 新增 / 重命名 / 删除**领域** | **A0** | 最高层骨架，漂了就回不来（tags 已有前车之鉴） |
| 删除任何内容（tag、段落、文件、链接） | **A0** | 不可逆 |
| 标记 `superseded` / `historical` / 解决 `disputed` | **A0** | 这是知识判断本身 |
| 改写带 `[INFERENCE]` / `[OPEN]` 标记的段落 | **A0** | V1 最有价值的发明，必须成为写保护边界 |
| `RESEARCH_MAP §4` 五个开放问题的拆分与措辞 | **A0** | 措辞就是研究问题本身 |
| 移动 LFS 对象 | **A0** | 工程风险 |
| 重写 `README.md` / `AGENTS.md` | **A0** | 门面与规矩 |
| 合并 nightly 分支到 main | **A0（永久红线）** | 不自动合并 → 回滚成本恒为零 |
| 判定 GitHub 仓库是否「官方」 | **A0** | AGENTS §6.2 明令禁止按名称猜测 |
| 领域归属（某文档属哪个 domain） | A1 | 语义判断 |
| tag 归一化与同义合并 | A1 | 可能丢语义 |
| 簇的迁移（`git mv`） | A1 | 影响路径与入链 |
| source registry 的字段抽取 | A1 | `canonical_url` 需核验 |
| L4/L5 身份候选（标题相同 / 正文相似） | A1 | 一律进 `review/`，不自动落地 |
| 结构重排型 living 更新（V2 §8.2 类别 C） | A1 | 需看 diff |

### 5.3 一条判定捷径

> 执行时如果拿不准某个操作该不该自动，问一句：
> **「如果它做错了，用户不看 diff 能发现吗？」**
> 不能发现 → 至少 A1。

---

## 6. 验证与回滚

### 6.1 每阶段必过的验证

| 检查 | 方法 | 适用阶段 |
|---|---|---|
| 正文字节未变 | 迁移前后对正文（去 front matter）取 sha256 比对 | M3 / M4 / M7 |
| 派生确定性 | 连跑两次 `check_health.py` + 索引生成，输出必须字节一致 | M2 起每阶段 |
| 链接不退化 | 断链数只能减少不能增加 | M2 起每阶段 |
| uid 唯一性 | 全库 uid 无重复、无空值 | M3 起 |
| 关系合法性 | `relations.type` 全部在闭集内；`prerequisite_of` 无环 | M4 起 |
| LFS 完整性 | `git lfs ls-files` 数量与 M0 基线一致 | 每阶段 |
| 孤儿不增加 | 孤儿数只能减少 | M6 起 |

### 6.2 回滚矩阵

| 出问题的对象 | 回滚方式 | 成本 |
|---|---|---|
| `derived/**` | `rm -rf derived/` + 重新生成 | 分钟级 |
| sidecar | 直接删除（可从 front matter + Git 重建，除 `previous_paths`） | 分钟级 |
| 单个簇的迁移 | `git revert <该簇的提交>` | 分钟级 |
| uid 注入 | `git revert`（纯 front matter 提交，与正文提交分离） | 分钟级 |
| nightly 分支的全部改动 | **删分支即可**（永不合并 main） | 秒级 |
| 整个 V2 | `git reset` 到 tag `v1-final`（仅在灾难场景，需用户明确指令） | 分钟级 |

> **分层提交是回滚能力的基础**（V2 §17.2）：`chore(source)` / `docs(knowledge)` / `chore(meta)` / `chore(derived)` / `chore(review)` 五类分开提交，才能做到「只回滚元数据，不回滚知识」。

---

## 7. 仍需用户确认的问题

按「影响面 × 越早定越好」排序。每项都给了**推荐默认值**——如果认同默认，回复「按默认执行」即可。

| # | 问题 | 选项 | 推荐默认 | 影响的阶段 |
|---:|---|---|---|---|
| Q1 | **`docs/` 与 `kb/` 的最终关系**：长期共存，还是最终全部迁入 `kb/embodied-ai/`？ | (a) 长期共存，使用时才迁 (b) 设定期限全量迁完 | **(a)**——V1 的「使用时迁移」原则已被验证有效，且全量迁移会产生一次巨大 diff | M7 |
| Q2 | **是否接受 sidecar（方案 C）**，还是坚持纯 front matter（方案 A）+ 仅靠提交分层解耦 diff？（注：双轴 status 与 durable/derived 划分已与 sidecar 决策解耦，二者正交） | (a) 方案 C (b) 方案 A | **(a)**——但如果你重视「一个文件看到全部信息」，方案 A + 严格提交分层也能工作，代价是 living document 的 diff 噪声 | M3 起 |
| Q3 | **uid 注入批次**：63 个文件一次性注入，还是按簇分 5–6 批？ | (a) 分批 (b) 一次性 | **(a)**——一次性会产生一个 63 文件的 diff，污染 `git blame`，且错了不好定位 | M3 / M4 |
| Q4 | **`RESEARCH_MAP.md` 何时分解**：现在（M6），还是等 `kb/` 有实际内容后？ | (a) M6 按计划 (b) 推迟到 M7 之后 | **(a)**——五个开放问题现在就无法被引用，这是当前最实际的损失 | M6 |
| Q5 | **`01_foundations/reinforcement_learning/` 归哪个领域？** | (a) `embodied-ai`（服务于 VLA 后训练） (b) `machine-learning`（RL 是通用方法） | **(b)**——它同时被 VLA 和通用 ML 使用，放 `machine-learning` 更符合 T 型结构；用 `relations: prerequisite_of` 连回 embodied-ai | M7 |
| Q6 | **nightly 是定时运行还是手动触发？**（注：本轮已确立 single-flight，定时触发下仍需保证「同一时刻仅一个 unresolved 运行」） | (a) 手动触发 (b) 定时（需调度器） | **(a)**——MVP 阶段先手动跑，观察若干次；定时会让「没人看报告」的风险提前到来 | M5 |
| Q7 | **`PAPER_MATRIX.md` 若无法完整派生怎么办？** | (a) 降级为 `synthesis.topic`，承认它是人的判断 (b) 强行拆成元数据字段 | **(a)**——如果矩阵里有对比判断，那它就不是索引 | M3（这是 M3 的关键检验） |
| Q8 | **`sources/files/` 里的 PDF 是入 LFS 还是 gitignore？** | (a) 继续 LFS（现状，9 个对象） (b) 改为 gitignore + 只留 registry | **(a)**——现状已验证可用，改动无收益且有风险 | M4 / M7 |
| Q9 | **`MIGRATION_BACKLOG.md` / `MIGRATION_CHECKLIST.md` 的归档时机与形态？** | (a) 保持原状不动 (b) 完成后移入 `archive/` | **(a)**——它们还在服役（历史材料迁移未完成） | 全程 |
| Q10 | **MVP 停在哪一步？**（术语：Foundation MVP = M2 只读健康层；Curator Dry-run MVP = M5） | (a) M2（Foundation MVP）后暂停两周 (b) 直接做到 M3 (c) 一路做到 M5（Curator Dry-run MVP） | **(a)**——M2 零风险、收益立刻可见，是最合适的观察点 | 全局 |
| Q11 | **本轮 4 份设计文档最终放哪？** | (a) 留在根目录 (b) 移入 `docs/_meta/` (c) 移入 `kb/agent-engineering/` | **(c)**——它们本身就是 agent-engineering 领域的知识，可作为该领域的首批内容 | M6 |
| Q12 | **`derived/` 是否入 Git？**（注：nightly 报告已独立为 `reports/`，不随 derived 入 Git 策略变化；`reports/` 单独决定） | (a) 入（GitHub 上可直接浏览索引） (b) 不入（本地生成） | **(a)**——个人库的一大价值是手机上打开 GitHub 就能查；用 `linguist-generated` 折叠 diff 噪声 | M2 |

---

## 8. 一句话计划摘要

> **先只读地把问题照亮（M0–M2），用 5 个文件验证方案（M3），把来源拆出来（M4），让机器空跑两周证明它判断得准（M5），然后才动用户的知识结构（M6），最后按需慢慢搬（M7）。**
>
> 全程 `docs/` 的正文一个字节都不改，nightly 分支永不合并 main——因此任何一步出错，回滚成本都趋近于零。

---

## 9. 更新记录

- 2026-08-07：建立 V2 迁移计划（Phase 1c）。定义 5 条约束、10 类先不迁内容、主/次试点及其选择理由、M0–M7 八个阶段（含目标 / 交付物 / 触碰文件 / 退出条件 / 回滚 / diff 预算）、自动化分级执行清单（A3/A2 十项、A0/A1 十五项）、7 项验证与 6 类回滚路径、12 个待用户确认问题（含推荐默认值）。**本轮为设计文档，未执行任何迁移。**
- 2026-08-07（rev1）：**同步 V2 架构提案的协议缺陷修正**。M2/M5 标题对齐术语（Foundation MVP = M2，Curator Dry-run MVP = M5）；M3 字段列表改为双轴 `document_state`/`knowledge_status` 且 sidecar 仅 durable；M5 与 §5.1 的 nightly 报告路径改为 `reports/nightly/`（非 Derived）；§1 约束 5 明确 human 可手动合并 nightly 分支；§7 的 Q2/Q6/Q10/Q12 补充本轮修正说明。**未执行任何迁移。**
