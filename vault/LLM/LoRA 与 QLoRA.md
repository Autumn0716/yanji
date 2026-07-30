---
title: LoRA 与 QLoRA
description: 全参微调要存全套梯度+优化器状态,显存吃不消。LoRA 冻结原权重,只训一个低秩增量 $BA$,参数量降几百倍;QLoRA 再把基座 4-bit 量化,单卡就能微调 65B。枢纽见 大模型训练与微调。
tags:
- LLM
- finetuning
- LoRA
- PEFT
- imported-from-obsidian
aliases:
- LoRA
- QLoRA
created: 2026-06-28
---

# LoRA 与 QLoRA(参数高效微调)

> [!abstract] 一句话
> 全参微调要存全套梯度+优化器状态,显存吃不消。**LoRA** 冻结原权重,只训一个低秩增量 $BA$,参数量降几百倍;**QLoRA** 再把基座 4-bit 量化,单卡就能微调 65B。枢纽见 [大模型训练与微调](./大模型训练与微调.md)。

---

## 1. LoRA 原理
假设微调引起的权重更新 $\Delta W$ **低秩**。冻结 $W_0$,用两个小矩阵表达增量:
$$W=W_0+\Delta W=W_0+\frac{\alpha}{r}BA,\quad B\in\mathbb{R}^{d\times r},\ A\in\mathbb{R}^{r\times k},\ r\ll\min(d,k)$$
- 前向:$y=W_0x+\frac{\alpha}{r}B(Ax)$。
- **只训 $A,B$**,参数量 $dk\to r(d{+}k)$(如 $d{=}k{=}4096,r{=}8$:16M→64K,**降 256×**)。
- **初始化**:$A\sim\mathcal N(0,\sigma^2)$、$B=0$ → 初始 $\Delta W=0$,不扰动预训练。
- **缩放** $\alpha/r$:解耦学习率与秩,调 $r$ 时无需重调 lr。

> [!tip] 推理零开销
> 部署时把 $W_0+\frac{\alpha}{r}BA$ **合并**回一个矩阵 → 推理与原模型完全同速、同显存(不像 adapter 层有额外延迟)。多 adapter 不合并则走 [Multi-LoRA 服务](./Multi-LoRA 服务.md)。

## 2. 放在哪些层
通常加在注意力的 $W_q,W_v$(有时全 $q,k,v,o$ + FFN)。覆盖越多效果越好但参数越多。

## 3. QLoRA
在 LoRA 基础上把**基座权重 4-bit 量化**,梯度只流经 LoRA:
- **NF4(NormalFloat4)**:为正态分布权重设计的 4-bit 分位量化,信息论最优。
- **双量化(Double Quant)**:连量化用的 scale 也量化,再省一点显存。
- **Paged Optimizer**:优化器状态用 NVIDIA 统一内存分页,显存峰值溢出时自动换 CPU,防 OOM。
- 效果:**单张 48GB 卡微调 65B**,精度接近全精度 LoRA。
> 基座量化细节见 [模型量化](./模型量化.md)。

## 4. 显存账(为什么省)
全参微调每参数需:权重 + 梯度 + Adam 一/二阶动量 ≈ 16 字节(混合精度,见 [训练并行 FSDP ZeRO](./训练并行 FSDP ZeRO.md))。LoRA 只有 $BA$ 这点参数走这套,基座**只读不训**(QLoRA 还压成 4-bit)→ 优化器状态显存骤降。

## 5. 局限
- 低秩假设并非总成立,某些任务全参微调仍更强。
- 多个 LoRA 难直接叠加(干扰);服务多 adapter 见 [Multi-LoRA 服务](./Multi-LoRA 服务.md)。

## 6. 自检
> [!question]
> - [ ] LoRA $W_0+\frac{\alpha}{r}BA$ + 参数量降幅
> - [ ] 为何 $B=0$ 初始化
> - [ ] $\alpha/r$ 缩放作用
> - [ ] 推理时合并 → 零开销
> - [ ] QLoRA:NF4 + 双量化 + paged optimizer
> - [ ] LoRA 为何省优化器状态显存

## 相关
[大模型训练与微调](./大模型训练与微调.md) · [模型量化](./模型量化.md) · [Multi-LoRA 服务](./Multi-LoRA 服务.md) · [训练并行 FSDP ZeRO](./训练并行 FSDP ZeRO.md)
