---
name: bd-status
description: Check Bright Data MCP connection status and view tool usage for this session
---

# Bright Data 状态

## 步骤 1：获取会话统计信息

调用 `session_stats` MCP 工具：

```
session_stats()
```

## 步骤 2：向用户报告

清晰展示结果：

- <strong>连接：</strong>MCP 服务器已连接 / 未连接
- <strong>此会话的工具使用情况：</strong>列出每个工具及其调用次数
- **可用的免费层级工具：**`search_engine`、`scrape_as_markdown`、`scrape_as_html`、`scrape_batch`、`search_engine_batch`、`extract`
- <strong>Pro 工具状态：</strong>报告 `scraping_browser_*` 和 `web_data_*` 工具是否可访问

## 如果 session_stats 失败

MCP 服务器尚未连接。告知用户运行 `/bd-setup` 以配置凭据并重新连接。