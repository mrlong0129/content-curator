# RSS Feed Monitor

> 博客 RSS/Atom 订阅监控：解析 feed、检测新文章、存储结构化数据。

---

## 适用场景

- 定期检查订阅博客是否有新文章
- 首次添加新博客时发现 RSS URL
- 批量采集多个博客的最新内容

---

## 输入

| 参数 | 必填 | 示例 |
|------|------|------|
| feed-registry.yaml | 是 | `capabilities/teammates/content-curator/feed-registry.yaml` |
| feed-state.json | 是 | `outputs/data/content-curator/feed-state.json` |
| 过滤条件 | 否 | `--category blog-curated` / `--since 2026-02-01` |

---

## 输出

```
outputs/data/content-curator/YYYY-MM/raw/blogs/
├── {feed-id}_{date}.json     # 每个 feed 的新文章
└── ...

feed-state.json（更新）       # 记录每个 feed 的最后检查状态
```

---

## 核心流程

### 1. RSS URL 发现

对于 `rss_url` 为 null 的博客，按优先级尝试发现：

```
1. WebFetch 博客首页 HTML
2. 查找 <link rel="alternate" type="application/rss+xml"> 或 type="application/atom+xml"
3. 如未找到，尝试常见路径：
   - /feed
   - /feed.xml
   - /rss
   - /rss.xml
   - /atom.xml
   - /index.xml
   - /blog/feed
   - /blog.xml
4. 发现后更新 feed-registry.yaml 的 rss_url 字段
5. 仍未找到 → 标记 rss_url: "not_found"，后续用首页对比法
```

### 2. Feed 解析

```
对每个有 rss_url 的 feed：
  1. WebFetch rss_url
  2. 从 XML 中提取 entries：
     - title（标题）
     - link（原文 URL）
     - published / updated（发布时间）
     - summary / description（摘要，截取前 500 字）
     - author（作者）
  3. 对比 feed-state.json 中该 feed 的 last_post_date
  4. 筛选出 last_post_date 之后的新 entries
  5. 存储新 entries 到 outputs/data/content-curator/YYYY-MM/raw/blogs/
```

### 3. 无 RSS 回退

```
对 rss_url 为 "not_found" 的博客：
  1. WebFetch 首页
  2. 提取文章列表（标题 + 链接 + 日期）
  3. 对比 feed-state.json 中记录的已知文章
  4. 新文章 → 存储
```

### 4. 状态更新

每个 feed 处理完后，更新 `feed-state.json`：

```json
{
  "feeds": {
    "blog-stephango": {
      "last_checked": "2026-02-12T10:00:00Z",
      "last_post_date": "2026-02-10T00:00:00Z",
      "last_post_url": "https://stephango.com/...",
      "total_posts_seen": 45,
      "posts_this_period": 2,
      "status": "ok"
    }
  }
}
```

`status` 值：
- `ok` — 正常
- `no_rss` — 无 RSS，用首页回退
- `error` — 抓取失败（记录原因）
- `dormant` — 超过 90 天无新内容

---

## 优先级排序

按 attention 加权排序处理：

| 优先级 | 条件 | 处理方式 |
|--------|------|---------|
| P0 | attention_ivan >= 4.0 | 每次必查 |
| P1 | attention_ivan >= 3.0 或 attention_curator >= 3.5 | 每次查 |
| P2 | 其余 | 隔次查或按需查 |

---

## 常见问题

| 问题 | 处理 |
|------|------|
| RSS 返回 403/429 | 标记 status: "error"，下次延迟重试 |
| Feed 格式异常 | 尝试 Atom 和 RSS 两种解析 |
| 时区不一致 | 统一转 UTC |
| 内容太长 | 摘要截取前 500 字，完整内容存原文链接 |

---

## 关联

- **content-classification** — 采集后的内容送分类
- **content-curator** — 调用此 Skill 执行采集

---

*v1.0 | 2026-02-12*
