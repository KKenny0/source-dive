---
name: source-dive
description: >
  全自动流水线：从技术主题的官方文档 + GitHub 源码出发，经过发现、并行分析、结构化撰写，
  输出一篇完整的中文技术深度解析长文。零外部 skill 依赖，所有核心规则已内联。
  当用户想要深入研究某个技术项目、框架、工具、库的内部实现时，都应该使用这个 skill，
  即使用户没有明确说"写文章"。比如用户说"帮我看看 X 的架构"、"研究一下 X 的实现原理"、
  "X 的源码是怎么工作的"、"深入分析 X 的设计"、"分析 X 的技术原理"、
  "写一篇关于 X 的技术文章"、"深度研究 X"、"source dive X"，
  或者给了一个 GitHub URL 并要求理解其内部机制，都应该触发。
---

# Source Dive

从源码 + 文档到中文技术深度解析长文的全自动流水线。

## 你不是什么

- 不是通用研究笔记工具
- 不是 ljg-qa 的替代（Q-A 提取是 compose 阶段的可选子步骤）
- 不是自动发布工具
- 不生成代码，只生成文章

## 你是什么

一个三阶段流水线：**发现 → 分析 → 撰写**。输入一个技术主题名（+可选的 repo/docs URL），输出一篇完整的、自包含的中文技术深度解析文章。

## 触发条件

用户想要深入了解某个技术项目的内部实现时触发，包括但不限于：

- "深度研究一下 X" / "深入研究 X"
- "写一篇 X 的深度解析" / "写一篇关于 X 的技术文章"
- "source dive X"
- "帮我研究 X 并写成文章"
- "分析 X 的源码写篇长文"
- "帮我看看 X 的架构" / "X 的架构是怎样的"
- "研究一下 X 的实现原理" / "X 是怎么实现的"
- "X 的源码是怎么工作的" / "分析 X 的源码"
- "深入分析 X 的设计" / "分析 X 的技术原理"
- 给了 GitHub URL 并要求理解其内部机制

## 输入参数

| 参数 | 必填 | 说明 |
|------|------|------|
| topic | 是 | 研究主题名，如 "Hermes Agent"、"LlamaIndex" |
| repo_url | 否 | GitHub 仓库 URL，不提供则自动搜索 |
| docs_url | 否 | 官方文档站 URL，不提供则自动搜索 |
| paper_urls | 否 | 论文 URL 列表 |
| output_dir | 否 | 输出目录，默认按项目 vault 结构 |

## 输出规范

**文件命名**: `{主题}-deep-dive_{YYYY-MM-DD}.md`

**输出路径**: 由当前项目 vault 结构决定：
- 长文：`03_Content_Output/Longform/{文件名}`（或用户指定路径）
- Topic Note：`02_Topic_Notes/{分类}/`（如适用）

**文件开头**必须包含 YAML front matter：

```yaml
---
title: {主题} 深度解析
date: {YYYY-MM-DD}
tags: [deep-dive, {主题标签}]
source_repo: {repo_url}
source_docs: {docs_url}
status: draft
---
```

## 三阶段流程

```
Phase 1: 发现          Phase 2: 分析           Phase 3: 撰写
┌──────────────┐      ┌──────────────┐       ┌──────────────┐
│ 搜索 repo     │      │ 并行分析源码  │       │ 规划章节结构  │
│ 抓取官方文档   │ ──→ │ 提取架构模式  │ ──→  │ 逐章写作      │
│ 识别核心文件   │      │ 识别可迁移模式│       │ 可选终章      │
│              │      │ 记录 doc vs   │       │ 工程迁移      │
│              │      │ 源码差异      │       │ 去 AI 味审查  │
└──────────────┘      └──────────────┘       └──────────────┘
```

详细步骤见各阶段 workflow：

- **Phase 1**: [01-discover.md](Workflows/01-discover.md) — 收集所有可用信息源
- **Phase 2**: [02-analyze.md](Workflows/02-analyze.md) — 并行源码分析 + 知识骨架构建
- **Phase 3**: [03-compose.md](Workflows/03-compose.md) — 结构化撰写 + 质量审查 + 输出

## 关键约束

- **并行化是核心**：Phase 2 必须使用 background agents 并行分析大文件，不要串行等待
- **源码优先于文档**：当文档描述与源码实现不一致时，以源码为准，并在文中标注差异
- **中文母语**：最终输出必须是自然的中文技术写作，不能有 AI 味
- **自包含**：文章必须自包含，读者不需要额外查阅源码就能理解核心机制
- **有据可查**：每个技术论断必须有源码或文档的证据支撑

## 依赖工具

| 工具 | 用途 | 阶段 |
|------|------|------|
| WebSearch / WebFetch | 搜索和抓取官方文档 | Phase 1 |
| mcp__zread__get_repo_structure | 获取 GitHub repo 目录结构 | Phase 1 |
| mcp__zread__read_file | 读取 GitHub repo 中的文件 | Phase 1-2 |
| mcp__web-reader__webReader | 抓取文档页面 | Phase 1 |
| Agent (background) | 并行分析大源码文件 | Phase 2 |
| Read / Grep / Glob | 本地文件分析 | Phase 2-3 |
| Write / Edit | 输出文章 | Phase 3 |

## Gotchas

| 问题 | 规则 |
|------|------|
| 文档页面 429 限流 | 用 mcp__web-reader__webReader 替代 WebFetch，或加延迟重试 |
| 源码文件太大无法一次读取 | 保存到本地 temp 文件，用 background agent 分段分析 |
| Background agent 慢（100-400s）| 先用已有信息开始写作，agent 结果回来后补充 |
| 用户没提供 URL | 自动 web search 补全，但要让用户确认搜索结果是否正确 |
| 目标没有公开 repo | 退化到纯文档分析，在文章中说明无源码验证 |
