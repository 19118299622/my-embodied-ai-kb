---
id: embodied-data-quality-paper-matrix
title: PAPER_MATRIX
kind: topic-note
domain: embodied-ai
created: 2026-07-31
updated: 2026-08-08
---
- 资源核验日期：2026-07-31
- 人类入口：[README](../../../../README.md)
- 研究地图：[具身智能研究地图](../../../../docs/RESEARCH_MAP.md)
- 关联项目（外部）：[数据质量评估（VLA/ 原文件夹）](../../../../../数据质量评估/)

# PAPER_MATRIX

> **2026-07-21 状态：** 27 篇 PDF 的主题定位与摘要级条目维护在 `docs/具身数据集质量评估-调研地图.md`；本矩阵保留已进入可追溯深读的论文。当前 L1 论文为 DemInf，L3 参考论文为 VLA 数据集综述。地图的目录、术语附录与正文锚点已统一到该唯一入口。后续每完成一篇论文的 TRIAGE 或 L1 阅读，更新对应行而非另建平行表格。

| 论文 | 状态 | 地图主分类 | 阅读层级 | 质量定义 | 粒度 | 输入 | 方法类型 | 输出 | 下游验证 | 实机条件与限制 |
|---|---|---|---|---|---|---|---|---|---|---|
| Robot Data Curation with Mutual Information Estimators | L1 首轮完成：待主张—证据审计 | A：直接质量评估与数据筛选 | L1：核心精读 | (I(S;A)=H(A)-H(A\mid S))：动作多样性与条件可预测性的折中 | state-action 估计，轨迹聚合 | 状态、动作 chunk，经独立 VAE 压缩 | VAE + KSG k-NN MI + 轨迹筛选 | state-action 分数、轨迹分数、过滤子集 | RoboMimic、Franka、RoboCrowd；BC/ACT/Diffusion Policy | 假设轨迹成功；可能偏好停顿；不显式建模长程动力学；多任务/语言未解 |
| Vision-Language-Action in Robotics: A Survey of Datasets, Benchmarks, and Data Engines | TRIAGE 完成：仅作参考（定义评估维度时可选读） | E：综述与参考地图 | L3：参考与验证 | 资源级质量画像：保真度、具身/动作对齐、多模态完整性、任务/环境覆盖、grounding/验证；非可运行评分器 | 数据集/基准/数据引擎级 | 文献与代表性 VLA 资源 | 综述分类法与比较框架 | 分类体系、表格、开放问题 | 无新数据选择实验；引用既有基准/数据引擎结果 | 未给可操作质量指标、阈值或与下游性能的因果验证 |


## 更新记录

- 2026-07-31：从 `VLA/数据质量评估/PAPER_MATRIX.md` 迁移至本知识库，补充 YAML 元数据与相对链接；正文内容保持不变。
