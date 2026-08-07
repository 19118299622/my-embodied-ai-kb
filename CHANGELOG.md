# Changelog

本文件记录项目结构、研究地图和核心专题的重要变化。单篇论文中的普通文字修订不必逐项记录。

## 2026-07-31

### Added

- 初始化 `my-embodied-ai-kb` 项目结构；
- 新增根目录 `README.md`；
- 新增 Agent 维护规范 `AGENTS.md`；
- 新增研究地图 `RESEARCH_MAP.md`；
- 新增历史材料迁移清单 `MIGRATION_BACKLOG.md`；
- 新增专题观察《VLA 实时控制、执行验证与在线学习》；
- 新增论文笔记、专题、实验、想法和双文档学习模板；
- 新增 `.gitignore` 与 `.gitattributes`。

### Changed

- 项目定位由“具身智能知识库”调整为“具身智能研究实验室”；
- 明确不宣称已收录过去所有知识；
- 采用“优先新增文档，次选扩写已有文档”的维护策略；
- 论文资源索引增加官方代码、项目主页和核验日期字段。

## 2026-07-31（历史材料迁移 · 第一批试点）

### Added

- 治理整备：为 `docs/` 下 5 个目录（01/03/04/05/06）补齐「当前条目」索引表；
- 统一论文笔记模板：`AGENTS.md` §4.1 对齐 `templates/PAPER_NOTE_TEMPLATE.md`（15 节、含模型或数据/核验日期、默认 `status: seed`）；
- 新增 `templates/REPORT_TEMPLATE.md`；
- 配置 Git LFS 管理 PDF（`.gitattributes` 中 `*.pdf` 走 LFS）；
- 在 `README.md` 固化目录放置约定、status 初始值与 tags 受控词表；
- 迁入专题《实机机器人数据质量评估总览》及 4 篇支撑文档（L1 总结、分层阅读计划、PAPER_MATRIX、READING_LOG）；
- 迁入 4 篇论文笔记（Data Quality in IL、Data Assessment、DemInf、VLA 数据综述）及对应 PDF（LFS）；
- 迁入 4 份进度汇报 PPTX（LFS）至 `docs/06_reports/数据质量评估进度汇报/`。

### Changed

- `MIGRATION_BACKLOG.md`：实机数据质量评估（迁移中→已完成）、数据质量论文地图（待迁移→部分完成）；
- `RESEARCH_MAP.md` §2.1 挂接当前专题链接，§5 前 4 项标注「已产出」。

## 2026-07-31（历史材料迁移 · Phase 3a RIPT-VLA）

### Added

- 迁入 RIPT-VLA 单论文知识包 `docs/03_papers/ript_vla/`：
  - 主笔记《RIPT-VLA 阅读笔记》（15 节 paper-note，含 YAML 元数据与资源核验）；
  - 3 篇配套拆解（同一论文深入，不单列为论文笔记）：GRPO vs RLOO/DS-LOOP 对比、PPO 裁剪参数 ε 调优、Rollout 存储与 On/Off-Policy 问答；
  - 论文 PDF `RIPT-VLA_2505.17016v1.pdf`（4.8 MB，Git LFS）。
- `docs/03_papers/README.md` 增加 RIPT-VLA 条目与配套分析说明表。

### Verified

- 论文版本：arXiv 2505.17016 仅 v1（2025-05-22 提交）；
- 项目主页 `ariostgx.github.io/ript_vla/`：截至 2026-07-31 返回 GitHub Pages 404（疑似下线）；
- 官方代码 `github.com/Ariostgx/ript-vla`：README 自述 Official implementation，168 stars，最后推送 2025-06-04；**仓库未声明许可证（license=null），商用/再分发存在版权风险**；
- 实验数字均为论文作者报告，本仓库尚未独立复现。

### Changed

- `RESEARCH_MAP.md` §2.4 增加 RIPT-VLA 代表工作链接，§5 增加第 7 项近期产出；
- `MIGRATION_BACKLOG.md` §6 增加 Phase 3 启动记录；
- `MIGRATION_CHECKLIST.md` §7 Phase 3 状态由「待启动」改为「进行中（3a 完成，3b 启动）」。

> 备注：按用户约定，本次仅 `git add` 追踪（含 PDF 走 LFS），未执行 `git commit`。

## 2026-07-31（历史材料迁移 · Phase 3b–3e RL 与 VLA 后训练）

### Added

- `docs/01_foundations/reinforcement_learning/`（双文档学习模式）：
  - `学习规划与进度.md`：RL 基础路线规划，锚定用户 `spinningup-simplified` PPO 实现；
  - `长期学习笔记.md`：阶段一四讲（MDP / 策略梯度与 REINFORCE / GAE 优势估计 / PPO 精解），以用户 PPO 实现超参（gamma=0.99, lam=0.97, clip=0.2, target_kl=0.03）为代码锚点，并与 RIPT-VLA 笔记交叉引用。
- `docs/02_topics/vla_post_training/VLA后训练中的奖励与信用分配.md`：横向比较 RIPT-VLA / VLA-RL / ConRFT / SimpleVLA-RL / GRPO-RLOO / 泛化实证六条路线，区分结论证据强度（作者报告 / 已发表实证 / 个人判断）。
- `docs/03_papers/rl_and_vla/INDEX.md`、`docs/03_papers/simplevla_rl/INDEX.md`：纯索引（PDF 不纳入 git）；联网识别 ConRFT(2502.05450)、VLA-RL(2505.18719)、RL→VLA 泛化实证(2505.19789)、SimpleVLA-RL(2509.09674)，RPD / SLIM 标题待核验。
- `docs/04_experiments/PPO_CartPole_可复现实验记录.md`：固化用户 PPO/CartPole 实现的代码版本、环境、超参；训练结果待从本地 `runs_cartpole` 日志回填。

### Changed

- `docs/01_foundations/README.md`、`docs/02_topics/README.md`、`docs/03_papers/README.md`：新增条目；
- `RESEARCH_MAP.md` §2.4 增加 VLA 后训练专题链接，§5 第 5 项标注「已产出」；
- `MIGRATION_BACKLOG.md` §6 增加 Phase 3b–3e 记录；`MIGRATION_CHECKLIST.md` §7 Phase 3 维持「进行中（3a 完成，3b 启动）」。

> 备注：PDF（RIPT-VLA、各数据质量论文）走 Git LFS；RL_and_VLA / SimpleVLA_RL 的 PDF 仅建索引、不纳入 git。本次仅 `git add` 追踪，未 `git commit`。

## 2026-08-03（历史材料迁移 · Phase 3c 完成 + Phase 4 启动）

### Added

- `docs/02_topics/action_chunk_flow_matching/动作块与流匹配动作生成.md`：动作块与流匹配动作生成专题（type: topic）。基于用户 `ACT/ACT梳理.docx` 与 4 篇已核验论文（ACT 2304.13705、动作分块理论 Zhang et al. 2025-11-27、Diffusion Policy 2303.04137、π0 2410.24164）；范式演进与权衡标为综合判断（未验证）。
- `docs/03_papers/ar_vla/AR-VLA_阅读笔记.md`：由用户 2 篇本地笔记（AR-VLA论文总结、AR-VLA架构分析）迁移改写，规范 paper-note（15 节 + YAML + 核验头）。
- `docs/03_papers/x_vla/X-VLA_阅读笔记.md`：由用户本地深度分析笔记迁移改写，规范 paper-note（15 节 + YAML + 核验头）；跨具身 + Flow Matching。

### Verified

- AR-VLA：arXiv 2603.10126v2（RSS 2026 accepted），项目主页 arvla.insait.ai 可访问；代码以 LeRobot 策略包（github.com/utomm/AR-VLA-lerobot）发布。
- X-VLA：arXiv 2510.10274v1（2025-10-11，preprint）；用户笔记标注 ICLR 2026，但 arXiv 仅标 preprint，截至 2026-08-03 未确认正式 venue。
- 实验数字均为作者报告，本仓库尚未独立复现。

### Changed

- `docs/03_papers/README.md`、`docs/02_topics/README.md`：新增 AR-VLA、X-VLA 与动作块与流匹配专题条目；
- `RESEARCH_MAP.md` §2.4 增加 AR-VLA、X-VLA、动作块与流匹配代表工作，§5 增加第 8–10 项近期产出；
- `MIGRATION_CHECKLIST.md` §7 Phase 3 改「已完成」、Phase 4 改「进行中（AR-VLA、X-VLA 已迁移）」；
- `MIGRATION_BACKLOG.md` §6 增加 Phase 3 收尾 + Phase 4 启动记录。

> 备注：本次仅 `git add` 追踪，未 `git commit`。ACT / action_chunk / Diffusion Policy / π0 的 PDF 与 PPTX 仅在专题中建立索引（路径引用），未纳入 git；RIPT-VLA 等已迁 PDF 仍走 Git LFS。

## 2026-08-03（续 · 数据质量 D6 新增 + Phase 4 收尾 + Phase 5 种子）

### Added

- **数据质量 D6 失败检测（9 篇）**：`VLA/数据质量评估/notes/D6-*.md`（8-3 新建）迁入 `docs/03_papers/d6_failure_detection/`，保留用户内容 + 规范化 YAML（type/status/created/updated/tags/来源/PDF 索引）+ 迁移核验头；建 `INDEX.md`（PDF 仅索引，本地路径见源仓库 `VLA/数据质量评估/papers/`）。
  - 9 篇：AHA(2410.00371, ICLR2025)、Foresight(2606.23085)、Guardian(2512.01946)、KITE(2604.07034, ICRA2026)、MotIF(2409.10683)、Handover-Failure(2402.18319, ICRA2024)、REFLECT(2306.15724, CoRL2023)、RoboFAC(2505.12224)、Robometer(2603.02115, RSS2026)。
- **LingBot 系列（7 篇）**：`VLA/lingbot/analysis_*.md` 迁入 `docs/03_papers/lingbot/`（LingBot-VA/VLA/World/Depth/Map + Qwen-VLA + 汇总），加 YAML + 迁移头；建 `INDEX.md`。
- **纯 PDF 模型索引（9 个）**：`docs/03_papers/{openvla,pi0,rt1,smolvla,trivla,ud_vla,vla_adapter,openvla_oft,pi05}/INDEX.md`，arXiv 全部联网核验。
- **Phase 5 种子**：`docs/02_topics/world_model/世界模型与跨具身_VLA_种子.md`，串接 LingBot-World/VA、DreamVLA、TriVLA、Foresight、X-VLA、Qwen-VLA、VLA_Aerial；建 `docs/03_papers/{dreamvla,vla_aerial}/INDEX.md`。

### Verified

- D6 四篇 2025-12 / 2026 arXiv（Foresight 2606.23085、Guardian 2512.01946、Robometer 2603.02115、KITE 2604.07034）均确认可访问。
- 9 个纯 PDF 模型 arXiv 全部核验；**纠正 SmolVLA 误标**——源文件名不含 ID，原猜测 2506.00944 实为高能物理论文，正确为 **2506.01844**（HuggingFace LeRobot）。
- LingBot-VA 笔记记录的 arXiv 为 `submit/7211108`（非标准投稿追踪号），正式 ID 待核验；LingBot-World/Depth/Map 无 arXiv（仅项目站）。
- VLA_Aerial 两篇（VLA-AN / π0-Aerial）arXiv / 出处**待核验**（文件名未含 ID）。
- 所有实验数字均为作者报告 / 公开资料，本仓库尚未独立复现。

### Changed

- `docs/02_topics/embodied_data_quality/实机机器人数据质量评估总览.md`：补 **A-D6 失败检测与失败数据分析** 专节（维度表 + 目录同步），链接 9 篇 D6 笔记。
- `docs/03_papers/README.md`、`docs/02_topics/README.md`：新增 D6、LingBot、9 纯 PDF 模型、world_model 条目。
- `RESEARCH_MAP.md` §2.3 世界模型、§2.6 数据生成与跨形态迁移补「代表工作（已迁移）」；§5 近期产出增至第 16 项。
- `MIGRATION_BACKLOG.md` §3 状态表：VLA 后训练 / 视觉—语言—动作模型 / 世界模型基础 改「部分完成」；§6 追加更新记录。
- `MIGRATION_CHECKLIST.md` §7：Phase 4 改「已完成」、Phase 5 改「进行中（种子级）」。

> 备注：本次仅 `git add` 追踪，未 `git commit`。**所有新增 PDF 均仅索引、不纳入 git**（默认边界：大型文件只记录索引）；RIPT-VLA 等此前已迁 PDF 仍走 Git LFS。
