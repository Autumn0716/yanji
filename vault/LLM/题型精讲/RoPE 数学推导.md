---
title: RoPE 数学推导
description: '设 $\thetai=10000^{-2i/d},\ i=0..\tfrac d2-1$。对位置 $m$,在每对维度 $(2i,2i{+}1)$ 上施加旋转: $$R{m,i}=\begin{pmatrix}\cos m\thetai & -\sin m\thetai\\ \sin m\thetai & \cos m\'
tags:
- LLM
- 题型精讲
- RoPE
- position-encoding
- imported-from-obsidian
aliases:
- RoPE 推导
created: 2026-06-28
---

# 题型精讲:RoPE 相对位置 + 长度外推(对应解答题 Q20)

> [!info] 考什么
> ①证明 $\langle R_mq,R_nk\rangle$ 只依赖相对位置 $m-n$;②解释相对外推优势 + NTK-aware scaling。概念见 [注意力机制](../注意力机制.md)、[大模型推理与部署](../大模型推理与部署.md) §II.4。

设 $\theta_i=10000^{-2i/d},\ i=0..\tfrac d2-1$。对位置 $m$,在每对维度 $(2i,2i{+}1)$ 上施加旋转:
$$R_{m,i}=\begin{pmatrix}\cos m\theta_i & -\sin m\theta_i\\ \sin m\theta_i & \cos m\theta_i\end{pmatrix}$$

---

## (1) 内积只依赖相对位置 $m-n$
记 $q_i=(q_{2i},q_{2i+1})^\top,\ k_i$ 同理。整向量内积按维度对拆分:
$$\langle R_mq,R_nk\rangle=\sum_{i=0}^{d/2-1} (R_{m,i}q_i)^\top(R_{n,i}k_i)=\sum_i q_i^\top R_{m,i}^\top R_{n,i}\,k_i$$
**关键:旋转矩阵的群性质**。$R_{m,i}^\top=R_{-m,i}$(旋转 $-m\theta_i$),且旋转可加:
$$R_{m,i}^\top R_{n,i}=R_{-m,i}R_{n,i}=R_{(n-m),i}=\begin{pmatrix}\cos(n{-}m)\theta_i & -\sin(n{-}m)\theta_i\\ \sin(n{-}m)\theta_i & \cos(n{-}m)\theta_i\end{pmatrix}$$
因此
$$\boxed{\langle R_mq,R_nk\rangle=\sum_i q_i^\top R_{(n-m),i}\,k_i}$$
**只依赖 $m-n$**($n{-}m$ 与 $m{-}n$ 仅差符号,cos 偶/sin 奇)。⇒ RoPE 把绝对位置编码转化为相对位置编码,无需在 attention 里显式加偏置项。

> [!tip] 复数视角(更快)
> 令 $z_i=q_{2i}+\mathrm{i}\,q_{2i+1}$,RoPE 即乘 $e^{\mathrm{i}m\theta_i}$。则
> $$\langle R_mq,R_nk\rangle=\text{Re}\sum_i z_i^{(q)}\overline{z_i^{(k)}}\,e^{\mathrm{i}(m-n)\theta_i}$$
> 一眼看出只含相对相位 $(m-n)\theta_i$。

---

## (2) 长度外推优势 + NTK-aware scaling
**绝对编码的问题**:训练到 $L_\text{train}$ 后,$>L_\text{train}$ 的位置编码从未见过,外推性能急剧崩。
**RoPE 的相对性**让模型只需见过"相对距离"分布即可,但 $m\theta_i$ 在 $m>L_\text{train}$ 时仍超出训练相位周期,直接外推也有损。
**NTK-aware**:修改频率基底,$s=L_\text{new}/L_\text{train}$:
$$\theta_i'=\big(\text{base}\cdot s^{\,d/(d-2)}\big)^{-2i/d}\quad\Rightarrow\quad \text{高频(小 }i\text{)几乎不缩、低频(大 }i\text{)按 }s\text{ 缩}$$
**直觉**:高频分量负责区分相邻 token(精细分辨率),不能动;低频分量周期长、负责远程,按 $s$ 拉长正好覆盖新长度。等价于"在频率空间做插值",比线性位置插值(PI,全频统一压 $\theta_i/s$)保留更多高频信息。**YaRN** 再加分频段 + attention 温度修正。

---

## 易错点
- $R_m^\top R_n=R_{n-m}$ 不是 $R_{m-n}$——写清方向(虽内积对称下等价)。
- RoPE **只作用 $Q,K$,不作用 $V$**;不改变 $O(n^2d)$ 复杂度(常见错项称其升到 $O(n^3)$)。
- NTK 的精髓是"非均匀缩放",PI 是"均匀缩放"——别混。

## 变式
- 问"为何除 $\sqrt{d_h}$"→ 那是 attention 缩放,与 RoPE 无关(见 [注意力机制](../注意力机制.md))。
- 问 ALiBi 区别:ALiBi 直接给分数加 $-\lambda|i-j|$ 线性偏置,天然外推,不旋转。

## 相关
[注意力机制](../注意力机制.md) · [大模型推理与部署](../大模型推理与部署.md) · [Transformer 架构](../Transformer 架构.md)
