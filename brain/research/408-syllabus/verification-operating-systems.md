---
title: 408考纲审阅·操作系统
description: 操作系统拆分审阅V1-V8对照聚英2025 PDF
date: 2026-07-11
status: provisional
tags:
  - research
  - "408"
  - syllabus
  - verification
  - 操作系统
---
# 408 考纲审阅 · 操作系统

对照：[操作系统拆分](./operating-systems.md) ↔ 聚英 2025 PDF 操作系统段。审阅日：2026-07-11。

## V1–V8

| # | 结果 | 说明 |
| --- | --- | --- |
| V1 | PASS | 主源叶子（含 2025 新增：信号/多处理机调度/页框分配）均有 OS-*；管程等 from0to1 补录已标 uncertain |
| V2 | PASS | 抽查「信号（2025 新增）」「Memory-Mapped Files」「磨损均衡」等与源一致；OCR「信号鼠/条件变昌/OMA」已在文首声明 |
| V3 | **FAIL→本轮修** | 缺考查目标 OCR 原文块 |
| V4 | **FAIL 未清** | 与组原相同：缩略表缺五栏统一格式 → 格式债 |
| V5 | PASS | 调度算法具名清单来自 from0to1 补强并标明 |
| V6 | PASS | 管程 vs 条件变量、磁盘章位置等有版本差 |
| V7 | PASS | OCR 与新增标签已注 |
| V8 | 待写回后复查 | — |

## 结论

`verified-conditional-format-debt`：考点覆盖通过；**格式门禁未过**；V3 本轮修。

## 挂载

- [拆分正文](./operating-systems.md) · [流水线](../408-syllabus-pipeline.md) · [数据结构审阅样板](./verification-data-structures.md)

## 2026 转载复核（2026-07-12 追加）

对照新源：[新东方2026转载](../../external-sources/408-syllabus-xdf-2026-derivative.md)。

| 检查 | 结果 |
| --- | --- |
| 章结构（五章） | 与聚英2025 一致；唯新源首章题作「操作系统**基础**」而聚英/from0to1 作「操作系统**概述**」→ 措辞差异记 uncertain，不影响叶子 |
| 2025 新增叶子在 2026 仍在 | 进程间通信含「信号」；CPU 调度算法/多处理机调度；页框分配与回收；内存映射文件；条件变量；锁 —— 均已在拆分 OS-* 中 |
| 其他 | inode、硬/软链接、虚拟文件系统、挂载、SPOOLing、SSD 磨损均衡均在两源一致；新源含 OCR 式笔误「l/O 端口」已逐字保留 |

结论不变：`verified-conditional`。

## 格式返工复核（2026-07-12）

全文重写为标准五栏表：去除 238 处 `\|` 转义竖线、每叶补「字段/内容」表头、补齐「版本差/备注」栏（无差异处填「一致/—」）。叶子 id、原文、应会、scope 内容与首版逐字一致，仅格式归一；另在 OS-1.1/OS-2.8/OS-5.1 备注中补记新东方2026 对照信息。V4（格式统一）由 FAIL(格式债) 转 PASS；frontmatter `verification` 由 `verified-conditional-format-debt` 升为 `verified-conditional`。

## 同文件整合与学习深化复核（2026-07-12）

- 原独立深化指南的重点、难点、易错点、8 道选择题、4 道综合题已完整并入 [操作系统主文件](./operating-systems.md)，不再保留第二份科目文档。
- 覆盖 OS-1～OS-7；重点/难点/易错按章与考点同文件排列，未改动上半部叶子级考纲原文及 scope 判定。
- P0/P1/P2 已明确为本库复习优先级，不构成官方频次或分值声明。
- 主文件显著保留“官方 PDF 尚未取得、`verified-conditional`、不代表最终官方范围”警告。
- 复算 RR 调度、银行家算法、FIFO/LRU 缺页次数和多级索引容量，与原验证结论一致。
