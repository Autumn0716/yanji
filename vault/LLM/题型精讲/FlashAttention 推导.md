---
title: FlashAttention 推导
description: $Q,K,V\in\mathbb{R}^{n\times d}$,$S,P\in\mathbb{R}^{n\times n}$,SRAM 大小 $M$。
tags:
- LLM
- 题型精讲
- flashattention
- imported-from-obsidian
aliases:
- FlashAttention 推导
- Online Softmax
created: 2026-06-28
---

# 题型精讲:FlashAttention 的 IO 与 Online Softmax(对应解答题 Q21)

> [!info] 考什么
> ①标准注意力的 HBM 读写量;②Tiling + running softmax 增量计算 + 更新公式;③Recomputation 为何"以算代存",IO 仍 $O(N^2d^2/M)$。概念见 [注意力机制](../注意力机制.md)、[CUDA 与算子优化](../CUDA 与算子优化.md)。

$Q,K,V\in\mathbb{R}^{n\times d}$,$S,P\in\mathbb{R}^{n\times n}$,SRAM 大小 $M$。

---

## (1) 标准注意力的 HBM 访问
| 步骤 | 读 | 写 |
|---|---|---|
| $S=QK^\top$ | $Q,K$:$2nd$ | $S$:$n^2$ |
| $P=\text{softmax}(S)$ | $S$:$n^2$ | $P$:$n^2$ |
| $O=PV$ | $P,V$:$n^2{+}nd$ | $O$:$nd$ |

总 HBM 访问 $\approx O(n^2+nd)$,$n\gg d$ 时 $n^2$ 主导,**远高于算术下界 $O(nd)$**。瓶颈不在算,在反复把 $n\times n$ 的 $S,P$ 写回再读出 HBM。

---

## (2) Tiling + Online Softmax
取 block 大小 $B_r,B_c=\Theta(M/d)$,使一块 $Q_i,K_j,V_j$ 能放进 SRAM。$Q$ 按行分 $T_r$ 块、$K,V$ 按列分 $T_c$ 块。外层遍历 $Q_i$,内层遍历 $K_j,V_j$,在 **SRAM 内**算局部 $S_{ij}=Q_iK_j^\top$。

softmax 分母需要**全局** max 与 sum,但我们只能逐块看到部分 → 维护 running statistic $(m,\ell)$ 增量更新:
$$m^\text{new}=\max\!\big(m^\text{old},\ \text{rowmax}(S_{ij})\big)$$
$$\ell^\text{new}=e^{\,m^\text{old}-m^\text{new}}\ell^\text{old}+\text{rowsum}\!\big(e^{\,S_{ij}-m^\text{new}}\big)$$
$$\tilde O^\text{new}=e^{\,m^\text{old}-m^\text{new}}\tilde O^\text{old}+e^{\,S_{ij}-m^\text{new}}V_j,\qquad O_i=\tilde O_i/\ell_i$$

> [!check] 为什么数值等价于全局 softmax
> 合并两段、各自局部最大 $m_1,m_2$、未归一化和 $\ell_1,\ell_2$、未归一化加权 $\tilde O_1,\tilde O_2$。取 $m=\max(m_1,m_2)$,则
> $$\ell=e^{m_1-m}\ell_1+e^{m_2-m}\ell_2,\quad \tilde O=e^{m_1-m}\tilde O_1+e^{m_2-m}\tilde O_2$$
> rescale 因子 $e^{m_\text{old}-m_\text{new}}$ 把"按旧 max 归一"修正成"按新 max 归一"。最终 $O=\tilde O/\ell$ 与一次性全局 $\text{softmax}$ **逐位相等**;减 max 同时保证 $e^{\cdot}\le1$ 不溢出。
> 关键:$S,P$ **从不写回 HBM**,只在 SRAM 流过,最后只写 $O$。

**IO 分析**:$Q$ 每块进一次,$K,V$ 被每个 $Q$ 块各扫一遍 → 共 $\approx O(\tfrac{n^2 d}{M}\cdot d)=O(N^2d^2/M)$。$M$ 越大,IO 越低。

---

## (3) Recomputation(反向以算代存)
反向传播需要 $S,P$,但**不存**它们(存了又回到 $n^2$ HBM)。改为反向时**重算** $S_{ij}=Q_iK_j^\top$:
- 代价:额外 $O(n^2d)$ FLOPs(相对前向多一倍计算)。
- 收益:HBM 访问仍 $O(N^2d^2/M)$,避免存 $n\times n$ 中间矩阵。

> [!note] 为何稳赚
> 现代 GPU **算力远富于带宽**(A100:312 TFLOPS vs 2TB/s,见 [CUDA 与算子优化](../CUDA 与算子优化.md))。多算 FLOPs 换少搬 HBM,在 Memory-bound 区净赚。这就是"以算代存"。

---

## 易错点
- 别把"时间复杂度"和"IO 复杂度"混了:FlashAttention **时间仍 $O(n^2d)$**(没减计算量),减的是 **HBM 访问**。
- 更新式里 $\tilde O$ 是**未归一化**的加权 $V$ 和,最后才除 $\ell$。
- 减 max 的目的双重:数值防溢出 + 让分块可合并。

## 变式
- **FlashDecoding**:Decode 时 $q$ 只有 1 行,改沿 KV 序列维切分并行,救长上下文 Decode。
- 问"为何不直接缓存 softmax 分母"→ 因为分母依赖全局 max,流式时 max 会变,必须 rescale。

## 相关
[注意力机制](../注意力机制.md) · [CUDA 与算子优化](../CUDA 与算子优化.md) · [大模型推理与部署](../大模型推理与部署.md)
