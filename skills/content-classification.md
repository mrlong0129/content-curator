# Content Classification

> 内容分类与过滤：6 维度评估 + know-how 关联 + 过滤决策。

---

## 适用场景

- 对采集到的博客文章/X 推文进行多维度分类
- 决定哪些内容值得推送给 Ivan
- 关联内容与 Ivan 已有的 know-how
- 识别沉淀候选内容

---

## 输入

| 参数 | 必填 | 说明 |
|------|------|------|
| 内容列表 | 是 | 采集到的文章/推文（标题、摘要、来源、链接） |
| know-how 索引 | 是 | Ivan 的 know-how/ 目录结构及关键文件 |
| feed-registry | 是 | 源的 attention 评分和 signal_type |

---

## 输出

每条内容附加分类标签：

```json
{
  "title": "...",
  "source": "@karpathy",
  "link": "https://...",
  "classification": {
    "temporality": "timeless",
    "domain": "AI",
    "depth": "deep",
    "relevance": "high",
    "novelty": "novel",
    "knowhow_connections": [
      {
        "file": "know-how/AI/agent/agent-design-principles.md",
        "relation": "contradicts",
        "label": "⚠️"
      }
    ]
  },
  "decision": "highlight",
  "sedimentation_suggestion": {
    "level": "know-how",
    "target": "know-how/AI/agent/",
    "reason": "反直觉发现，与已有认知矛盾"
  }
}
```

---

## 6 维度分类框架

### 1. 时效性 (Temporality)

| 值 | 定义 | 示例 |
|----|------|------|
| **timeless** | 无时效限制，任何时候读都有价值 | 设计原则、深度分析、方法论 |
| **timely** | 有时效但短期内有价值 | 新模型发布分析、行业趋势 |
| **ephemeral** | 快速过期 | 产品更新公告、日常新闻 |

**判断方法**：这条内容 6 个月后还值得读吗？
- 是 → timeless
- 可能 → timely
- 不 → ephemeral

### 2. 领域 (Domain)

匹配 Ivan 的 know-how 目录结构：

| 值 | 对应 know-how |
|----|-------------|
| AI | `know-how/AI/` |
| product | `know-how/product/` |
| business | `know-how/business/` |
| engineering | `know-how/engineering/` |
| life | `know-how/gua/`、`know-how/learning/` |
| other | 不匹配现有目录 |

### 3. 深度 (Depth)

| 值 | 定义 | 信号 |
|----|------|------|
| **deep** | 原创分析、独立思考、有论证 | 长文、数据支撑、反直觉 |
| **surface** | 信息传递、简单评论 | 短文、转述、无新观点 |
| **announcement** | 纯公告/发布 | 产品发布、版本更新 |

### 4. 关联性 (Relevance)

与 Ivan 当前关注点的匹配度：

| 值 | 判断依据 |
|----|---------|
| **high** | 直接命中 Ivan 的核心关注领域（AI Agent、产品设计、Amazon） |
| **medium** | 间接相关或边缘领域 |
| **low** | 几乎不相关 |

### 5. 新颖度 (Novelty)

相对于 Ivan know-how 的增量：

| 值 | 标注 | 说明 |
|----|------|------|
| **novel** | 🆕 | Ivan 和 curator 都不知道的新发现 |
| **reinforces** | ✅ | 验证了 Ivan 已有认知 |
| **contradicts** | ⚠️ | 与 Ivan 已有认知矛盾 |
| **updates** | 🔄 | 更新了 Ivan 已知信息的状态 |

### 6. Know-how 连接 (Know-how Connections)

```
1. 从内容中提取关键概念/关键词
2. Grep know-how/ 查找匹配文件
3. 读取匹配文件的核心观点
4. 判断关系：reinforces / contradicts / updates / novel
5. 记录 file path + relation + label
```

---

## 过滤决策矩阵

| 条件组合 | 决策 | 说明 |
|----------|------|------|
| timeless + deep + high relevance | **highlight** | 必推，放 Highlights 区 |
| timeless + deep + medium relevance | **recommend** | 推荐阅读 |
| timely + deep + high relevance | **recommend** | 推荐但标注时效 |
| 任何 + contradicts know-how | **highlight** | 必推（⚠️ 矛盾信号） |
| 高 attention 源 + non-ephemeral | **recommend** | 高信任源默认推 |
| ephemeral + surface | **discard** | 丢弃 |
| low relevance + surface | **discard** | 丢弃 |
| announcement（非重大） | **discard** | 丢弃 |

过滤率目标：丢弃 60-80% 的采集内容。

---

## 沉淀建议规则

| 条件 | 建议层级 | 目标位置 |
|------|---------|---------|
| timeless + deep + contradicts/novel | know-how | `know-how/{domain}/` |
| timeless + deep + reinforces（多源验证） | know-how（升级） | 更新已有 know-how 文件 |
| timely + deep + novel | memory | `memory/0_Inbox/` |
| 多源同时讨论同一话题 | memory + signal | `memory/0_Inbox/` + Signals 区 |

---

## 关联

- **rss-feed-monitor** — 提供博客采集数据
- **x-timeline-monitor** — 提供 X 采集数据
- **content-curator** — 调用此 Skill 执行分类和过滤

---

*v1.0 | 2026-02-12*
