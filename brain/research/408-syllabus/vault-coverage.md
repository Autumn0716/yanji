---
date: 2026-07-11
description: 考纲叶子到vault/408笔记的覆盖矩阵与落库规则
status: provisional
tags:
  - research
  - "408"
  - syllabus
  - vault
  - coverage
title: 408 vault覆盖矩阵
---
# 408 vault 覆盖矩阵

> 每个考纲叶子必须映射到 ≥1 个 `vault/408/**` 笔记，且笔记正文含可复习知识点（非空壳）。
> **规则**：已有笔记只 `append`；新建笔记写完整知识点。每批更新后跑覆盖验证。

主拆分：[数据结构](./data-structures.md) · [组成原理](./computer-organization.md) · [操作系统](./operating-systems.md) · [计算机网络](./computer-networks.md)

## 现状盘点（2026-07-11）

| 科目 | vault 现状 | 缺口 |
| --- | --- | --- |
| 数据结构 | 仅 `线性表/`（线性表、数组） | 基本概念/栈队列/树/图/查找/排序 全缺 |
| 计组 | CPU/存储/指令/总线/I/O/运算 有若干笔记 | 需按 CO-* 叶子对账补洞 |
| 操作系统 | **目录空** | 全科新建 |
| 计算机网络 | 应用层、编码方式 | 概述/物理层大部/链路/网络/传输 缺 |

## 计算机网络映射（本批完成）

| 叶子 | vault | 状态 |
| --- | --- | --- |
| CN-1 | [概述](../../../vault/408/计算机网络/概述.md) | done |
| CN-2 | [物理层](../../../vault/408/计算机网络/物理层.md)+[编码方式](../../../vault/408/计算机网络/编码方式.md) | done |
| CN-3 | [数据链路层](../../../vault/408/计算机网络/数据链路层.md) | done |
| CN-4 | [网络层](../../../vault/408/计算机网络/网络层.md) | done |
| CN-5 | [传输层](../../../vault/408/计算机网络/传输层.md) | done |
| CN-6 | [应用层](../../../vault/408/计算机网络/应用层.md) 追加 | done |

审阅：[verification-vault-computer-networks](./verification-vault-computer-networks.md)

## 操作系统映射（已完成）

| 叶子范围 | vault 路径 | 状态 |
| --- | --- | --- |
| OS-1.* | [概述](../../../vault/408/操作系统/概述.md) | done |
| OS-2.* | [进程管理](../../../vault/408/操作系统/进程管理.md) | done |
| OS-3.* | [内存管理](../../../vault/408/操作系统/内存管理.md) | done |
| OS-4.* | [文件管理](../../../vault/408/操作系统/文件管理.md) | done |
| OS-5.* | [输入输出管理](../../../vault/408/操作系统/输入输出管理.md) | done |

审阅：[verification-vault-operating-systems](./verification-vault-operating-systems.md)

## 数据结构映射（已完成）

| 叶子范围 | vault 路径 | 状态 |
| --- | --- | --- |
| DS-1.* | [基本概念](../../../vault/408/数据结构/基本概念.md) | 本批新建 |
| DS-2.* | [线性表](../../../vault/408/数据结构/线性表/线性表.md) + [数组](../../../vault/408/数据结构/线性表/数组.md) | 追加补全 |
| DS-3.* | [栈队列与数组](../../../vault/408/数据结构/栈队列与数组.md) | 本批新建 |
| DS-4.* | [树与二叉树](../../../vault/408/数据结构/树与二叉树.md) | 本批新建 |
| DS-5.* | [图](../../../vault/408/数据结构/图.md) | 本批新建 |
| DS-6.* | [查找](../../../vault/408/数据结构/查找.md) | 本批新建 |
| DS-7.* | [排序](../../../vault/408/数据结构/排序.md) | 本批新建 |

## 验证字段（W7）

每科完成后写 `verification-vault-<subject>.md`：

- C1 每个 in-syllabus 叶子在 vault 有锚点章节
- C2 未删除任何迁入原文段落（diff 只增）
- C3 知识点可复习（定义/性质/复杂度/注意点至少一类）
- C4 链回对应 DS/CO/OS/CN 叶子 id
- C5 死链清零
