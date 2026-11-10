---
name: bd-scrape
description: "Fetch and extract content from any URL, bypassing bot protection, CAPTCHAs, and geo-blocks using Bright Data's Web Unlocker. Returns clean Markdown by default. For multiple URLs use scrape_batch. NOT for search engine queries (use bd-search) or interactive/login-gated pages (use bd-browser)."
compatibility: Requires Bright Data MCP server with API_TOKEN and WEB_UNLOCKER_ZONE configured.
allowed-tools: mcp(Bright Data:scrape_as_markdown), mcp(Bright Data:scrape_as_html), mcp(Bright Data:scrape_batch)
metadata:
  author: Bright Data
---

# URL 内容提取

从以下位置提取内容：$ARGUMENTS

## 步骤 1：选择正确的工具

**单个 URL：整洁易读的输出（默认）：**
```
scrape_as_markdown({ url: "<target url>" })
```

**单个 URL：需要原始 HTML（DOM 解析、链接提取）：**
```
scrape_as_html({ url: "<target url>" })
```

**多个 URL（最多 10 个）：批处理：**
```
scrape_batch({
  urls: ["<url1>", "<url2>", "<url3>"]
})
```

网络解锁器会自动处理动态代理、验证码破解和 JavaScript 渲染。你无需配置其中任何一项。

## 步骤 2：返回内容

按以下格式展示提取的内容：

**[页面标题](URL)**

然后按照以下规则展示内容：
- 忠实保留来源内容：不要改写或加入主观评论
- 完整提取所有编号列表项和项目符号列表项
- 仅移除明显的无关内容：导航菜单、Cookie 横幅、页脚、广告
- 保留所有事实、名称、数字、日期和引文
- 对于你引用的任何事实，请使用 `[Page Title](url)` 在行内引用来源 URL

对于批量结果，按顺序展示每个 URL 的内容，并使用其 URL 进行标注。

## 步骤 3：如果需要后续操作

如果页面内容不够充分，请考虑：
- 使用 `bd-search` 查找更多来源
- 如果页面需要登录或用户交互才能显示内容，请使用 `bd-browser`

## 如果 MCP 服务器未连接

如果工具不可用，**立即停止**。不要自行获取 URL，也不要使用内置获取工具。告知用户：

1. Bright Data MCP 服务器尚未连接
2. 运行 `/bd-setup` 以配置并验证连接
3. 然后重试其请求