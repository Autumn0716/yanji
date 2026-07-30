---
title: Mamba 状态空间模型
description: SSM 用"递归状态"替代注意力:推理时每 token 只更新一个定长状态,O(1) 显存、无 KV Cache 爆炸,O(n) 总算。Mamba 的关键是让状态转移随输入变化(选择性),补上了线性注意力的表达力短板。是 注意力机制 的强力替代。主线见 大模型推理与部署。
tags:
- LLM
- SSM
- mamba
- architecture
- imported-from-obsidian
aliases:
- Mamba
- SSM
- State Space Model
created: 2026-06-28
---

# Mamba 状态空间模型(SSM)

> [!abstract] 一句话
> SSM 用"递归状态"替代注意力:推理时**每 token 只更新一个定长状态,O(1) 显存、无 KV Cache 爆炸,O(n) 总算**。Mamba 的关键是让状态转移**随输入变化(选择性)**,补上了线性注意力的表达力短板。是 [注意力机制](./注意力机制.md) 的强力替代。主线见 [大模型推理与部署](./大模型推理与部署.md)。

---

## 1. 动机:干掉 $O(n^2)$ 与 KV 增长
注意力时间 $O(n^2d)$、KV Cache 随 $n$ 线性涨(见 [大模型推理与部署](./大模型推理与部署.md))。SSM 把序列建模成线性递归 → 推理常数显存、线性时间,长序列尤其香。

## 2. SSM 基础
连续状态空间(控制论经典):
$$h'(t)=Ah(t)+Bx(t),\quad y(t)=Ch(t)$$
离散化(步长 $\Delta$,零阶保持):$\bar A=e^{\Delta A},\ \bar B=(\Delta A)^{-1}(e^{\Delta A}-I)\Delta B$,得
$$h_t=\bar A h_{t-1}+\bar B x_t,\quad y_t=Ch_t$$
**双形式**:
- **递归形式**(推理):逐 token 更新状态,O(1)/token。
- **卷积形式**(训练):展开成全局卷积核 $\bar K$,可并行。

## 3. S4 → Mamba(S6)
- **S4**:用 **HiPPO** 理论初始化 $A$(让状态最优记忆历史),解决长程依赖,但 $A,B,C$ **不随输入变化**(线性时不变 LTI)→ 不能按内容选择性记忆/遗忘。
- **Mamba(S6)**:让 $\Delta,B,C$ **随输入 $x_t$ 变化**(selective / 时变)→ 模型能"按内容决定记什么、忘什么"。代价:时变后不能用卷积,改用 **硬件感知的并行扫描(parallel scan)** 在 GPU SRAM 上高效算(类似 FlashAttention 的 IO 思路,见 [CUDA 与算子优化](./CUDA 与算子优化.md))。

## 4. 推理优势(部署视角)⭐
| | Transformer | Mamba |
|---|---|---|
| 每 token 显存 | KV 随 $n$ 增长 | **定长状态 O(1)** |
| 总时间 | $O(n^2)$ | $O(n)$ |
| 长上下文 | KV 爆炸 | 天然友好 |
| 检索/in-context | ★★★(强) | 较弱(状态有限) |

> [!note] 没有 KV Cache
> Mamba 推理只维护一个固定大小状态 $h$,不随上下文增长 → 不需要 PagedAttention / KV 量化 / offload 那套(那些都是为 KV 服务的,见 [大模型推理与部署](./大模型推理与部署.md) §III)。这是架构级的"减少要搬的数据"。

## 5. 局限与混合架构
- **局限**:定长状态信息瓶颈,精确检索/复制长程具体 token 不如注意力(attention 能精确回看任意位置)。
- **混合(主流折中)**:**Jamba**、**Zamba** 等把少量注意力层插进 Mamba 堆叠 —— 大多数层用 Mamba(省 KV、线性),少数注意力层补检索能力。兼顾长上下文效率与精确回看。

## 6. 自检
> [!question]
> - [ ] SSM 连续式 + 离散化递归式
> - [ ] 递归形式(推理)vs 卷积形式(训练)
> - [ ] S4(LTI + HiPPO)→ Mamba(selective 时变)的关键改动
> - [ ] 为何时变后用并行扫描而非卷积
> - [ ] Mamba 推理 O(1) 状态、无 KV 的部署含义
> - [ ] 局限(检索弱)与混合架构(Jamba)

## 相关
[大模型推理与部署](./大模型推理与部署.md) · [注意力机制](./注意力机制.md) · [Transformer 架构](./Transformer 架构.md) · [CUDA 与算子优化](./CUDA 与算子优化.md)
