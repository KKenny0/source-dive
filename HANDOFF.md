# Source Dive — Handoff Document

> 本文档为 source-dive skill 的交接文档。新 session 读此文件即可完整接手，无需其他上下文。同时作为 README 撰写的素材来源。

## 1. 这是什么

source-dive 是一个 Claude Code skill，自动化「技术主题深度研究 → 中文长文」的全流程。输入一个技术主题名（如 "Hermes Agent"）加上可选的 GitHub repo URL、官方文档 URL，输出一篇完整的、自包含的中文技术深度解析文章。

## 2. 为什么做这个

这个 skill 从一次实际研究中提取出来。那次研究的对象是 Hermes Agent（Nous Research 的 AI Agent 框架），工作流程经过验证：

1. 抓取官方文档 + 定位 GitHub repo
2. 用 background agents 并行分析大源码文件（run_agent.py 211KB, hermes_state.py 152KB, gateway/run.py 510KB）
3. 提取架构模式，记录文档 vs 源码差异
4. 规划 8 章结构，逐章写作
5. 去 AI 味审查后输出

最终产出约 15000 字的深度解析长文。整个流程可复用，所以固化成 skill。

实践产出的文章位于：`D:\VibeProject\knowledge-vaults\ai-wiki-vault\03_Content_Output\Longform\hermes-agent-deep-dive_2026-05-27.md`

## 3. 文件清单

```
D:\VibeProject\skills\source-dive\
├── SKILL.md                    主入口。触发条件、输入输出规范、三阶段流程概览、关键约束、依赖工具、gotchas
├── HANDOFF.md                  本文件。交接文档 + README 素材
└── Workflows/
    ├── 01-discover.md          Phase 1 发现阶段（7 步）：搜索 repo → 抓取文档 → 识别源码文件 → 汇总发现
    ├── 02-analyze.md           Phase 2 分析阶段（6 步）：启动并行 agents → 提取模式 → 构建知识地图 → 规划章节
    └── 03-compose.md           Phase 3 撰写阶段（6 步）：确认章节 → 逐章写作 → 去 AI 味审查 → 输出文件
```

## 4. 关键设计决策

### 4.1 规则内联，零外部依赖

所有依赖 skills 的核心规则直接内联到 `03-compose.md`，不运行时调用其他 skills。

**内联了什么**：
- **规则 A**（文章风格）：来自 `technical-writing` skill（`D:\VibeProject\knowledge-vaults\.claude\skills\technical-writing\SKILL.md`）。包含核心原则、文章结构模板、语气规范、用户历史约束。
- **规则 B**（去 AI 味）：来自 `write-zh-prose`（`C:\Users\Administrator\.claude\skills\write\references\write-zh-prose.md`）。完整 10 条 AI 痕迹检查清单 + 句式规则 + 用词去正式化表。
- **规则 C**（Q-A 预处理）：来自 `ljg-qa`（`C:\Users\Administrator\.claude\skills\ljg-qa\References\QuestionDesign.md`）。Q 的四类/禁忌/语气，A 的四段结构，标注为可选。

**为什么内联**：避免 skill 嵌套调用问题（路径不同、加载失败、版本不兼容）。代价是需要手动同步。

**同步要求**：当 `technical-writing`、`write`、`ljg-qa` 这三个 skill 更新时，需要手动检查 `03-compose.md` 中的内联规则是否需要同步。

### 4.2 放在 D:\VibeProject\skills\ 下

不放在 user-level（`~/.claude/skills/`）也不放在 project-level（`.claude/skills/`），而是放在独立的 skills 工作目录。

**需要确认**：Claude Code 是否已配置 `D:\VibeProject\skills\` 为 skill 搜索路径。如果没有，需要在 settings 中添加。

### 4.3 并行化是核心设计

Phase 2 明确要求用 background agents 并行分析大文件，主线程同时处理小文件。这是从 Hermes 研究中验证过的模式——4 个 background agent 同时跑，比串行快 3-4 倍。

### 4.4 源码优先于文档

当文档描述与源码实现不一致时，以源码为准，并在文章中标注「文档 vs 源码差异」。Hermes 研究中发现至少 6 处重大差异（schema 版本、方法分布、配置参数等），证明这条规则的必要性。

### 4.5 内联规则保语义优先

去 AI 味审查的元规则是「保语义 > 去 AI 味」。如果为「更口语」改坏了原意，属于失败改写。

## 5. 已知问题和优化方向

### 5.1 需要 Claude Code 路径配置

当前 `D:\VibeProject\skills\` 不在 Claude Code 默认的 skill 搜索路径中。需要确认 settings 配置，否则 source-dive 不会被自动发现。

### 5.2 Workflow 文件可进一步压缩

当前三个 workflow 文件内容有少量重复（如输出规范在 SKILL.md 和 03-compose.md 中都写了）。可以考虑用更精简的方式减少重复。

### 5.3 缺少实际测试

skill 刚创建，还没有用第二个技术主题跑过一遍验证。建议下次找一个不同类型的项目（如前端框架、数据库、CLI 工具）试跑一次，根据实际执行情况调整 workflow 步骤。

### 5.4 Q-A 预处理步骤可能需要调整

Hermes 研究中 Q-A 预处理被跳过了（直接写作更高效）。当前设计中它被标注为可选，但触发条件（"3+ 个相互关联的机制"）可能不够精确。需要试跑后根据实际效果调整。

### 5.5 缺少 README.md

skill 目录下没有 README.md。本 HANDOFF 文档可以作为 README 的素材来源。

### 5.6 内联规则可能过时

`03-compose.md` 中的内联规则快照于 2026-05-27。如果原 skill 更新，这里不会自动同步。建议每次使用前快速检查一次。

## 6. Hermes 研究中的具体经验

以下是设计 workflow 时参考的实际经验，供优化时参考：

| 经验 | 对应 workflow 步骤 | 备注 |
|------|---------------------|------|
| 文档页面 429 限流 | 01-discover Step 4 | 用 webReader 替代 WebFetch |
| 源码文件 >200KB 无法一次读取 | 02-analyze Step 2 | 保存到本地 temp，background agent 分段分析 |
| Background agent 耗时 100-400s | 02-analyze Step 3 | 主线程先分析小文件，不空等 |
| 文档 vs 源码差异是文章核心价值 | 03-compose Step 2.3 | 以源码为准，差异单独标注 |
| 去AI味需要逐章检查 + 全文复查 | 03-compose Step 2.4 + Step 3 | 两遍检查比一遍更干净 |
| 每章写完立即去 AI 味比攒到最后改效率高 | 03-compose Step 2.4 | 减少全文级返工 |
| run_agent.py 是 facade，192 个方法中 51 个是纯转发 | 02-analyze Step 2 | 大文件中大量方法是噪音，agent prompt 要强调提取核心 |
| 用户学到的约束（不过度口语化、加信息来源） | 03-compose 规则 A | 来自 technical-writing 的 evolution.json |

## 7. 目录结构参考（输出目标）

skill 默认输出到 ai-wiki-vault 的 Obsidian vault 结构：

```
D:\VibeProject\knowledge-vaults\ai-wiki-vault\
├── 02_Topic_Notes/              Topic Note 输出目录
│   └── {分类}/
│       └── {date}_{主题}.md
├── 03_Content_Output/
│   └── Longform/                长文输出目录
│       └── {主题}-deep-dive_{date}.md
└── 00_Index/                    索引文件（可选更新）
    ├── TOPIC_INDEX.md
    ├── TIMELINE_INDEX.md
    └── CONNECTION_INDEX.md
```
