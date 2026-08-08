---
id: embodied-data-quality-reading-log
title: READING_LOG
kind: report
domain: embodied-ai
created: 2026-07-31
updated: 2026-08-08
---
- 资源核验日期：2026-07-31
- 人类入口：[README](../../../../README.md)
- 研究地图：[具身智能研究地图](../../../../docs/RESEARCH_MAP.md)
- 关联项目（外部）：[数据质量评估（VLA/ 原文件夹）](../../../../../数据质量评估/)

# READING_LOG

## 2026-07-21

- 按读者导航需求重构调研地图：完整目录已移至标题之后并收录新增章节；正文中的术语表移至文末“附录 A：核心术语与英文缩写”；仅保留与数据质量方法直接相关的术语，不收录具身 AI、VLA、IL、BC、policy、rollout、Sim2Real 等基础背景概念。正文首次关键出现处链接到附录，术语条目以稳定锚点链接回对应方法或评估层级。[INFERENCE]
- 将 `docs/具身数据集质量评估-调研地图.md` 确认为唯一的跨论文技术体系入口，并开始从“论文分类索引”改造成“初学者可读的评估方法地图”。本轮新增：阅读路径、五个关键概念区分、具身／VLA／IL／轨迹／闭环等基础术语、七层评估层级、代表方法速查表和问题→方法选择表。[INFERENCE]
- 保留原有 A–E 治理分类、D1–D5 质量维度与论文条目；新增的七层框架只负责说明质量问题出现的位置，不替代原分类。明确将直接评估、训练端补救与外部验证分开，避免把 ADP、评测基准和质量评分器混为同类方法。[INFERENCE]
- 用户确认 `papers/` 中的 27 份本地 PDF 应进入 Git 追踪；`.gitignore` 已改为仅排除 `tmp/` 和本地工具状态。PDF 尚待与本轮文档修改一并提交到 `docs/data-quality-framework` 分支。

## 2026-07-17

- 新建 `docs/L1核心精读总结.md`，将已读的 `Data Quality in Imitation Learning` 与 `Data Assessment for Embodied Intelligence` 收敛为“局部可辨识—有效变化—覆盖—下游验证”的四层框架，并明确区分质量信号与数据分配决策。
- `Robot Data Curation with Mutual Information Estimators` 的 L1 首轮阅读完成：Joey Hejna 等，arXiv:2502.08623v3（2025-04-22），METHOD。DemInf 以状态 VAE、动作 chunk VAE 与 KSG k-NN 估计逐 state-action 的互信息贡献，再以轨迹时间均值排序筛选。[PAPER]
- 论文以 (I(S;A)=H(A)-H(A\mid S)) 同时约束动作多样性和条件可预测性；RoboMimic、Franka 与 RoboCrowd 上有人工标签与下游策略证据。[PAPER] 其前提包括轨迹成功、对停顿敏感、未显式处理长时程动力学。[PAPER]
- 代码状态：论文首页给出 `https://jhejna.github.io/demonstration-info`；本地未发现代码，官方仓库与提交尚未核验，未创建代码映射。
- 已将 DemInf 笔记扩展为方法精读版：新增 BC→MI 推导、KSG 邻居计数、具体状态/动作数据流、VAE/action-chunk 超参数、基线分数定义、数据规模与实机试验次数，并标记了实验协议中的混杂因素与未核验实现细节。
- 继续将 DemInf 笔记升级为公式精读版：逐项定义式 (1)–(5)、状态访问分布上界、条件动作熵、特权信息不等式、VAE 与 KSG 的全部符号；补充从 KL/密度比到 MI、从经验期望到样本贡献、从第 k 近邻到轨迹分数的推导，并逐式说明用途、设计理由和不能支持的结论。
- 重构公式排版与 KSG 讲解：所有数学表达统一为 Markdown/LaTeX 的 `$...$` 与 `$$...$$`；补充 KSG 全称、要解决的连续 MI 估计问题、k-NN 密度直觉、共享 joint 半径消除三项熵偏差的原因、完整 KSG-1 估计式、局部分数、合法数值例子、k 的偏差—方差权衡及高维边界，并加入 KSG 原论文来源。

## 2026-07-16

- 建立持续阅读日志。
- 当前阅读对象：`Robot Data Curation with Mutual Information Estimators`。
- 状态：第一轮尚未开始；计划仅读取摘要、Introduction、方法总览图、主要实验表格和 Conclusion。
- 代码状态：本地未发现实现仓库；官方仓库尚未验证。
- 下一检查点：摘要阅读结束后提出理解问题，再继续 Introduction。

- `ASurveyofDatasets.pdf` TRIAGE 完成（约 15 分钟）：Wang 等，arXiv:2604.23001v1，2026-04-24。结论为**仅作参考**：它是 VLA 数据集、基准和数据引擎的资源级综述，不提出可执行的数据质量分数或数据筛选实验。可复用的评估维度是保真度、具身/动作对齐、多模态完整性、任务/环境覆盖与 grounding/验证；这些维度属于本项目的推断，尚需轨迹级代理指标和下游验证。详见 `notes/ASurveyofDatasets_reading.md`。

- 对 `ASurveyofDatasets.pdf` 的引用链完成初筛：优先追踪 DROID、Open X-Embodiment/RT-X、MimicGen、COLOSSEUM、CALVIN；分别对应真实数据覆盖、跨具身动作对齐、生成数据验证、组合扰动鲁棒性与长时程评估。该综述未直接给出“轨迹级评分 + 数据筛选 + 下游收益验证”的被引核心方法，需在现有质量评估论文中另行筛选。

- 已将上述引用链及第二梯队先登记到 `docs/具身数据集质量评估-调研地图.md`：新增目录、按“直接质量评估 / 数据资源与采集质量 / 生成与扩增数据质量 / 外部验证与评测协议”的治理规则和 10 篇待下载条目。每篇条目已依据摘要及可访问的结论/局限确定主分类，下载前不新增重复的详细笔记。

- 已从公开 arXiv 下载并验证 10 篇入图文献：DROID、Open X-Embodiment、RH20T、MimicGen、GenSim、RoboGen、COLOSSEUM、CALVIN、SIMPLER、VLABench。全部可由 PDF 解析器读取（合计 281 页）；地图状态已更新为“已入库”。

- 完成 `papers/` → 调研地图同步审计：28 个 PDF 对应 27 个唯一主题条目，地图现已覆盖全部。新增 `IQA.pdf`（A 类：具身视觉输入质量）和 `ASurveyofDatasets.pdf`（E 类：综述与参考地图）；确认 `PSD.pdf` 与 `AnEfficientMetricforDataQualityMeasurementinImitation.pdf` 内容重复，仅作为别名记录。地图同时补充了同步范围与 E 类目录入口。

- 已按 `SURVEY` 卡片模板重写 `notes/ASurveyofDatasets_reading.md`：将快速判断、领域地图、可迁移框架和阅读路线前置；完整证据审计与引用链保留为附录。论文结论与阅读决策不变。

- 新增 `docs/分层阅读计划.md`，将已有调研地图的 A–E 主分类与 L0–L3 阅读层级分离：分类用于定位论文角色，层级用于决定当前阅读深度。L0 初筛已覆盖 27 个唯一主题条目；后续优先 L1 核心方法，随后按缺口定向阅读 L2，并以 D/E 类材料设计验证与回查。地图和 `PAPER_MATRIX.md` 已新增对应入口／字段。

- 经 SHA-256 复核后，删除与 `AnEfficientMetricforDataQualityMeasurementinImitation.pdf` 完全重复的 `papers/PSD.pdf`；`papers/` 现保留 27 个 PDF 工件，均对应唯一主题条目。


## 更新记录

- 2026-07-31：从 `VLA/数据质量评估/READING_LOG.md` 迁移至本知识库，补充 YAML 元数据与相对链接；正文内容保持不变。
