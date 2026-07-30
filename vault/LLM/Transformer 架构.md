---
title: Transformer 架构
description: 现代 LLM = 堆叠 $L$ 个 (注意力 + FFN) 块的 decoder-only 模型,残差 + 归一化把它训得深。推理时每块的注意力产生 KV Cache、FFN 占权重大头。部署见 大模型推理与部署,注意力细节见 注意力机制。
tags:
- LLM
- transformer
- architecture
- imported-from-obsidian
aliases:
- Transformer
- Decoder-only
created: 2026-06-28
---

# Transformer 架构(Decoder-only LLM)

> [!abstract] 一句话
> 现代 LLM = 堆叠 $L$ 个 **(注意力 + FFN)** 块的 decoder-only 模型,残差 + 归一化把它训得深。推理时每块的注意力产生 KV Cache、FFN 占权重大头。部署见 [大模型推理与部署](./大模型推理与部署.md),注意力细节见 [注意力机制](./注意力机制.md)。

---

## 1. 数据流(一个 token 的旅程)
```
token id → Embedding(查表 d 维)
   ↓  (+ 位置信息,RoPE 在注意力内施加)
┌─ × L 层 ────────────────────────┐
│  x → RMSNorm → Attention → +残差 │
│  x → RMSNorm → FFN/MoE   → +残差 │
└──────────────────────────────────┘
   ↓
最终 RMSNorm → LM Head(投影到词表 V)→ logits → 采样
```

## 2. 关键组件

### 2.1 Embedding & LM Head
- Embedding:$V\times d$ 查表(权重大,$V\sim10^5$)。
- LM Head:$d\times V$ 投影出 logits;常与 Embedding **权重绑定(weight tying)**。
- 大词表导致 logits 层很大 → TP 时按词表维切(vocab parallel)。

### 2.2 归一化:Pre-Norm + RMSNorm
- **Pre-Norm**(norm 在子层前)训练更稳,几乎所有 LLM 采用;Post-Norm 深了易爆。
- **RMSNorm**:$\hat x=\dfrac{x}{\sqrt{\frac1d\sum x_i^2+\epsilon}}\cdot g$,比 LayerNorm 省去均值中心化,更快、效果相当。
- 在 [TP](./大模型推理与部署.md) 中 RMSNorm **被复制到每个 rank,不切分**(经典考点)。

### 2.3 FFN / SwiGLU
- 经典 FFN:$\text{FFN}(x)=W_2\,\text{act}(W_1x)$,中间维 $\sim4d$。
- **SwiGLU**(LLaMA 系):$\text{FFN}(x)=W_2\big(\text{SiLU}(W_1x)\odot(W_3x)\big)$,门控提升质量,中间维取 $\frac83d$ 保参数量。
- FFN 占每层约 2/3 参数 → **短上下文推理的权重读取主要在 FFN**。

### 2.4 残差连接
$x_{l+1}=x_l+\text{Sublayer}(\text{Norm}(x_l))$,残差流是信息高速公路,使梯度直达、深层可训。

## 3. 因果自回归
decoder-only 用 causal mask,训练时一次并行算所有位置(teacher forcing),推理时逐 token(每步依赖 KV Cache)。这正是 Prefill(并行)vs Decode(串行)分野的根源。

## 4. 参数量估算
单层 ≈ 注意力 $4d^2$(QKVO)+ FFN $\sim8d^2$(SwiGLU 约 $3\times\frac83d^2$)$\approx12d^2$。总 $\approx 12Ld^2 + Vd$(嵌入)。
> 用此可反推:给定参数量 $N$,推理每 token FLOPs $\approx2N$(见 [大模型推理与部署](./大模型推理与部署.md) §I.1)。

## 5. 与其他架构对比
| | Decoder-only | Encoder-Decoder | SSM/Mamba |
|---|---|---|---|
| 代表 | GPT/LLaMA | T5/BART | Mamba |
| 推理 | KV Cache 增长 | cross-attn | 常数状态,见 [Mamba 状态空间模型](./Mamba 状态空间模型.md) |
| 主流度 | ★★★ | 翻译/特定任务 | 新兴 |

## 6. 自检
> [!question]
> - [ ] 一个 token 的完整数据流
> - [ ] Pre-Norm vs Post-Norm;RMSNorm vs LayerNorm
> - [ ] SwiGLU 公式 + 中间维为何 8/3 d
> - [ ] 残差作用;weight tying
> - [ ] 单层 ~12d² 参数 → 推 2N FLOPs
> - [ ] 为何 decoder-only Prefill 并行 Decode 串行

## 相关
[大模型推理与部署](./大模型推理与部署.md) · [注意力机制](./注意力机制.md) · [MoE 混合专家](./MoE 混合专家.md) · [Mamba 状态空间模型](./Mamba 状态空间模型.md)
