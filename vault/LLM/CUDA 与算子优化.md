---
title: CUDA 与算子优化
description: GPU 推理性能 = 让 Tensor Core 不饿、让 HBM 不堵、让 kernel 不空转。理解 SM/warp/显存层级,才能解释 FlashAttention、算子融合、CUDA Graph 为何有效。主线见 大模型推理与部署。
tags:
- GPU
- CUDA
- kernel
- inference
- imported-from-obsidian
aliases:
- CUDA
- Kernel Optimization
created: 2026-06-28
---

# CUDA 与算子优化

> [!abstract] 一句话
> GPU 推理性能 = **让 Tensor Core 不饿、让 HBM 不堵、让 kernel 不空转**。理解 SM/warp/显存层级,才能解释 FlashAttention、算子融合、CUDA Graph 为何有效。主线见 [大模型推理与部署](./大模型推理与部署.md)。

---

## 1. GPU 硬件层级
```
GPU ── 多个 SM(流多处理器)
        ├─ Tensor Core(矩阵乘单元,FP16/INT8/FP8)
        ├─ CUDA Core(标量/向量)
        ├─ Register / Shared Memory(SRAM,~KB,极快)
        └─ 共享 L2 + HBM(显存,~GB,慢 ~100×)
```
- **执行模型**:线程 → warp(32 线程,SIMT 锁步)→ block(同 SM,共享 SRAM)→ grid。
- **延迟隐藏**:warp 等内存时切到别的 warp,靠**高占用率(occupancy)**掩盖访存延迟。

## 2. 显存层级与带宽墙
| 层级 | 容量 | 带宽/延迟 |
|---|---|---|
| Register | ~256KB/SM | 最快 |
| Shared Mem(SRAM) | ~228KB/SM(H100) | ~TB/s,低延迟 |
| L2 | ~50MB | 中 |
| **HBM** | 80GB | 2–4.8 TB/s,**高延迟** |

> FlashAttention 的全部意义:把 $n\times n$ 中间矩阵留在 **SRAM**,绝不写回 HBM(见 [注意力机制](./注意力机制.md) online softmax)。

## 3. Roofline 与算术强度
$$I=\frac{\text{FLOPs}}{\text{HBM 字节}},\quad I<I^*(\text{算力/带宽})\Rightarrow\text{Memory-bound}$$
- **GEMM**(矩阵×矩阵,Prefill):$I$ 高 → Compute-bound,吃满 Tensor Core。
- **GEMV**(矩阵×向量,Decode):$I\approx1$ → Memory-bound,Tensor Core 饿肚子。
- 优化方向 = 把 GEMV 凑成 GEMM(batch)、减少 HBM 往返。

## 4. 三大优化手段

### 4.1 算子融合(Kernel Fusion)
多个小 kernel 合一,省 kernel launch + 省中间结果的 HBM 往返:
- QKV 三投影 → 一个 GEMM
- RMSNorm + 残差 add 融合
- Rotary(RoPE)融进 attention kernel
- SwiGLU:$\text{SiLU}(xW_1)\odot(xW_3)$ 融合
- bias + 激活 + dropout 融合

### 4.2 CUDA Graph
Decode 每步是一长串**极小 kernel**,CPU 提交(launch)开销(μs 级 × 几百 kernel)能盖过 GPU 计算。CUDA Graph **录制一次、重放一次 launch**,消灭提交瓶颈。对小 batch Decode 提速显著。

### 4.3 高效 GEMM/注意力库
- **cuBLAS / CUTLASS**:GEMM 模板,tiling + 双缓冲 + Tensor Core。
- **FlashAttention**:见 [注意力机制](./注意力机制.md)、[大模型推理与部署](./大模型推理与部署.md) §VII.1。
- **Triton**:用 Python 写高性能 kernel(vLLM/许多融合 kernel 用它)。

## 5. 数值精度与 Tensor Core
- Tensor Core 吞吐:FP16 → INT8/FP8 翻倍(见 [模型量化](./模型量化.md))。
- 累加常在 FP32 防溢出;FP16 范围窄(±65504),BF16 范围同 FP32。

## 6. 通信(多卡)
TP 每层 AllReduce = ReduceScatter + AllGather;Ring 带宽最优 $2\frac{N-1}{N}$ 数据。Decode 的 TP 通信是**小消息→延迟敏感**,NVLink(600GB/s 低延迟)远优于 PCIe。详见 [大模型推理与部署](./大模型推理与部署.md) §VII.4。

## 7. 自检
> [!question]
> - [ ] SM/warp/block;occupancy 如何隐延迟
> - [ ] SRAM vs HBM 带宽差;FlashAttn 为何快
> - [ ] GEMM(Prefill)vs GEMV(Decode)的 Roofline 位置
> - [ ] 算子融合融了哪些;CUDA Graph 为何对 Decode 关键
> - [ ] Tensor Core 低比特吞吐;FP32 累加原因

## 相关
[大模型推理与部署](./大模型推理与部署.md) · [注意力机制](./注意力机制.md) · [模型量化](./模型量化.md) · [vLLM 源码](./vLLM 源码.md)
