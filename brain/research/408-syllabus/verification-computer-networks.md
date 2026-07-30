---
date: 2026-07-11
description: 计算机网络拆分审阅V1-V8
status: provisional
tags:
  - research
  - "408"
  - syllabus
  - verification
  - 计算机网络
title: 408考纲审阅·计算机网络
---
# 408 考纲审阅 · 计算机网络

对照：[计网拆分](./computer-networks.md) ↔ 聚英 2025 PDF 计网段 + from0to1。审阅日：2026-07-11。

## V1–V8

| # | 结果 | 说明 |
| --- | --- | --- |
| V1 | PASS | 主源各最低层条目均有 CN-*；HDLC/网桥为 from0to1 补录并标 uncertain |
| V2 | PASS | 抽查：VLAN、SDN、GBN/SR、滑动窗口、ICMP、三次握手相关条；OCR 原样保留在「原文」栏 |
| V3 | PASS | 考查目标含 OCR 原文块 + 理顺块 |
| V4 | PASS | 统一五栏：原文/应会/scope/版本差/备注 |
| V5 | PASS | 慢开始等 TCP 拥塞具名、IMAP、Cookie 等已标 borderline/uncertain |
| V6 | PASS | VLAN/SDN 仅 2025；HDLC/网桥/令牌环细写仅 from0to1；滑轮→滑动已记 |
| V7 | PASS | 文首 OCR 对照表；源缺编号「3.」已注 |
| V8 | PASS | 写本报告后审阅自链应闭合 |

## 结论

`verified-conditional`：对照聚英转载通过细拆门禁；**仍非教育考试院官方 PDF**。

## 挂载

- [拆分正文](./computer-networks.md) · [流水线](../408-syllabus-pipeline.md) · [数据结构审阅样板](./verification-data-structures.md)

## 2026 转载复核（2026-07-12 追加）

对照新源：[新东方2026转载](../../external-sources/408-syllabus-xdf-2026-derivative.md)。

| 检查 | 结果 |
| --- | --- |
| 章结构（六章） | 与聚英2025 一致：概述/物理层/数据链路层/网络层/传输层/应用层 |
| **2026 唯一标记改动** | 「ISO/OSI 参考模型和 TCP/IP 参考模型【26 改动】」→ 疑为「TCP/IP 模型」→「TCP/IP 参考模型」措辞变化；已写入 CN-1.6 版本差，scope 不变（in-syllabus），具体以官方 PDF 裁定 |
| 其他叶子 | SDN、VLAN（新源错字 VIAN 已逐字保留）、移动 IP、IP 组播、BGP 等与 2025 一致，无增删 |

结论不变：`verified-conditional`。

## 同文件整合与学习深化复核（2026-07-12）

- 原独立深化指南的重点、难点、易错点、8 道选择题、4 道综合题已完整并入 [计网主文件](./computer-networks.md)，不再保留第二份科目文档。
- 覆盖 CN-1～CN-6；重点/难点/易错按章与考点同文件排列，未改动上半部叶子级考纲原文及 scope 判定。
- P0/P1/P2 已明确为本库复习优先级，不构成官方频次或分值声明。
- 主文件显著保留“官方 PDF 尚未取得、`verified-conditional`、不代表最终官方范围”警告。
- 复算存储转发时延、IPv4 分片、最长前缀匹配和 TCP cwnd 序列，与原验证结论一致。
