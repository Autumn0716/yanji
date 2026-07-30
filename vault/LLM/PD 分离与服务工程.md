---
title: PD 分离与服务工程
description: Prefill 和 Decode 对资源诉求相反(算力 vs 带宽),挤在同一实例会互相拖累。P/D 分离把两者拆成独立集群、各自优化、用 KV 传输衔接,是大规模 LLM 服务的前沿架构。主线见 大模型推理与部署 §IV.4、§VIII。
tags:
- LLM
- serving
- disaggregation
- inference
- imported-from-obsidian
aliases:
- Prefill Decode 分离
- Disaggregation
created: 2026-06-28
---

# P/D 分离与服务工程

> [!abstract] 一句话
> Prefill 和 Decode 对资源诉求相反(算力 vs 带宽),挤在同一实例会互相拖累。**P/D 分离**把两者拆成独立集群、各自优化、用 KV 传输衔接,是大规模 LLM 服务的前沿架构。主线见 [大模型推理与部署](./大模型推理与部署.md) §IV.4、§VIII。

---

## 1. 动机:两个阶段水火不容
| | Prefill | Decode |
|---|---|---|
| 决定指标 | **TTFT**(首 token) | **TPOT**(token 间隔) |
| 算子 | GEMM(Compute-bound) | GEMV(Memory-bound) |
| 想要的卡 | 高算力 | 高带宽/大显存 |
| 互相干扰 | 长 Prefill 阻塞 Decode → P99 毛刺 | — |

同实例混跑(即便有 [Chunked Prefill](./大模型推理与部署.md) 缓解)仍是折中。

## 2. P/D 分离架构
```
请求 → [Prefill 集群] 算完整 prompt 的 KV
              │ KV 经 NVLink / RDMA 传输
              ▼
        [Decode 集群] 接 KV 继续逐 token 生成 → 流式返回
```
- 两集群**独立扩缩容**:在线对话 Decode 重、批处理 Prefill 重,按负载分别配比。
- 各自独立 batch 策略,**毛刺互不传染**。
- 代价:KV 跨实例**传输带宽与延迟**(一次 prompt 的全量 KV 不小)。

## 3. 代表系统
- **DistServe**:理论刻画 P/D 分离的 goodput 最优配比。
- **Mooncake**(月之暗面):以 **KV Cache 为中心**的池化架构,KV pool 跨节点共享 + 分级存储(见 [大模型推理与部署](./大模型推理与部署.md) §VIII 三级存储)。

## 4. 服务指标(SLA 核心)
| 指标 | 含义 |
|---|---|
| **TTFT** | 首 token 延迟,Prefill 决定 |
| **TPOT / ITL** | token 间隔,Decode 决定,影响"流畅度" |
| **Throughput** | 总 token/s |
| **Goodput** | **满足 SLO 前提下**的有效吞吐(最贴生产) |

> 优化目标常是"在 TTFT≤X、TPOT≤Y 约束下最大化 goodput",而非裸吞吐。

## 5. 服务工程要点
- **Autoscaling**:按队列长度/SLO 余量弹性扩缩 replica。
- **负载均衡**:请求路由到 KV 命中率高的实例(亲和性,配合 [Prefix Caching](./大模型推理与部署.md))。
- **请求优先级 / 抢占**:在线请求优先,离线批处理可被抢占(swap/recompute,见 [vLLM 源码](./vLLM 源码.md))。
- **延迟-吞吐 Pareto**:batch↑ 吞吐升但单请求延迟升;按 SLO 选工作点。
- **成本**:$/token ∝ 1/(吞吐×利用率)→ "提 SM 占用 = 降成本"。

## 6. 自检
> [!question]
> - [ ] Prefill/Decode 资源诉求为何相反
> - [ ] P/D 分离架构 + KV 传输代价
> - [ ] Mooncake 的 KV-centric 思想
> - [ ] TTFT/TPOT/throughput/goodput 区别
> - [ ] 为何优化 goodput 而非裸吞吐
> - [ ] 延迟-吞吐 Pareto 与成本关系

## 相关
[大模型推理与部署](./大模型推理与部署.md) · [vLLM 源码](./vLLM 源码.md) · [分布式训练与并行](./分布式训练与并行.md) · [Multi-LoRA 服务](./Multi-LoRA 服务.md)
