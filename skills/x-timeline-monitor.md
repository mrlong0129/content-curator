# X Timeline Monitor

> X/Twitter 账号内容采集：浏览器自动化提取推文、结构化存储。

---

## 适用场景

- 定期采集关注的 X 账号最近推文
- 追踪某个话题在 X 上的讨论热度
- 识别高互动内容（潜在高价值信号）

---

## 输入

| 参数 | 必填 | 示例 |
|------|------|------|
| feed-registry.yaml | 是 | X 账号列表及评分 |
| 采集范围 | 否 | `--category x-pro` / `--handle @karpathy` |
| 时间范围 | 否 | `--since 2026-02-10` |

---

## 输出

```
outputs/data/content-curator/YYYY-MM/raw/x/
├── {handle}_{date}.json       # 每个账号的推文
└── ...
```

每条推文结构：
```json
{
  "handle": "@karpathy",
  "text": "推文内容...",
  "datetime": "2026-02-12T08:30:00Z",
  "link": "https://x.com/karpathy/status/...",
  "metrics": {
    "reply": 45,
    "retweet": 230,
    "like": 1200
  },
  "is_retweet": false,
  "is_thread": true,
  "has_media": false
}
```

---

## 核心流程

### 1. 采集方式

**方式 A: Chrome MCP（主要）**

使用 `mcp__claude-in-chrome__navigate` 导航到 X 账号页面，然后用 `mcp__claude-in-chrome__javascript_tool` 执行提取脚本：

```javascript
(() => {
  const tweets = [];
  const articles = document.querySelectorAll('article[data-testid="tweet"]');

  articles.forEach((article, i) => {
    const textEl = article.querySelector('[data-testid="tweetText"]');
    const text = textEl?.textContent?.trim() || '';

    const timeEl = article.querySelector('time');
    const datetime = timeEl?.getAttribute('datetime') || '';

    const linkEl = article.querySelector('a[href*="/status/"]');
    const link = linkEl ? `https://x.com${linkEl.getAttribute('href')}` : '';

    const metrics = {};
    ['reply', 'retweet', 'like'].forEach(type => {
      const el = article.querySelector(`[data-testid="${type}"]`);
      metrics[type] = parseInt(el?.textContent) || 0;
    });

    if (text.length > 0) {
      tweets.push({
        index: i + 1,
        text: text.slice(0, 2000),
        datetime,
        link,
        metrics,
        is_retweet: !!article.querySelector('[data-testid="socialContext"]'),
        is_thread: text.length > 500
      });
    }
  });

  return {
    account: window.location.pathname.replace('/', ''),
    collected_at: new Date().toISOString(),
    count: tweets.length,
    tweets
  };
})()
```

**方式 B: Nitter RSS（回退）**

部分 Nitter 实例暴露 RSS：
- `https://nitter.net/{handle}/rss`
- 可用时优先使用（轻量、无需浏览器）

### 2. 采集策略

| 策略 | 说明 |
|------|------|
| **优先级排序** | attention >= 4.0 的账号先处理 |
| **限速** | 每个账号间隔 5-10 秒 |
| **批次** | 每批 3-5 个账号 |
| **滚动** | 默认不滚动（首屏推文即可），高 attention 可滚动 1-2 次 |
| **登录状态** | 依赖用户已登录的 Chrome 会话 |

### 3. 内容过滤（采集阶段）

采集时即可做初步过滤：

| 条件 | 动作 |
|------|------|
| 纯转推（无评论） | 标记 is_retweet，保留但降低优先级 |
| 互动极低（<10 likes） | 低 attention 源可跳过 |
| 纯图片/视频无文字 | 跳过 |
| Thread 首条 | 标记 is_thread，后续可深度采集 |

### 4. Thread 深度采集

当检测到高价值 Thread（高 attention 源 + 高互动）：

```
1. 点击进入 Thread 页面
2. 提取所有 Thread 内的推文
3. 拼接为完整文本
4. 存储为单条 thread entry
```

---

## 优先级排序

| 优先级 | 条件 | 处理 |
|--------|------|------|
| P0 | attention_ivan >= 4.0（@karpathy, @ilyasut, @dotey, @foxshuo） | 必查，可滚动 |
| P1 | attention_ivan >= 3.5 | 必查，首屏 |
| P2 | attention_ivan < 3.5 或未评分 | 隔次查 |

---

## 常见问题

| 问题 | 处理 |
|------|------|
| X 未登录 | 提醒 Ivan 在 Chrome 中登录 X |
| 页面加载慢 | 等待 3-5 秒后再执行脚本 |
| 反爬检测 | 降低频率，增加间隔 |
| 推文被删/不可见 | 跳过，标记 status: "unavailable" |
| DOM 结构变更 | 更新提取脚本，记录到 reflections.md |

---

## 关联

- **content-classification** — 采集后的推文送分类
- **content-curator** — 调用此 Skill 执行 X 采集
- **external-channel-data-acquisition** — 共享浏览器采集模式

---

*v1.0 | 2026-02-12*
