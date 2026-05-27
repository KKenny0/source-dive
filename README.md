# Source Dive

给一个技术主题名，自动产出中文深度解析长文。

从 GitHub 源码和官方文档出发，跑一遍三阶段流水线（发现、分析、撰写），最终输出一篇自包含的技术文章。所有规则内联，不依赖其他 skill。

这是一个 Claude Code skill。它存在的原因：Hermes Agent 的源码研究走通了一次完整流程，4 个 background agent 并行啃 200KB+ 的大文件，发现 6 处文档和代码对不上的地方，最终产出 15000 字的长文。流程可复用，所以固化了。

## 怎么触发

在 Claude Code 里说类似这些话：

- `source dive Hermes Agent`
- `深度研究一下 LlamaIndex`
- `帮我看看 React 的架构`
- `写一篇关于 Vite 的技术文章`
- 给一个 GitHub URL + "分析它的实现原理"

可以附上 GitHub repo URL、官方文档 URL、论文 URL，不附也行，会自动搜。

## 三阶段流水线

```
Phase 1: 发现          Phase 2: 分析           Phase 3: 撰写
搜索 repo              启动 background agents   确认章节规划
抓取官方文档     ──→   并行分析源码文件    ──→   逐章写作
识别核心文件           提取架构模式             去 AI 味审查
                       记录文档 vs 源码差异     输出 .md 文件
```

每个阶段的详细步骤在 `Workflows/` 目录下：
- `01-discover.md`，搜索 repo、抓文档、标记要分析的源码文件
- `02-analyze.md`，并行分析 + 构建知识地图 + 规划章节
- `03-compose.md`，逐章写作 + 10 条 AI 痕迹检查 + 输出

## 需要的工具

| 工具 | 干什么 |
|------|--------|
| WebSearch / WebFetch | 搜索 repo 和文档 |
| mcp__zread__* | 读 GitHub 仓库结构和源码 |
| mcp__web-reader__webReader | 抓文档页面（WebFetch 被 429 限流时的替代） |
| Agent (background) | Phase 2 并行分析大文件 |

## 输出

文件名格式：`{主题}-deep-dive_{YYYY-MM-DD}.md`

默认写到 `03_Content_Output/Longform/`，也可以在触发时指定路径。文件开头带 YAML front matter（title、date、tags、source_repo、status）。

## 几个设计上的选择

**源码优先**，文档说的和代码写的不一致时以代码为准，差异在文章里标出来。Hermes 研究中发现了 6 处这种不一致。

**并行分析**，Phase 2 用 background agents 同时处理大文件，主线程同时啃小文件，不空等。实测 4 个 agent 并行比串行快 3-4 倍。

**零外部 skill 依赖**，写作风格、去 AI 味、Q-A 预处理的规则全部内联在 `03-compose.md`，运行时不调用其他 skill。代价是原 skill 更新时需要手动同步内联规则。

## 文件结构

```
source-dive/
├── SKILL.md              主入口：触发条件、输入输出规范、约束
├── README.md             本文件
├── HANDOFF.md            设计背景和交接信息
└── Workflows/
    ├── 01-discover.md    Phase 1：发现信息源
    ├── 02-analyze.md     Phase 2：并行分析 + 知识骨架
    └── 03-compose.md     Phase 3：撰写 + 去AI味审查 + 输出
```
