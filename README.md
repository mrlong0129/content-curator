# Content Curator

> RSS/X 内容策展助手 — IvanOS Teammate

信息采集 + 洞察筛选 + 知识关联 + 沉淀推荐。

**核心原则**：推送穿越时间的洞察，不是新闻。

---

## 它做什么

1. **采集** — 监控 100+ 订阅源（X 账号 + 博客 RSS + Karpathy HN 86 feeds）
2. **分类** — 6 维度评估每条内容（时效性/领域/深度/关联性/新颖度/know-how连接）
3. **过滤** — 丢弃噪音，只推高信号：`timeless + deep = 必推`，`ephemeral + surface = 丢弃`
4. **关联** — 每条内容关联已有 know-how，标注 🆕 新发现 / ✅ 验证已知 / ⚠️ 矛盾 / 🔄 更新
5. **沉淀** — 建议哪些值得存入 know-how 或 memory
6. **进化** — 基于反馈调整源评分和筛选模式

---

## 文件结构

```
content-curator/
├── README.md                           # 本文件
├── teammate/
│   ├── content-curator.md              # Teammate 角色定义（JD、工作流、评价标准）
│   ├── feed-registry.yaml              # 订阅源注册表（~105 feeds）
│   ├── know-how.md                     # 个人认知库（源质量、筛选模式、分歧）
│   ├── reflections.md                  # 执行反思
│   └── execution-log.md               # 执行历史
├── skills/
│   ├── rss-feed-monitor.md             # 博客 RSS 采集方法论
│   ├── x-timeline-monitor.md           # X/Twitter 采集方法论
│   └── content-classification.md       # 6 维度分类框架
├── commands/
│   ├── curate.md                       # /curate 命令定义
│   └── feed.md                         # /feed 命令定义
├── agent/
│   └── teammate-content-curator.md     # Claude Code Agent 定义
└── data/
    └── feed-state.json                 # 采集状态追踪
```

---

## 使用方式

### 作为 IvanOS Teammate

将文件部署到 IvanOS 对应目录：

```
teammate/content-curator.md         → capabilities/teammates/
teammate/feed-registry.yaml         → capabilities/teammates/content-curator/
teammate/{know-how,reflections,execution-log}.md → capabilities/teammates/content-curator/
skills/*.md                         → capabilities/skills/content-acquisition/
commands/*.md                       → .claude/commands/
agent/*.md                          → .claude/agents/
data/feed-state.json                → outputs/data/content-curator/
```

### 命令

```
/curate                  # 全量采集 + 筛选 + digest
/curate --blogs          # 仅博客
/curate --x              # 仅 X 账号
/curate --topic [kw]     # 话题检索

/feed list               # 列出订阅源
/feed add [url]          # 新增
/feed score [id] [n]     # 调整评分
```

---

## 订阅源

### 分类

| Category | 类型 | 来源 |
|----------|------|------|
| `x-pro` | X 账号 | AI/ML 权威（@karpathy, @ilyasut, @drfeifei 等） |
| `x-cn` | X 账号 | 中文圈 AI/Tech（@dotey, @foxshuo, @9hills 等） |
| `blog-curated` | 博客 | 亲选博客（stephango.com, moretothat.com 等） |
| `blog-karpathy` | 博客 | Karpathy 推荐的 HN 热门博客（86 feeds） |

### Attention 评分

- `attention_ivan` (0-5) — 人工评分，稳定
- `attention_curator` (0-5) — curator 学习的质量评估，动态
- **有效优先级** = `ivan * 0.6 + curator * 0.4`

---

## 输出：Digest 报告

```markdown
# Content Digest: YYYY-MM-DD

## Highlights（必读）
高 attention 源 + timeless + deep 的内容

## Worth Reading（推荐）
按领域分组的推荐阅读

## Signals（趋势）
多源同时讨论的话题

## 沉淀建议
值得存入 know-how 或 memory 的内容
```

---

## 反馈循环

| Ivan 反馈 | 系统响应 |
|-----------|---------|
| "好" | +0.2 源评分，记录内容特征 |
| "噪音" | -0.3 源评分，收紧过滤 |
| "漏了" | 扩大采集范围 |
| "太多了" | 提高过滤阈值 |
| 3 次无互动 | 主动建议降级 |

---

## 依赖

- **Claude Code** — 执行环境
- **WebFetch** — RSS 抓取
- **Chrome MCP / Puppeteer** — X 内容采集
- **IvanOS know-how/** — 知识关联基准

---

## 版本

- **V1.0** (2026-02-12) — 初始版本
- **V2.0** (planned) — 定时采集、Newsletter 集成、话题聚类、know-how 盲区检测

---

*Part of [IvanOS](https://github.com/mrlong0129/ivanos) ecosystem*
