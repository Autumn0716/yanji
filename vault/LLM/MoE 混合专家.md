---
title: MoE 混合专家
description: MoE 把 FFN 换成"多个专家 + 路由器,每 token 只激活 top-k 个专家",解耦参数量与计算量:总参数大(知识容量)、激活参数小(每 token 算力)。代价是推理时全部专家都要常驻显存,Memory-bound 更严重。主线见 大模型推理与部署,被替代的稠密 FFN 见 Transformer 架构
tags:
- LLM
- MoE
- inference
- imported-from-obsidian
aliases:
- Mixture of Experts
- 混合专家
created: 2026-06-28
---

# MoE 混合专家(Mixture of Experts)

> [!abstract] 一句话
> MoE 把 FFN 换成"多个专家 + 路由器,每 token 只激活 top-k 个专家",**解耦参数量与计算量**:总参数大(知识容量)、激活参数小(每 token 算力)。代价是推理时**全部专家都要常驻显存**,Memory-bound 更严重。主线见 [大模型推理与部署](./大模型推理与部署.md),被替代的稠密 FFN 见 [Transformer 架构](./Transformer 架构.md)。

---

## 1. 核心动机
稠密模型扩容 → 参数与算力同步涨。MoE 让"知识容量"与"单 token 算力"脱钩:
$$\text{总参数}=E\times(\text{单专家}),\quad \text{激活参数}\approx k\times(\text{单专家})\ (k\ll E)$$
> Mixtral 8×7B:总 ~47B,每 token 仅激活 ~13B(top-2)。质量近 47B,算力近 13B。

## 2. 结构与数学
第 $l$ 层 FFN → $E$ 个专家 $\{f_1..f_E\}$ + 门控路由:
$$g=\text{softmax}(W_r x)\in\mathbb{R}^E,\quad \mathcal{T}=\text{TopK}(g,k),\quad y=\sum_{i\in\mathcal{T}}\frac{g_i}{\sum_{j\in\mathcal{T}}g_j}f_i(x)$$
路由器是小线性层 $W_r\in\mathbb{R}^{d\times E}$;常 $k{=}2$。

## 3. 训练侧难点(决定推理形态)
- **负载均衡**:不约束会"赢者通吃"。加 **aux load-balancing loss**,或 **Expert-Choice routing**(专家选 token)天然均衡。
- **容量因子 cf**:每专家每 batch 限收 $C=\text{cf}\cdot\frac{kN}{E}$,超出 **drop**(走残差跳过);cf↑ 减 drop 但增显存/padding。
- **路由不可导**:top-k 离散,用 softmax 权重回传 / straight-through。

## 4. 推理特性(部署关键 ⭐)
| 维度 | 稠密 | MoE |
|---|---|---|
| 显存 | 总参数 | **总参数(所有专家都 load)** |
| 单 token 算力 | 全参 | 仅 top-k 激活 |
| 瓶颈 | 平衡 | **更 Memory-bound** |

> [!warning] 反直觉
> MoE 省的是 **FLOPs 不是显存**。8×7B 推理仍需 ~47B 权重进显存 → 部署核心矛盾是"装不下全部专家"。

**优化**:
- **专家并行 EP**:专家分卡,token 经 **all-to-all** 路由(主通信瓶颈,见 [大模型推理与部署](./大模型推理与部署.md) §VII.4)。
- **Expert Offloading**:冷专家放 CPU 按需换入。
- **Expert Caching/预取**:利用专家激活时间局部性。
- **Fused MoE kernel**:gather→分组 GEMM→scatter 融合(vLLM/TRT-LLM)。
- **量化**:专家权重量化收益最大(显存是瓶颈),见 [模型量化](./模型量化.md)。

## 5. 代表模型
| 模型 | 配置 |
|---|---|
| Mixtral 8×7B | E=8,k=2 |
| DeepSeek-MoE/V2/V3 | 细粒度专家 + 共享专家 + 高 E |
| Qwen-MoE | E=60+ 细粒度 |

> **细粒度 + 共享专家**(DeepSeek):专家切更小增数量提升组合灵活度;几个"共享专家"恒激活承载通用知识,降冗余。

## 6. 自检
> [!question]
> - [ ] 解耦总参/激活参;Mixtral 47B vs 13B
> - [ ] 路由 softmax + top-k 加权公式
> - [ ] 负载均衡(aux loss / Expert-Choice)+ 容量因子
> - [ ] 为何"省算力不省显存"→ 更 Memory-bound
> - [ ] EP all-to-all;专家 offload/caching

## 相关
[大模型推理与部署](./大模型推理与部署.md) · [Transformer 架构](./Transformer 架构.md) · [模型量化](./模型量化.md) · [注意力机制](./注意力机制.md) · [分布式训练与并行](./分布式训练与并行.md)
