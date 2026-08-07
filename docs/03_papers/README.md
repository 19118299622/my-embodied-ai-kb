# 03 Papers

该目录用于单篇论文阅读笔记。

## 当前条目

| 名称 | 状态 | 最近更新 | 说明 |
|---|---|---|---|
| [Data Quality in Imitation Learning 阅读笔记](./data_quality_in_imitation_learning/Data_Quality_in_Imitation_Learning_阅读笔记.md) | seed | 2026-07-31 | 动作一致性、转移多样性；IL 数据质量理论基线 |
| [Data Assessment for Embodied Intelligence 阅读笔记](./data_assessment_for_embodied_intelligence/Data_Assessment_for_Embodied_Intelligence_阅读笔记.md) | active | 2026-07-31 | 数据集级多样性熵与可学习性 |
| [Robot Data Curation with Mutual Information Estimators 阅读笔记](./robot_data_curation_with_mutual_information_estimators/Robot_Data_Curation_with_Mutual_Information_Estimators_阅读笔记.md) | active | 2026-07-31 | 状态—动作互信息估计与轨迹筛选 |
| [A Survey of Datasets, Benchmarks, and Data Engines 阅读笔记](./a_survey_of_datasets/A_Survey_of_Datasets_阅读笔记.md) | seed | 2026-07-31 | VLA 数据资源综述（REFERENCE_ONLY） |
| [RIPT-VLA 阅读笔记](./ript_vla/RIPT-VLA_阅读笔记.md) | active | 2026-07-31 | 交互式后训练；DS-LOOP（RLOO + PPO + 动态采样）；稀疏二元奖励 |
| [RL_and_VLA 论文索引](./rl_and_vla/INDEX.md) | seed | 2026-07-31 | 仅建索引（PDF 不纳入 git）；ConRFT / VLA-RL / RL→VLA 泛化实证 等 5 PDF |
| [SimpleVLA_RL 论文索引](./simplevla_rl/INDEX.md) | seed | 2026-07-31 | 仅建索引（PDF 不纳入 git）；SimpleVLA-RL（arXiv 2509.09674） |
| [AR-VLA 阅读笔记](./ar_vla/AR-VLA_阅读笔记.md) | active | 2026-08-03 | 真自回归动作专家；KV cache 跨时间因果生成；异步 V-L-A 模态同步 |
| [X-VLA 阅读笔记](./x_vla/X-VLA_阅读笔记.md) | active | 2026-08-03 | Soft Prompt 跨具身 VLA；Flow Matching 动作生成；0.04% 参数吸收异构性 |
| [D6 失败检测与失败数据分析（9 篇）](./d6_failure_detection/INDEX.md) | active | 2026-08-03 | A-D6 维度；AHA/Guardian/RoboFAC/REFLECT/Foresight/KITE/Robometer/MotIF/Handover-Failure；PDF 仅索引 |
| [LingBot 系列（6 篇 + 汇总）](./lingbot/INDEX.md) | active | 2026-08-03 | LingBot-VA/VLA/World/Depth/Map + Qwen-VLA；因果世界建模 / 务实 VLA / 开源世界模型 / 深度 / 三维重建 |

`lingbot/` 目录为 LingBot 体系多篇深度分析（同一家族，平铺同目录）：

| 名称 | 类型 | 说明 |
|---|---|---|
| [LingBot-VA 深度分析](./lingbot/analysis_LingBot_VA.md) | paper-note | 因果世界建模用于机器人控制（arXiv 待核验） |
| [LingBot-VLA 深度分析](./lingbot/analysis_LingBot_VLA.md) | paper-note | 务实 VLA 基础模型（2601.18692v2） |
| [LingBot-World 深度分析](./lingbot/analysis_LingBot_World.md) | paper-note | 开源世界模型（无 arXiv，项目站） |
| [LingBot-Depth 深度分析](./lingbot/analysis_LingBot_Depth.md) | paper-note | 掩码深度建模（无 arXiv，项目站+代码） |
| [LingBot-Map 深度分析](./lingbot/analysis_LingBot_Map.md) | paper-note | 流式三维重建（无 arXiv，项目站+代码） |
| [Qwen-VLA 深度分析](./lingbot/analysis_Qwen_VLA.md) | paper-note | 跨具身统一 VLA（2605.30280v1） |
| [LingBot 系列汇总分析](./lingbot/analysis_summary.md) | topic | 五篇 + Qwen-VLA 横向对比 |

纯 PDF 索引（无用户笔记，仅 arXiv 核验 + 一句话；PDF 不纳入 git）：

| 名称 | 模型 | arXiv | 一句话 |
|---|---|---|---|
| [OpenVLA 索引](./openvla/INDEX.md) | OpenVLA | 2406.09246 | 7B 开源 VLA，OpenX 970k 演示 |
| [π0 索引](./pi0/INDEX.md) | π0 | 2410.24164 | Physical Intelligence，VLM + Flow Matching |
| [RT-1 索引](./rt1/INDEX.md) | RT-1 | 2212.06817 | Google，RT 系列开山 |
| [SmolVLA 索引](./smolvla/INDEX.md) | SmolVLA | 2506.01844 | HuggingFace LeRobot，0.5B 紧凑 VLA |
| [TriVLA 索引](./trivla/INDEX.md) | TriVLA | 2507.01424 | 三系统 + 情景世界模型 |
| [UD-VLA 索引](./ud_vla/INDEX.md) | UD-VLA | 2511.01718 | 联合离散去噪扩散（JD3P） |
| [VLA-Adapter 索引](./vla_adapter/INDEX.md) | VLA-Adapter | 2509.09372 | 0.5B 轻量桥接 VL→A |
| [OpenVLA-OFT 索引](./openvla_oft/INDEX.md) | OpenVLA-OFT | 2502.19645 | OFT 微调配方，LIBERO 76.5%→97.1% |
| [π0.5 索引](./pi05/INDEX.md) | π0.5 | 2504.16054 | Physical Intelligence，开放世界泛化 |

`ript_vla/` 目录内的配套分析（同一论文的深入拆解，不单列为论文笔记）：

| 名称 | 类型 | 说明 |
|---|---|---|
| [GRPO 与 RLOO/DS-LOOP 对比分析](./ript_vla/RIPT-VLA_GRPO_vs_RLOO_对比分析.md) | topic | 十维度对比；组内相对优势估计谱系 |
| [PPO 裁剪参数 ε 调优分析](./ript_vla/RIPT-VLA_PPO_clip_epsilon_调优分析.md) | topic | ε 语义与 N／K／动态采样的联动（多为未验证推演） |
| [Rollout 存储与 On/Off-Policy 问答](./ript_vla/RIPT-VLA_Rollout存储与OnOffPolicy_问答.md) | topic | 工程细节问答六段；显存估算为量级推算 |

建议：

- 一篇论文对应一个主笔记；
- 同一论文的修订优先更新原文件；
- 不把多个弱相关论文塞入同一篇阅读笔记；
- 论文链接、项目主页和官方代码需要单独记录；
- 作者报告结果和个人判断必须区分。

模板：

- [`../../templates/PAPER_NOTE_TEMPLATE.md`](../../templates/PAPER_NOTE_TEMPLATE.md)
