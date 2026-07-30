---
title: 训练并行 FSDP ZeRO
description: 训练显存远比推理大(还要存梯度 + 优化器状态 + 激活)。ZeRO/FSDP 把这些冗余分片到各卡,让万亿参数可训。是 分布式训练与并行 在训练侧的深化。
tags:
- LLM
- training
- parallelism
- ZeRO
- imported-from-obsidian
aliases:
- ZeRO
- FSDP
created: 2026-06-28
---

# 训练并行:FSDP / ZeRO

> [!abstract] 一句话
> 训练显存远比推理大(还要存梯度 + 优化器状态 + 激活)。**ZeRO/FSDP 把这些冗余分片到各卡**,让万亿参数可训。是 [分布式训练与并行](./分布式训练与并行.md) 在训练侧的深化。

---

## 1. 训练显存账(为什么爆)
混合精度 + Adam,每参数需:
| 项 | 字节 |
|---|---|
| FP16 权重 | 2 |
| FP16 梯度 | 2 |
| FP32 master 权重 | 4 |
| Adam 一阶 $m$(FP32) | 4 |
| Adam 二阶 $v$(FP32) | 4 |
| **合计** | **16 字节/参数** |

> 7B 模型仅"模型态"就 **112GB**(还没算激活)→ 单卡装不下。激活另算(随 batch×序列长)。

## 2. ZeRO(Zero Redundancy Optimizer)
数据并行下每卡本来存**全套**状态(冗余)。ZeRO 分阶段把冗余切到各卡:
| 阶段 | 分片什么 | 显存降幅 |
|---|---|---|
| **ZeRO-1** | 优化器状态($m,v$,FP32 master) | ~4× |
| **ZeRO-2** | + 梯度 | ~8× |
| **ZeRO-3** | + 参数本身 | ~线性于卡数 |

- 用时临时 **all-gather** 出完整参数/梯度,用完即弃 → 通信换显存。
- **ZeRO-Offload/Infinity**:再把状态卸到 CPU/NVMe(类比推理的 [KV offload](./大模型推理与部署.md))。

## 3. FSDP(Fully Sharded Data Parallel)
PyTorch 原生,等价 ZeRO-3:
- 参数/梯度/优化器状态按 **shard** 切到各 rank。
- 前向/反向**逐层 all-gather** 出完整权重算完即释放,梯度 **reduce-scatter** 回分片。
- 通信与计算可重叠(prefetch 下一层权重)。

## 4. 与 3D 并行组合
大规模训练叠加多维:
$$\text{DP(+ZeRO/FSDP)}\times\text{TP}\times\text{PP}$$
- **TP** 切层内矩阵(NVLink 内,见 [分布式训练与并行](./分布式训练与并行.md))
- **PP** 切层(跨节点,Bubble)
- **DP+ZeRO** 横向扩 + 去冗余
- 再配 **梯度检查点**(见 [优化器与训练稳定性](./优化器与训练稳定性.md))压激活。

## 5. 训练 vs 推理并行(对照)
| | 训练 | 推理 |
|---|---|---|
| 主压力 | 权重+梯度+优化器+激活 | 权重 + KV |
| 主方案 | DP+ZeRO/FSDP + TP + PP | TP(+PP) |
| ZeRO | 核心 | 几乎无用(无优化器状态) |

## 6. 自检
> [!question]
> - [ ] 16 字节/参数的来源;7B→112GB
> - [ ] ZeRO-1/2/3 分别分片什么
> - [ ] ZeRO-3/FSDP 的 all-gather + reduce-scatter 流程
> - [ ] ZeRO-Offload 思想
> - [ ] 3D 并行如何组合;梯度检查点配合
> - [ ] 为何 ZeRO 对推理几乎无用

## 相关
[分布式训练与并行](./分布式训练与并行.md) · [优化器与训练稳定性](./优化器与训练稳定性.md) · [大模型训练与微调](./大模型训练与微调.md) · [大模型推理与部署](./大模型推理与部署.md)
