---
title: vLLM 源码
description: vLLM = PagedAttention(去显存碎片)+ Continuous Batching(iteration 级调度) 的高吞吐推理引擎。读懂它的调度器与 block manager,就懂了现代推理服务的骨架。原理见 大模型推理与部署。
tags:
- LLM
- vLLM
- serving
- inference
- imported-from-obsidian
aliases:
- vLLM
created: 2026-06-28
---

# vLLM 源码与架构

> [!abstract] 一句话
> vLLM = **PagedAttention(去显存碎片)+ Continuous Batching(iteration 级调度)** 的高吞吐推理引擎。读懂它的调度器与 block manager,就懂了现代推理服务的骨架。原理见 [大模型推理与部署](./大模型推理与部署.md)。

---

## 1. 顶层架构
```
LLMEngine(同步) / AsyncLLMEngine(在线服务)
  ├─ Scheduler          决定每 iteration 跑哪些请求
  │    └─ BlockManager   KV 物理块分配/回收/换入换出
  ├─ Worker × N          每张 GPU 一个(TP/PP)
  │    └─ ModelRunner    准备输入、跑前向、采样
  └─ PagedAttention kernel
```

## 2. PagedAttention(显存)
KV Cache 切固定 **block(默认 16 token)**,**Block Table** 做逻辑 token→物理块映射:
- 按需分配,几乎零碎片,利用率 >90%。
- **Copy-on-Write**:beam/并行采样共享前缀块,分叉时才复制。
- attention kernel 改为"按 block 索引 gather KV"。
> 治内/外碎片 + 预分配浪费;**不**管算子融合(见 [大模型推理与部署](./大模型推理与部署.md) §III.3)。

## 3. Scheduler(调度)
每 iteration 维护 **waiting / running / swapped** 三态:
- 显存够 → 从 waiting 提请求做 Prefill;否则只跑 running 的 Decode。
- **抢占**:KV 不够踢低优先级请求,二选一 **Swap**(KV→CPU,费 PCIe)/ **Recompute**(丢 KV 重跑 Prefill,费算力)。
- **Chunked Prefill / Prefix Caching**:长 Prefill 切块与 Decode 混跑、复用公共前缀(见 [大模型推理与部署](./大模型推理与部署.md) §IV)。

## 4. Continuous Batching
以 **iteration(token)为单位**调度:请求生成 EOS 立即让槽给新请求,GPU 不空等。高吞吐关键。

## 5. 采样与输出
ModelRunner 得 logits → **SamplingParams**(temperature/top-k/top-p/min-p,见 [大模型推理与部署](./大模型推理与部署.md) §VI.2)→ 采样 → 流式返回;支持 **guided decoding**(FSM/语法掩码)。

## 6. 分布式
TP(层内权重切,每层 AllReduce/NCCL)+ PP(跨层流水),详见 [分布式训练与并行](./分布式训练与并行.md)。

## 7. 能力
高并发吞吐;支持 GPTQ/AWQ/FP8 + KV cache 量化(见 [模型量化](./模型量化.md))、Speculative Decoding、多 LoRA。

## 8. 自检
> [!question]
> - [ ] Engine/Scheduler/BlockManager/Worker 职责
> - [ ] PagedAttention block table + CoW
> - [ ] 三态调度 + 抢占 swap vs recompute
> - [ ] Continuous Batching 为何高吞吐

## 相关
[大模型推理与部署](./大模型推理与部署.md) · [CUDA 与算子优化](./CUDA 与算子优化.md) · [模型量化](./模型量化.md) · [分布式训练与并行](./分布式训练与并行.md)
