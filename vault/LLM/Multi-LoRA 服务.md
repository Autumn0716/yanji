---
title: Multi-LoRA 服务
description: 一个基座模型 + 大量 LoRA adapter(每客户/任务一个),想在一张卡上同时服务而非每个 adapter 单独部署。关键是把 adapter 权重像 KV 一样分页管理、批内不同 adapter 也能高效批算。LoRA 本身见 LoRA 与 QLoRA,主线见 大模型推理与部署 §VIII.2。
tags:
- LLM
- LoRA
- serving
- inference
- imported-from-obsidian
aliases:
- Multi-LoRA
- S-LoRA
created: 2026-06-28
---

# Multi-LoRA 服务(一基座多适配器)

> [!abstract] 一句话
> 一个基座模型 + 大量 LoRA adapter(每客户/任务一个),想在**一张卡上同时服务**而非每个 adapter 单独部署。关键是把 adapter 权重像 KV 一样分页管理、批内不同 adapter 也能高效批算。LoRA 本身见 [LoRA 与 QLoRA](./LoRA 与 QLoRA.md),主线见 [大模型推理与部署](./大模型推理与部署.md) §VIII.2。

---

## 1. 动机
LoRA 微调产出大量小 adapter(每个 $BA$ 仅几十 MB)。若每 adapter 起一个完整推理实例 → 基座权重重复占显存、利用率极低。Multi-LoRA:**基座只存一份,adapter 按需挂载**,一卡服千 adapter。

## 2. 核心挑战
推理时 $y=Wx+\frac{\alpha}{r}B(Ax)$,基座 $W$ 共享,但**同一 batch 内不同请求用不同的 $(A,B)$** → 不能简单合并成一个 GEMM。

## 3. S-LoRA
- **Unified Paging**:把所有 adapter 权重和 KV Cache 一起,用统一的分页内存池管理(借鉴 [PagedAttention](./vLLM 源码.md)),按需换入换出 adapter,显存碎片小。
- **异构批处理**:基座部分正常批算,LoRA 部分按 adapter 分组算。
- 可在单卡服务**数千** adapter,吞吐接近纯基座。

## 4. Punica
- **SGMV(Segmented Gather Matrix-Vector)**:自定义 CUDA kernel,一次 kernel 内处理"batch 内多个不同 adapter"的小矩阵乘,避免逐请求串行。
- 让"多 adapter 混合 batch"的额外开销趋近于零。

## 5. 与量化结合
基座可量化(GPTQ/AWQ,见 [模型量化](./模型量化.md))进一步省显存,adapter 保持高精度小权重 → 显存利用率再上一层。

## 6. 自检
> [!question]
> - [ ] 为何不能把多 adapter 简单合并成一个 GEMM
> - [ ] S-LoRA Unified Paging 思想(类比 PagedAttention)
> - [ ] Punica SGMV 解决什么
> - [ ] 基座量化 + adapter 高精度的组合收益

## 相关
[大模型推理与部署](./大模型推理与部署.md) · [LoRA 与 QLoRA](./LoRA 与 QLoRA.md) · [vLLM 源码](./vLLM 源码.md) · [模型量化](./模型量化.md) · [PD 分离与服务工程](./PD 分离与服务工程.md)
