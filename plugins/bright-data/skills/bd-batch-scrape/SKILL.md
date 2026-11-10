---
name: bd-batch-scrape
description: "Process up to 10 URLs or search queries in a single parallel call. Use when given a list of URLs to scrape or a list of queries to search simultaneously. Significantly faster than running them one by one."
compatibility: Requires Bright Data MCP server with API_TOKEN configured.
allowed-tools: mcp(Bright Data:scrape_batch), mcp(Bright Data:search_engine_batch)
metadata:
  author: Bright Data
---

# 批量抓取 / 批量搜索

并行处理：$ARGUMENTS

## 选择正确的批处理工具

### 批量 URL 抓取（最多 10 个 URL）

当用户提供要获取的 URL 列表时使用：

```
scrape_batch({
  urls: [
    "https://example.com/page1",
    "https://example.com/page2",
    "https://example.com/page3"
  ]
})
```

以 Markdown 格式返回由 `{ url, content }` 对组成的数组。网络解锁器会自动处理每个 URL 的反爬保护。

### 批量搜索（最多 10 个查询）

当用户提供搜索查询列表时使用：

```
search_engine_batch({
  queries: [
    { query: "topic one", engine: "google" },
    { query: "topic two", engine: "google" },
    { query: "topic three", engine: "bing" }
  ]
})
```

为每个查询返回 JSON（Google）或 Markdown（Bing/Yandex）。

## 限制

- 每次 `scrape_batch` 调用最多处理 **10 个 URL**
- 每次 `search_engine_batch` 调用最多处理 **10 个查询**
- 如果用户提供的数量超过 10 个，请拆分为多个批量调用并按顺序处理

## 展示结果

对于批量 URL 抓取：
- 使用来源 URL 清晰标注每项结果
- 应用与 `bd-scrape` 相同的内容保真规则：保留事实，仅移除无关内容
- 包含“来源”部分，列出所有已处理的 URL

对于批量搜索：
- 如果所有查询涉及共同主题，请综合整理各查询的发现
- 使用原始来源 URL 引用每项事实
- 在末尾包含汇总的“来源”部分

## 如果 MCP 服务器未连接

如果批处理工具不可用，**立即停止**。不要自行处理 URL 或查询。告知用户：

1. Bright Data MCP 服务器尚未连接
2. 运行 `/bd-setup` 以配置并验证连接
3. 然后重试批处理请求