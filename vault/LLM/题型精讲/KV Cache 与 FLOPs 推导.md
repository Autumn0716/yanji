---
title: KV Cache 与 FLOPs 推导
description: 设 $L$ 层、隐藏维 $d$、头数 $h$、$dh=d/h$、fp16。矩阵乘 $A{m\times k}B{k\times n}$ 的 FLOPs $=2mkn$(每输出元素 $k$ 次乘加)。
tags:
- LLM
- 题型精讲
- KV-cache
- imported-from-obsidian
aliases:
- KV Cache 推导
created: 2026-06-28
---

# 题型精讲:KV Cache 必要性与 FLOPs 推导(对应解答题 Q19)

> [!info] 考什么
> 用 FLOPs 逐项算清"不用 KV Cache vs 用 KV Cache"的复杂度差,据此解释为何降 $n$ 倍、为何 Decode 是 Memory-bound。概念见 [大模型推理与部署](../大模型推理与部署.md) §III、[注意力机制](../注意力机制.md)。

设 $L$ 层、隐藏维 $d$、头数 $h$、$d_h=d/h$、fp16。矩阵乘 $A_{m\times k}B_{k\times n}$ 的 FLOPs $=2mkn$(每输出元素 $k$ 次乘加)。

---

## (1) 不使用 KV Cache —— 单步 $O(n^2d)$
生成第 $n{+}1$ 个 token,要重算前 $n{+}1$ 个位置的 $K,V$:
$$K=X_{1:n+1}W_K,\quad V=X_{1:n+1}W_V,\quad X\in\mathbb{R}^{(n+1)\times d}$$
- **KV 构造**:两次 $(n{+}1)\times d$ 乘 $d\times d$ → 各 $2(n{+}1)d^2$,合计 $4(n{+}1)d^2$。
- **注意力分数** $S=QK^\top\in\mathbb{R}^{(n+1)\times(n+1)}$:$2(n{+}1)^2d$。
- **加权** $O=\text{softmax}(S)V$:$2(n{+}1)^2d$。
- 合计 $\approx 4(n{+}1)^2d+4(n{+}1)d^2=O(n^2d+nd^2)$。

> 长序列 $n\gg d$ ⇒ 主导项 **$O(n^2d)$**。每生成一个 token 都把历史重算一遍,极其浪费。

---

## (2) 使用 KV Cache —— 单步 $O(nd)$
$K,V$ 已缓存,只需算**当前位置**:
$$q_{n+1}=x_{n+1}W_Q\ (2d^2),\quad s=q_{n+1}K^\top\in\mathbb{R}^{1\times n}\ (2nd),\quad o=\text{softmax}(s)V\ (2nd)$$
- 合计 $\approx 4nd+2d^2=O(nd)$。

> [!check] 核心结论
> 相比 (1) 的 $O(n^2d)$,**降了 $n$ 倍**。本质:把"对全部历史 token 重复算 $K,V$"换成"只算当前 $q$",**二次项 → 一次项**。代价是用显存存 $K,V$(空间换时间)。

---

## (3) 显存公式 + 为何 Decode 是 Memory-bound
$$\boxed{M_{KV}=2\,(K{+}V)\times L\times n\times h_{KV}\times d_h\times b_{kv}\times B}$$
**算术强度**:每步 FLOPs $O(nd)$,需读 KV 字节 $O(L\,n\,h_{KV}\,d_h)$,
$$I=\frac{\text{FLOPs}}{\text{字节}}\sim\frac{2nd}{2\cdot L n h_{KV}d_h\cdot 2}\sim\frac{1}{d_h}\approx\frac{1}{128}\ \ll\ I^*_\text{A100}\approx156$$
⇒ **Decode 被显存带宽卡死(Memory-bound)**,这是所有 KV 优化(GQA/量化/分页/offload)的根因。

---

## 易错点
- 矩阵乘 FLOPs 是 $2mkn$,别忘那个因子 2(乘+加)。
- KV Cache 不降**总**复杂度的量级直觉,而是降**单步**:整段生成 $n$ 个 token,无缓存 $O(n^3d)$ → 有缓存 $O(n^2d)$。
- 显存公式里是 $h_{KV}$(GQA/MQA 用 KV 头数)不是 $h$。

## 变式
- 把 fp16 换 int8:显存与带宽都减半,$I$ 翻倍 → 见 [模型量化](../模型量化.md) KV 量化。
- 改 GQA:$h_{KV}{=}h/8$,KV 与读取量降 8×。

## 相关
[大模型推理与部署](../大模型推理与部署.md) · [注意力机制](../注意力机制.md) · [模型量化](../模型量化.md)
