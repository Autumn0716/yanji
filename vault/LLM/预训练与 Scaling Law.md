---
title: 预训练与 Scaling Law
description: 预训练 = 在海量语料上做 next-token prediction,用自监督把世界知识压进参数。Scaling Law 告诉你"给定算力预算,模型多大、数据多少最优"。枢纽见 大模型训练与微调。
tags:
- LLM
- pretraining
- scaling-law
- imported-from-obsidian
aliases:
- Scaling Law
- 预训练
created: 2026-06-28
---

# 预训练与 Scaling Law

> [!abstract] 一句话
> 预训练 = 在海量语料上做 **next-token prediction**,用自监督把世界知识压进参数。Scaling Law 告诉你"给定算力预算,模型多大、数据多少最优"。枢纽见 [大模型训练与微调](./大模型训练与微调.md)。

---

## 1. 目标:自监督 next-token
最大化语料似然 = 最小化交叉熵:
$$\mathcal{L}=-\frac1N\sum_{t}\log p_\theta(x_t\mid x_{<t})$$
- 无需人工标注,语料本身即监督信号 → 可扩到 TB 级。
- 困惑度 $\text{PPL}=e^{\mathcal L}$ 是常用度量。

## 2. 算力账:6N
每 token 训练 FLOPs $\approx 6N$($N$=参数量;前向 2N + 反向 4N),推理仅 2N(见 [KV Cache 与 FLOPs 推导](./题型精讲/KV Cache 与 FLOPs 推导.md))。总算力 $C\approx 6ND$($D$=训练 token 数)。

## 3. Scaling Law
**幂律**:loss 随参数 $N$、数据 $D$、算力 $C$ 幂律下降:$L(N)\approx (N_c/N)^{\alpha}+L_\infty$。
- **Kaplan(2020)**:偏重把算力投给**更大模型**。
- **Chinchilla(2022)**:修正——在固定算力 $C$ 下,**$N$ 与 $D$ 应同比增长**,经验 **$D\approx 20N$**(每参数约 20 token)。
$$N_\text{opt}\propto C^{0.5},\quad D_\text{opt}\propto C^{0.5}$$
> Chinchilla 结论:很多早期大模型**训练数据不足**(欠训练)。70B 应配 ~1.4T token。

## 4. 推论(对部署的影响)
- **小而久训** 比 **大而欠训** 在推理时更划算:同等能力下参数更少 → 推理显存/带宽更省。这是 Llama 系"小模型多喂数据"的依据,直接利好 [推理成本](./大模型推理与部署.md)。
- 推理算力会被**长期摊薄**:训练一次,推理无数次 → 倾向"训练多花、换推理便宜"。

## 5. 数据工程
- **质量 > 数量**:去重、清洗、过滤低质;高质量数据可"超越" Chinchilla 比例多训。
- **配比(data mixture)**:代码/多语言/数学比例影响能力。
- **课程(curriculum)**:由易到难、退火阶段提高优质数据比例。

## 6. 涌现能力
某些能力(多步推理、in-context learning)在参数/数据越过阈值后**突然出现**,小模型上看不到 → Scaling 的质变动机。

## 7. 自检
> [!question]
> - [ ] next-token 交叉熵目标;PPL
> - [ ] 6N 训练 vs 2N 推理;C≈6ND
> - [ ] Kaplan vs Chinchilla;D≈20N
> - [ ] "小而久训"为何利好推理成本
> - [ ] 数据质量/配比/课程
> - [ ] 涌现能力

## 相关
[大模型训练与微调](./大模型训练与微调.md) · [KV Cache 与 FLOPs 推导](./题型精讲/KV Cache 与 FLOPs 推导.md) · [大模型推理与部署](./大模型推理与部署.md) · [优化器与训练稳定性](./优化器与训练稳定性.md)
