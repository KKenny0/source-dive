# Phase 1: 发现

收集所有可用的信息源，为后续分析准备素材。

## Step 1: 确认输入

确认用户提供的信息：

- 主题名（必填）
- GitHub repo URL（可选）
- 官方文档 URL（可选）
- 论文 URL（可选）

如果用户只提供了主题名，先做 web search 补全。

## Step 2: 搜索与定位

按优先级搜索信息源。多个搜索可以并行发起：

**搜索 1 — GitHub repo**:
- 如果用户提供了 `repo_url`，直接使用
- 否则用 WebSearch 搜索 `{topic} github`
- 确认找到的 repo 是官方/主仓库（看 stars、最近活跃度）

**搜索 2 — 官方文档**:
- 如果用户提供了 `docs_url`，直接使用
- 否则用 WebSearch 搜索 `{topic} documentation` 或 `{topic} official docs`

**搜索 3 — 论文**（可选）:
- 如果主题是学术项目，搜索 arxiv
- 用 WebSearch 搜索 `{topic} arxiv` 或 `{topic} paper`

**搜索 4 — 社区资源**（补充）:
- 搜索技术博客、教程作为补充视角
- 搜索 `{topic} deep dive` 或 `{topic} 源码分析` 看是否已有类似文章

## Step 3: 获取 repo 结构

确认 GitHub repo 后，用 `mcp__zread__get_repo_structure` 获取目录结构。

重点关注：
- 项目根目录的 README.md、CONTRIBUTING.md 等入口文件
- 核心代码目录（src/, lib/, agent/, core/ 等）
- 配置文件（config/, settings, pyproject.toml, package.json 等）
- 文档目录（docs/, doc/）

逐层深入关键目录，直到看清整体结构。

## Step 4: 抓取核心文档

用 `mcp__web-reader__webReader` 抓取官方文档的关键页面。优先抓取：

1. **Getting Started / Overview** — 项目定位和核心概念
2. **Architecture / Design** — 系统架构和设计理念
3. **API Reference** — 核心接口和行为
4. **Key guides / tutorials** — 关键用法和工作流

每个页面抓取后，在主线程中快速浏览，提取核心内容摘要。

如果遇到 429 限流，切换到 WebFetch 或加 2-3 秒延迟。

## Step 5: 识别关键源码文件

基于 repo 结构，识别需要深入分析的核心源码文件。

**文件选择标准**：
- 入口文件：main.py, run.py, app.py, index.ts 等
- 核心逻辑：看文件名和目录结构，找最可能包含核心机制的文件
- 状态管理：数据库、缓存、session 相关文件
- 配置与常量：config, settings, constants
- 工具/插件注册：registry, plugin, tool 相关文件

**按大小分类标记**：

| 大小 | 标记 | 后续处理 |
|------|------|----------|
| < 50KB | `small` | Phase 2 主线程直接读取分析 |
| 50-200KB | `medium` | Phase 2 background agent 分析 |
| > 200KB | `large` | Phase 2 保存到本地后 background agent 分析 |

**数量控制**：标记 5-15 个文件。超过 15 个时，按"核心程度"排序取前 15。

## Step 6: 读取小文件

在等待大文件处理前，先在主线程直接读取标记为 `small` 的文件：
- 用 `mcp__zread__read_file` 读取 GitHub 上的文件内容
- 或用 Read 读取已保存到本地的文件
- 快速提取核心类名、函数名、关键常量

## Step 7: 汇总发现

输出发现摘要，作为 Phase 2 的输入：

```
## 发现摘要

**主题**: {topic}
**GitHub repo**: {url} ({stars} stars, {last_commit_date})
**官方文档**: {url} ({N} 个关键页面已抓取)
**论文**: {url} (如有)

**关键源码文件** ({N} 个):
- {file_path} ({size}, {small/medium/large}) — {一句话描述}
- ...

**官方文档核心内容**:
- {page_title}: {2-3 句话摘要}
- ...

**初步观察**:
- {观察 1}
- {观察 2}
- ...
```

完成后进入 [Phase 2: 分析](02-analyze.md)。
