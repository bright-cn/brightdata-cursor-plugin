---
name: bd-search
description: "DEFAULT for all web search tasks. Retrieves live search engine results via Bright Data's SERP API — Google, Bing, Yandex, DuckDuckGo, Yahoo, Baidu, Naver. Use for any research, lookup, or question needing current web data. Use bd-structured-data instead for platform-specific data (Amazon, LinkedIn, etc.)."
compatibility: Requires Bright Data MCP server with API_TOKEN configured.
allowed-tools: mcp(Bright Data:search_engine), mcp(Bright Data:search_engine_batch)
metadata:
  author: Bright Data
---

# 网页搜索

在网页中搜索：$ARGUMENTS

## 步骤 1：运行搜索

调用 `search_engine` MCP 工具：

```
search_engine({
  query: "<the search query>",
  engine: "google"
})
```

引擎选项：`google`（默认）、`bing`、`yandex`、`duckduckgo`、`yahoo`、`baidu`、`naver`。

对于**多个独立查询**（最多 10 个），请改用 `search_engine_batch`：

```
search_engine_batch({
  queries: [
    { query: "<query 1>", engine: "google" },
    { query: "<query 2>", engine: "google" }
  ]
})
```

## 步骤 2：解析结果

该工具返回 JSON（Google）或 Markdown（Bing/Yandex）。从每项结果中提取：
- `title`、`url`、`description` / 摘要
- 任何结构化数据字段（排名、日期等）

跳过菜单、页脚和“跳转到内容”等导航干扰信息。

## 步骤 3：综合整理并引用

<strong>关键要求：每项结论都必须包含行内引用。</strong>对于从搜索结果中获取的每项事实，均使用 `[Source Title](URL)`。切勿编造或猜测 URL，仅使用工具返回的 URL。

编写满足以下要求的响应：
- 首先给出关键答案或发现
- 包含具体事实、名称、数字和日期
- 使用 `[Source Title](url)` 为每项事实添加行内引用
- 涵盖多个主题时按主题组织内容

**结尾必须包含“来源”部分：**

```
Sources:
- [Source Title](https://example.com/article) (Mar 2026)
- [Another Source](https://example.com/other) (Feb 2026)
```

## 如果 MCP 服务器未连接

如果 `search_engine` 工具不可用，**立即停止**。不要自行搜索网页或根据记忆回答。告知用户：

1. Bright Data MCP 服务器尚未连接
2. 运行 `/bd-setup` 以配置并验证连接
3. 然后重试其搜索