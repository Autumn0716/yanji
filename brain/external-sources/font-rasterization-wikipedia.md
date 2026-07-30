---
date_fetched: 2026-07-11
description: 字体光栅化、抗锯齿与次像素渲染要点备份。
preservation: text-extracted
source_url: https://en.wikipedia.org/wiki/Font_rasterization
tags:
  - source
  - immutable
  - layer-ingest
  - text
  - font
  - antialiasing
title: Font rasterization (Wikipedia 摘录备忘)
---
> 摘录自 Wikipedia Font rasterization 页（2026-07-11）。完整原文见 source_url。

- 字体光栅化：把矢量字形（如 TrueType）转成屏幕上的光栅/点阵描述。
- 最简单是双色（黑白）描边，无中间灰度，边缘呈锯齿（aliased / jagged）。
- 抗锯齿：按像素被字形覆盖的比例填灰度，边缘更平滑；可配合 hinting 把笔画尽量对齐像素网格以免发糊。
- 次像素渲染：利用 LCD 等每个像素含 R/G/B 子像素，在子像素级提高有效水平分辨率；微软 ClearType 是其一例。
- 现代系统通常由共享库完成光栅化（Windows DirectWrite、macOS Quartz、FreeType 等）。
