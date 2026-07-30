---
title: Speculative Decoding 证明
description: 设目标大模型分布 $p=p\phi(\cdot\mid x{<i})$,草稿小模型 $q=q\theta(\cdot\mid x{<i})$,草稿一次提 $k$ 个候选 $\tilde x1..\tilde xk$,目标模型并行验证。
tags:
- LLM
- 题型精讲
- speculative-decoding
- imported-from-obsidian
aliases:
- 推测解码证明
created: 2026-06-28
---

# 题型精讲:Speculative Decoding 无损性证明(对应解答题 Q23)

> [!info] 考什么
> ①接受/拒绝规则 + 拒绝后的修正分布;②证明最终输出**严格服从目标分布** $p$;③加速比与 $k$ 的选择。概念见 [大模型推理与部署](../大模型推理与部署.md) §VI.1。

设目标大模型分布 $p=p_\phi(\cdot\mid x_{<i})$,草稿小模型 $q=q_\theta(\cdot\mid x_{<i})$,草稿一次提 $k$ 个候选 $\tilde x_1..\tilde x_k$,目标模型**并行**验证。

---

## (1) 接受/拒绝规则
对候选 $\tilde x_i$,以概率
$$r_i=\min\!\Big(1,\ \frac{p(\tilde x_i)}{q(\tilde x_i)}\Big)$$
**接受**;若**拒绝**,从修正分布重采样一个 token 并**停止本轮**后续候选:
$$p_\text{resample}(x)=\frac{\big(p(x)-q(x)\big)^+}{\sum_{x'}\big(p(x')-q(x')\big)^+},\qquad (\cdot)^+=\max(0,\cdot)$$

---

## (2) 证明:输出严格服从 $p$
只需对单个位置证 $P(\text{output}=x)=p(x)$。输出 $x$ 有两条互斥路径:

**路径 A(草稿提了 $x$ 且被接受)**:
$$P_A(x)=q(x)\cdot\min\!\Big(1,\frac{p(x)}{q(x)}\Big)=\min\big(q(x),p(x)\big)$$

**路径 B(发生拒绝,然后重采样恰好得到 $x$)**:先算总拒绝质量
$$Z=\sum_{x'}q(x')\Big(1-\min\big(1,\tfrac{p(x')}{q(x')}\big)\Big)=\sum_{x'}\big(q(x')-\min(q(x'),p(x'))\big)=\sum_{x'}\big(q(x')-p(x')\big)^+$$
由 $\sum q=\sum p=1$ 知 $\sum(q-p)^+=\sum(p-q)^+$,故 $Z=\sum_{x'}(p(x')-q(x'))^+$。于是
$$P_B(x)=Z\cdot p_\text{resample}(x)=Z\cdot\frac{(p(x)-q(x))^+}{Z}=\big(p(x)-q(x)\big)^+$$

**合并**:
$$P(\text{output}=x)=\min(q,p)+(p-q)^+$$
分情况:若 $p\ge q$:$\min=q,\ (p-q)^+=p-q\Rightarrow q+(p-q)=p$;若 $p<q$:$\min=p,\ (p-q)^+=0\Rightarrow p$。
$$\boxed{P(\text{output}=x)=p(x)\quad\forall x}$$
⇒ **输出精确等于目标分布**,与草稿模型 $q$ 无关(只影响速度,不影响正确性)。对 **Greedy 与 Top-p**(截断后的 $p$)都成立。

> [!tip] 直觉
> 接受规则"接受了 $q$ 中不超过 $p$ 的那部分质量 $\min(p,q)$",拒绝重采样"补上 $p$ 比 $q$ 多出来的部分 $(p-q)^+$"。两块拼起来正好是 $p$。

---

## (3) 加速比与 $k$ 的选择
设草稿/验证每步延迟 $t_d,t_v$,接受率 $\alpha$(各 token 独立近似)。一轮提 $k$ 个:
- **期望接受 token 数**:$\mathbb{E}=\sum_{i=1}^{k}\alpha^i=\dfrac{\alpha(1-\alpha^k)}{1-\alpha}$(第 $i$ 个被接受需前 $i$ 个都接受)。
- 一轮总延迟 $\approx t_d\cdot k$(草稿)$+\,t_v$(一次并行验证)。
$$S(k)=\frac{\mathbb{E}[\text{产出 token}]\cdot t_v}{t_d+t_v}=\frac{\alpha(1-\alpha^k)}{1-\alpha}\cdot\frac{t_v}{t_d+t_v}$$
**$k$ 选择**:$k$↑ 期望产出增,但草稿延迟线性涨;最优 $k^*$ 满足 $\partial S/\partial k=0$(边际产出 = 边际成本)。工程上按在线实测 $\alpha$ 动态调 $k\in\{2,3,4,8\}$:$\alpha$ 高 → $k$ 大更值;$\alpha$ 低 → 小 $k$ 防空跑。

> 数值例:$\alpha{=}0.8,k{=}4$ ⇒ 期望接受 $0.8{+}0.64{+}0.512{+}0.41\approx2.36$ token/轮。

---

## 易错点
- 修正分布是 $(p-q)^+$ 归一化,**不是** $p-q$(要截负为零)。
- $Z=\sum(p-q)^+=\sum(q-p)^+$ 这一步靠 $\sum p=\sum q=1$,是证明枢纽。
- "无损"指**分布无损**,不是"输出和纯目标模型逐 token 相同"(随机采样下序列可不同,但同分布)。

## 变式
- 免草稿:Medusa(多头)、EAGLE(特征层)、Lookahead(n-gram);见 [大模型推理与部署](../大模型推理与部署.md) §VI.1。
- 树注意力:一轮验证多条候选分支,提升每轮接受数。

## 相关
[大模型推理与部署](../大模型推理与部署.md) · [注意力机制](../注意力机制.md)
