---
title: RLHF DPO PPO
description: 预训练+SFT 后模型"会说话"但不一定"说人想听的"。对齐用人类偏好数据把模型拉向有用/无害。RLHF 走"奖励模型+PPO"三段式;DPO 用一行损失绕过强化学习。枢纽见 大模型训练与微调。
tags:
- LLM
- alignment
- RLHF
- DPO
- imported-from-obsidian
aliases:
- RLHF
- DPO
- 对齐
created: 2026-06-28
---

# RLHF / DPO / PPO(偏好对齐)

> [!abstract] 一句话
> 预训练+SFT 后模型"会说话"但不一定"说人想听的"。对齐用**人类偏好数据**把模型拉向有用/无害。RLHF 走"奖励模型+PPO"三段式;**DPO** 用一行损失绕过强化学习。枢纽见 [大模型训练与微调](./大模型训练与微调.md)。

---

## 1. RLHF 三阶段
1. **SFT**:得到初始策略 $\pi_\text{ref}$。
2. **奖励模型 RM**:收集偏好对 $(x, y_w\succ y_l)$,按 **Bradley-Terry** 建模:
$$P(y_w\succ y_l)=\sigma\big(r(x,y_w)-r(x,y_l)\big)$$
训练 RM 最小化 $-\log\sigma(r_w-r_l)$。
3. **RL(PPO)**:用 RM 当奖励,优化策略:
$$\max_\pi \mathbb{E}_{x,y\sim\pi}\big[r(x,y)\big]-\beta\,\mathrm{KL}\big(\pi(\cdot|x)\,\|\,\pi_\text{ref}(\cdot|x)\big)$$
KL 项防止跑偏(reward hacking)。

## 2. PPO 要点
- **Actor-Critic**:策略(actor)+ 价值函数(critic 估优势)。
- **Clip**:限制单步策略更新幅度 $\min(\rho A,\text{clip}(\rho,1\pm\epsilon)A)$,$\rho$=新旧策略比,防训练崩。
- 实操复杂:要同时跑 actor/critic/RM/ref 四个模型,显存与稳定性都难。

## 3. DPO —— 绕过 RM 和 RL(推导)⭐
**关键洞察**:上面带 KL 的目标有**闭式最优解**:
$$\pi^*(y|x)=\frac1{Z(x)}\pi_\text{ref}(y|x)\exp\!\Big(\tfrac1\beta r(x,y)\Big)$$
反解出奖励:
$$r(x,y)=\beta\log\frac{\pi^*(y|x)}{\pi_\text{ref}(y|x)}+\beta\log Z(x)$$
代入 Bradley-Terry($Z(x)$ 在差值中**抵消**),偏好概率只剩策略比:
$$P(y_w\succ y_l)=\sigma\!\Big(\beta\log\tfrac{\pi(y_w|x)}{\pi_\text{ref}(y_w|x)}-\beta\log\tfrac{\pi(y_l|x)}{\pi_\text{ref}(y_l|x)}\Big)$$
于是直接对策略做**监督式**最大似然,得 **DPO 损失**:
$$\mathcal{L}_\text{DPO}=-\mathbb{E}_{(x,y_w,y_l)}\log\sigma\!\Big(\beta\log\tfrac{\pi_\theta(y_w|x)}{\pi_\text{ref}(y_w|x)}-\beta\log\tfrac{\pi_\theta(y_l|x)}{\pi_\text{ref}(y_l|x)}\Big)$$
> **直觉**:抬高 chosen 相对 ref 的对数概率、压低 rejected 的,$\beta$ 控制偏离 ref 的强度。**无需训练 RM、无需采样、无需 RL**,只要偏好对 + 两个模型(策略 + 冻结 ref)的前向。稳定、好实现 → 已成开源对齐主流。

## 4. 谱系
- **RLAIF**:用 AI(强模型)代替人标偏好,降成本。
- **Constitutional AI**:用一套"宪法"原则让模型自我批判改写,生成偏好数据。
- **DPO 变体**:IPO(防过拟合)、KTO(用好/坏单点标注而非成对)、SimPO(去掉 ref 模型)。

## 5. 对比
| | RLHF(PPO) | DPO |
|---|---|---|
| 组件 | SFT+RM+PPO(4 模型) | 策略 + 冻结 ref |
| 稳定性 | 难调 | 稳 |
| 表达力 | 在线探索更强(上限高) | 离线,依赖数据覆盖 |
| 工程 | 重 | 轻 |

## 6. 自检
> [!question]
> - [ ] RLHF 三阶段;Bradley-Terry RM
> - [ ] PPO 的 KL 约束 + clip 作用
> - [ ] DPO 推导:KL 目标闭式解 → 反解 r → Z 抵消 → DPO loss
> - [ ] DPO 为何无需 RM/RL
> - [ ] RLAIF / Constitutional AI / DPO 变体

## 相关
[大模型训练与微调](./大模型训练与微调.md) · [采样与解码策略](./采样与解码策略.md) · [大模型推理与部署](./大模型推理与部署.md)
