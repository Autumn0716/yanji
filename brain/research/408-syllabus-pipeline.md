---
title: 408 考纲流水线（进度与量规）
description: 408 四科考纲 ingest→拆分→校验 的可续跑工作流检查点
status: provisional
date: 2026-07-11
tags:
  - research
  - "408"
  - syllabus
  - workflow
  - provisional
---
# 408 考纲流水线

> [!NOTE]
> 动态 loop 推进中。规则：**先 ingest 原文，再拆分加工**；模糊/疑似超纲一律入库并标注。

## Question

如何把教育部教育考试院体系下的《计算机学科专业基础》（408）考纲，按四科完整、可追溯地写入本知识库？

## 量规（已由用户任务确认）

1. **忠于原文**：官方/权威来源先入 `brain/external-sources/`，再加工
2. **四科全覆盖**：数据结构 / 组成原理 / 操作系统 / 计算机网络；每科详细考点
3. **深度拆分**：考点树进 `brain/research/408-syllabus/`
4. **vault 落库（主交付）**：`vault/408/` 按章节写入**可复习知识点**，覆盖全部考纲叶子；**已有正文只追加补充，禁止删除/覆盖原文段落**
5. **校验强制**：拆分审阅 + vault 覆盖审阅；缺验证 = 未完成
6. **模糊信息**：`uncertain | borderline | beyond` 一律入库并标注

## 进度

| 阶段 | 状态 | 说明 |
| --- | --- | --- |
| W0 骨架 | done | 进度文档 + skill 已 install |
| W1 扫描 | done | 无既有考纲专文 → Path A |
| W2 ingest 原文 | in_progress | 唯一 Markdown 考纲源为新东方2026；from0to1/聚英的 provenance 已并入，三份原始 HTML/PDF 均保留；官方 PDF 仍缺 |
| W3 四科拆分 | done | 四科叶子级已有 |
| W4 拆分校验 | done（条件） | 四科均 verified-conditional；CO/OS 格式债已于 2026-07-12 返工清除（五栏统一） |
| W5 知识地图挂载 | done | 已链拆分文档 |
| W6a 数据结构 vault | done | [verification-vault-data-structures](./408-syllabus/verification-vault-data-structures.md) |
| W6b 操作系统 vault | done | [verification-vault-operating-systems](./408-syllabus/verification-vault-operating-systems.md) |
| W6c 计网 vault | done | [verification-vault-computer-networks](./408-syllabus/verification-vault-computer-networks.md) |
| W6d 计组 vault | done | [verification-vault-computer-organization](./408-syllabus/verification-vault-computer-organization.md) |
| **W6 vault 全覆盖** | **done（首轮）** | 四科均有 vault 章级覆盖+审阅；可继续加深 |
| W7 vault 验证 | done（首轮） | 四科均有独立 vault 覆盖审阅；后续新增内容继续增量复核 |
| W8 四科知识深化 | done（条件） | 重点/难点/易错与32道选择题+15道综合题已并入四科主文件；验证分别并入四科 verification |

### 产物

- 唯一 Markdown 考纲源：[新东方2026转载全文](../external-sources/408-syllabus-xdf-2026-derivative.md)；三份原始证据 `_raw-xdf-408-2026.html`、`_raw-juying-408.pdf`、`_raw-from0to1-408.html` 均保留
- 拆分：[数据结构](./408-syllabus/data-structures.md) · [组成原理](./408-syllabus/computer-organization.md) · [操作系统](./408-syllabus/operating-systems.md) · [计算机网络](./408-syllabus/computer-networks.md)
- Skill：`408-syllabus-pipeline`

### 校验备忘（初步）

- 转载文「滑动窗口」处写作「滑轮窗口」→ 疑似笔误，标 uncertain，不改正文
- 本转载 OS/部分结构偏「旧版叙述」；公开检索另有「信号（2025新增）」等说法 → **必须以官方 PDF 裁定**，差异进「疑似超纲/版本差」表
- 2026-07-12 复核：新东方2026转载与聚英2025 结构一致；计网唯一【26改动】标在「ISO/OSI 参考模型和 TCP/IP 参考模型」（疑措辞变化，已记 CN-1.6）；四科审阅均已追加「2026 转载复核」节
- NEEA 研考网（yankao.neea.edu.cn）考试大纲栏目已直接抓取验证：公开页面最新仅 2022 年条目，且页面无正文、无 PDF 附件链接 → 官方原件需线下/高教社《考试大纲》纸质版，网络不可得；全库结论维持 verified-conditional

## 后续待补来源

- 教育部教育考试院/高教社发布的最新官方 408 PDF（当前网络不可得）；取得后立即 ingest，并重新裁定全部 `verified-conditional` 结论。

## Open questions

- 最新年份官方 PDF 是否可直接下载并 `ingest`（二进制）？
- 转载全文与官方 PDF 字句差异如何处理？（以官方 PDF 为准，转载标 `derivative`）

## 细拆规范（强制 · 2026-07-11 起）

禁止只写「章级一览表」。每科拆分必须满足：

1. **叶子 = 考纲原文最低一层条目**（如「邻接矩阵」「折半查找法」），不可合并成「图的存储」糊弄
2. 每个叶子固定字段：`id` / `原文` / `应会` / `scope` / `版本差` / `备注`
3. **忠于原文**：`原文` 栏逐字（含 OCR 原样）；理顺只写在「理顺」或备注，禁止只给润色版冒充原文
4. 原文只有「基本概念」而无展开时：先保留叶子，再单开「教材常见展开」并标 `uncertain`/`borderline`
5. 禁止缩略表代替叶子（排序算法等必须逐叶全字段）

## 写入后审阅门禁（强制 · 缺审阅 = 未完成）

每科 `write` 之后必须另写 `verification-<subject>.md`，跑完 V1–V8 才能标 `verified-conditional`：

| # | 检查 |
| --- | --- |
| V1 | 主源叶子清单 vs 文档 id 一一对应 |
| V2 | `原文` 与转载/PDF 逐字一致 |
| V3 | 考查目标：原文块与理顺块分离 |
| V4 | 全文字段格式统一 |
| V5 | 应会中的具名扩展已标 scope |
| V6 | 版本差完整 |
| V7 | OCR/章节重号已注明 |
| V8 | 无死链 |

已有审阅：[数据结构](./408-syllabus/verification-data-structures.md) · [组成原理](./408-syllabus/verification-computer-organization.md) · [操作系统](./408-syllabus/verification-operating-systems.md) · [计算机网络](./408-syllabus/verification-computer-networks.md)。

## 下一步

四科拆分、同文件知识深化（重点/难点/易错点已按章内联至各章末尾）、vault 首轮覆盖和对应验证均已完成。`brain/research/408-syllabus/` 仅保留四科主文件 + 各 verification + vault-coverage；考纲 Markdown 唯一源为 `brain/external-sources/408-syllabus-xdf-2026-derivative.md`。当前唯一未闭环项是官方 PDF：取得后应先原样 ingest，再逐叶复核版本差、scope 与 `verified-conditional` 状态。
