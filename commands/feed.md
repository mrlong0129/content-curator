# Feed Command

管理内容策展订阅源。

---

## 使用场景

- 查看当前订阅源列表和评分
- 新增/移除订阅源
- 调整源的 attention 评分
- 查看各源的产出统计

---

## 输入

```
/feed list                         # 列出所有订阅源（按 category 分组）
/feed add [url]                    # 新增订阅源（自动检测类型和 RSS）
/feed remove [id]                  # 移除订阅源
/feed score [id] [score]           # 调整 Ivan attention 评分
/feed stats                        # 各源产出统计（文章数、insight 比例）
```

示例：
- `/feed list` — 概览所有源
- `/feed add https://x.com/newaccount` — 新增 X 账号
- `/feed add https://newblog.com` — 新增博客（自动发现 RSS）
- `/feed score x-op7418 2.5` — 降低 @op7418 的评分
- `/feed stats` — 看每个源的产出效率

---

## 执行步骤

### /feed list

1. 读取 `feed-registry.yaml`
2. 按 category 分组展示：
   - 名称/handle
   - attention_ivan / attention_curator
   - signal_type
   - tags
3. 标注哪些源 Ivan 还没评分

### /feed add [url]

1. 判断类型（X 账号 or 博客）
2. X → 提取 handle，添加到 x-pro 或 x-cn
3. 博客 → WebFetch 发现 RSS URL
4. 写入 feed-registry.yaml
5. 初始 attention 分数为 null（curator 首次采集后评分）

### /feed score [id] [score]

1. 在 feed-registry.yaml 中找到对应 feed
2. 更新 attention_ivan 为指定值
3. 记录变更到 execution-log.md

### /feed stats

1. 读取 feed-state.json
2. 统计每个源：总文章数、最近活跃度、insight 产出比例
3. 展示排名

---

## 输出

直接在对话中展示，不生成文件。

---

*v1.0 | 2026-02-12*
