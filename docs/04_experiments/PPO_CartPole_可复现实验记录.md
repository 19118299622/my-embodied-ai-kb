---
title: PPO on CartPole/HalfCheetah 可复现实验记录
type: experiment
status: seed
created: 2026-07-31
updated: 2026-07-31
tags:
  - experiment
  - ppo
  - reinforcement-learning
  - reproducibility
---

# PPO on CartPole/HalfCheetah 可复现实验记录

> 目的：把用户已跑通的 PPO 实现固化成可复现实验记录，作为后续 VLA 后训练实验（RIPT-VLA / SimpleVLA-RL 风格）的对照基线。
> 结论强度：代码版本、环境、超参为**已确认（来自用户仓库 README 与 PPO.ipynb）**；训练结果为**待回填**（源：`VLA/RL/spinningup-simplified-main/runs_cartpole` 与 `runs/`）。

## 1. 研究问题

近端策略优化（PPO）在经典控制 / 连续控制基准上是否能稳定收敛？其裁剪目标 + 多轮更新 + KL 早停的组合对样本效率与稳定性有何影响？这构成理解 VLA 后训练（同样基于 PPO 骨架）的最低共识基线。

## 2. 可证伪假设

- H1：PPO（clip=0.2, target_kl=0.03）在 CartPole / HalfCheetah 上能稳定收敛到接近原 Spinning Up 基准；
- H2：若 `train_pi_iters`（内循环步数）过大，策略偏离 on-policy，回报会下降或崩溃（与 RIPT-VLA 中 $N$ 的约束对应）；
- H3：去掉 KL 早停（或把 target_kl 设为很大）会增加训练不稳定性。

## 3. 数据

- 数据名称：Gymnasium 内置环境（CartPole-v1 / HalfCheetah-v4）
- 数据版本：Gymnasium 0.29.1
- 数据划分：无（在线交互，无离线数据集）
- 数据路径：环境内生成，无需外部数据
- 质量检查：环境 reset/step 接口标准，无人工数据质量风险

## 4. 代码与环境

- 仓库：`VLA/RL/spinningup-simplified-main/`（用户本地，未复制进本库；仅关联引用）
- 关键文件：`PPO.ipynb`（PPO）、`vpg.ipynb`（对照 VPG/REINFORCE）
- 分支 / Commit：未记录（建议回填 commit 哈希以便复现）
- Python：3.8.20
- CUDA：CPU 即可（README 注明 CPU 运行，且支持 Windows）
- GPU：无强制要求
- 依赖：Torch 2.4.1、Scipy 1.10.1、Gymnasium 0.29.1、MuJoCo 3.1.6、Numpy 1.24.3、Pillow 10.4.0

## 5. 基线与实验组

- 基线：随机策略（回报约环境下限）；
- 实验组：PPO（裁剪目标）、VPG（REINFORCE+基线）作为对照；仓库另含 DDPG / TD3 / SAC 供横向对比。

## 6. 指标

- 每 epoch 平均回报（Average Episode Return）；
- TotalEnvInteracts（用户示例 250,000 = steps_per_epoch 5000 × epochs 50）；
- 策略与旧策略的 KL 散度（用于早停判断）；
- 价值损失与策略损失曲线。

## 7. 实验矩阵

| 算法 | 环境 | steps_per_epoch | epochs | clip | lam | target_kl | 备注 |
|---|---|---|---|---|---|---|---|
| PPO | HalfCheetah-v4 | 5000 | 50 | 0.2 | 0.97 | 0.03 | 用户跑通示例 |
| PPO | CartPole-v1 | 5000 | 50 | 0.2 | 0.97 | 0.03 | runs_cartpole 日志待回填 |
| VPG | CartPole-v1 | — | — | — | — | — | 对照（vpg.ipynb） |

## 8. 结果

> 待回填：从 `VLA/RL/spinningup-simplified-main/runs_cartpole/` 与 `runs/` 读取每 epoch 回报与 KL 曲线，填入下表。

| 算法 | 环境 | 最终平均回报 | 收敛 epoch | 备注 |
|---|---|---|---|---|
| PPO | HalfCheetah-v4 | 待回填 | 待回填 | GIF 见仓库 `gifs/ppo.gif` |
| PPO | CartPole-v1 | 待回填 | 待回填 | 待回填 |

## 9. 异常与失败记录

- 未发现明确失败（用户示例 GIF 显示 250k 步后策略可用）。具体异常待回填自 `runs/` 日志。

## 10. 结论边界

- 本记录仅覆盖经典基准，不直接外推到真实机器人 / VLA；但 PPO 的「裁剪 + 多轮 + KL 早停」机制与 RIPT-VLA 的 DS-LOOP、SimpleVLA-RL 一致，是后续 VLA 后训练实验的共识骨架。
- `train_pi_iters`、`target_kl` 与 RIPT-VLA 中内循环步数 $N$ 的等价/差异关系，需在 VLA 后训练专题中进一步澄清。

## 11. 下一步

1. 回填 `runs_cartpole` / `runs` 的实际回报与 KL 曲线；
2. 做 H2 消融：固定其他超参，扫描 `train_pi_iters ∈ {20, 80, 200}` 观察 on-policy 偏离；
3. 把该基线对照迁移到 VLA 后训练专题，与 RIPT-VLA / SimpleVLA-RL 的训练稳定性结论并列。

## 关联文档

- 基础笔记：[`../../kb/machine-learning/reinforcement-learning/长期学习笔记.md`](../../kb/machine-learning/reinforcement-learning/长期学习笔记.md)（第 4 讲 PPO 精解，已锚定本实验超参）
- VLA 后训练脉络：[`../../03_papers/ript_vla/RIPT-VLA_阅读笔记.md`](../../03_papers/ript_vla/RIPT-VLA_阅读笔记.md)

## 更新记录

- 2026-07-31：建立可复现实验记录骨架，固化代码版本、环境与超参；训练结果待从用户本地 `runs_cartpole` 日志回填。
