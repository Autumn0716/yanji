---
title: RISC 与 CISC
description: 精简指令集与复杂指令集对比：指令密度、访存约束，以及「乘法可访存 ⇒ CISC」等判断题要点。
tags: [408, 计组, 指令结构, imported-from-obsidian]
---

# RISC 与 CISC

> [!abstract] 一句话
> **CISC**（如 x86）指令多而复杂，可直接访存；**RISC**（如 ARM）指令精简，访存通常只走 load/store。

## 原文要点

### CISC — Complex Instruction Set Computer
- 代表：x86
- 经验法则：约 80% 的语句只会用到 20% 的指令（指令使用极不均衡）
- 允许复杂寻址；**乘法等指令可以访存** → 若某架构乘法可直接访存，则更像 CISC

### RISC — Reduced Instruction Set Computer
- 代表：ARM（手机、Apple Silicon 笔记本等）
- 访存通常**只能**通过 load / store；运算在寄存器间完成

![课堂截图：RISC/CISC 对比](附件/Screenshot%202026-07-05%20at%2019-53-14.png)

## 补充对照

| | CISC | RISC |
| --- | --- | --- |
| 指令数量/复杂度 | 多、可变长、语义丰富 | 少、定长、语义简单 |
| 访存 | 多数运算指令可带内存操作数 | 典型 load/store 架构 |
| 译码/流水 | 译码复杂，优化靠微码/融合 | 译码简单，易深流水/超标量 |
| 代码密度 | 往往更高（一条顶多条） | 可能更低，靠编译器优化 |

## 自检

- [ ] 能用一句话区分 RISC / CISC
- [ ] 知道「乘法可访存」为何指向 CISC
- [ ] 能举出 x86 vs ARM 的例子

## 相关

- [指令](指令.md)
- [机器语言](机器语言.md)
- [汇编指令](汇编指令.md)
- [函数调用](函数调用.md)
