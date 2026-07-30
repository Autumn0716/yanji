---
title: Work Log
description: Append-only audit trail of changes to this knowledge base.
---

# Work Log

Append-only audit trail. Add one dated entry per turn that creates, edits, or restructures content. The knowledge-base skill describes what to log and the entry shape.

## 2026-07-11

- Moved empty `brain/external-sources/Untitled` → [`brain/research/openknowledge-usage`](./research/openknowledge-usage.md) (usage guide is not an ingest source).
- Wrote provisional guide: project structure (`external-sources` → `research` → `articles`) and how to use OK in this repo.
- Imported Obsidian vault → `vault/` (61 md, 76 images, 18 pdf, 5 excalidraw; excluded `.obsidian`/`.claude`/`.claudian`).
- Reformatted `vault/` for OK: `[[wiki-link]]` → relative markdown links, added `title`/`description`, fixed image/PDF embeds, supplemented stubs + LLM hub mermaid; folder frontmatter on `vault/LLM`.
- Added [`vault/408/计组/输入输出/输入输出`](../vault/408/计算机组成原理/输入输出/输入输出.md): 408 计组 I/O 四种控制方式与接口/中断/DMA/通道要点；挂到知识地图与总线相关。
- Fixed vault image embeds: paths with spaces failed in OK preview; rewrote to content-root absolute URL-encoded hrefs (`/vault/.../file.png`).
- Supplemented remaining short 408 notes (总线/编码/线性表/CPU/存储/指令等) with review outlines; added [`vault/408/00-知识地图`](../vault/408/00-知识地图.md).

## 2026-07-12

- 仅调整 [新东方2026年408考纲转载](./external-sources/408-syllabus-xdf-2026-derivative.md) 的 Markdown 层级与列表，保留原始字词、笔误和原始 HTML 存档。
- 新增四科深化指南：数据结构、计算机组成原理、操作系统、计算机网络；逐章补充重点、难点、易错点，并加入 32 道原创选择题与 15 道原创综合题。
- 新增四科深化指南验证记录（后已并入四科 verification，历史索引已删除），完成答案抽算、范围分层和死链检查；结论为 `verified-conditional`（官方 PDF 仍缺）。
- 更新 [408 知识地图](../vault/408/00-知识地图.md) 与四科考纲拆分页，挂载深化指南并修正操作系统目录旧说明。

- 严格子代理首轮拒绝后，四科指南均补显著官方 PDF 缺失声明，并加粗 Cache 题的 LRU 条件；同一代理复审为 **APPROVE**。

- 按“每科同文件维护”要求，将四份独立深化指南完整并入四科主文件：考纲叶子、应会知识、重点、难点、易错点、32 道选择题与 15 道综合题现在按科集中维护。
- 将 `brain/external-sources/` 中 408 考纲 Markdown 收敛为唯一的 [新东方2026转载全文](./external-sources/408-syllabus-xdf-2026-derivative.md)；聚英/from0to1 的来源元数据并入该文件，三份原始 HTML/PDF 证据全部保留。
- 删除 4 份已合并的 `*-study-guide.md` 与 2 份重复考纲 Markdown；旧统一验证入口后已删除，实质验证分别并入四科 verification。
- 更新 [408知识地图](../vault/408/00-知识地图.md)、[考纲流水线](./research/408-syllabus-pipeline.md) 和残余来源引用。整合前回滚点：`80608d0a19bd885942e929480f0a8e6132025af5`。

- 将四科主文件中「复习深化」区块按章内联：每章考纲叶子后附 `### 复习要点`（重点/难点/易错点），全科选择题与综合题保留在文末 `## 全科针对性练习`。
- 计组/OS/计网补回试卷定位区的范围警告与 `### 总体优先级` 表；删除冗余 `verification-study-guides` 历史索引。
