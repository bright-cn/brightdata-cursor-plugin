---
name: bd-setup
description: Set up the Bright Data plugin — configure API credentials and verify MCP connection
---

# Bright Data 插件设置

## 步骤 1：检查 MCP 服务器连接

尝试调用 `session_stats` MCP 工具，验证服务器是否可访问：

```
session_stats()
```

如果成功，则 MCP 服务器已连接。跳至**步骤 3：验证区域**。

## 步骤 2：配置凭据

如果 `session_stats` 失败，则 MCP 服务器尚未连接。指导用户完成配置：

### 选项 A：托管服务器（推荐，无需安装 Node.js）

将以下内容添加到 Cursor 的 MCP 设置中（`~/.cursor/mcp.json`，或通过 Cursor 设置 → MCP）：

```json
{
  "mcpServers": {
    "Bright Data": {
      "url": "https://mcp.brightdata.com/mcp?token=YOUR_API_TOKEN_HERE"
    }
  }
}
```

将 `YOUR_API_TOKEN_HERE` 替换为从以下位置获取的 API 令牌：**https://www.bright.cn/cp/setting/users**

### 选项 B：本地服务器（可完全控制区域和 Pro 模式）

需要 Node.js 18 或更高版本。添加到 Cursor 的 MCP 设置：

```json
{
  "mcpServers": {
    "Bright Data": {
      "command": "npx",
      "args": ["@brightdata/mcp"],
      "env": {
        "API_TOKEN": "YOUR_API_TOKEN_HERE",
        "WEB_UNLOCKER_ZONE": "mcp_unlocker",
        "BROWSER_ZONE": "mcp_browser"
      }
    }
  }
}
```

**要启用 Pro 工具**（结构化数据、浏览器自动化），请添加：
```json
"PRO_MODE": "true"
```

或者仅启用特定组：
```json
"GROUPS": "browser,ecommerce,social"
```

保存后，重新加载 Cursor 或重启 MCP 服务器。

## 步骤 3：验证区域

连接后，再次调用 `session_stats` 检查可用内容并报告：

- MCP 服务器版本和连接状态
- 此会话的工具使用次数
- Pro 工具（`web_data_*`、`scraping_browser_*`）是否可访问

测试核心工具：
1. 使用简单查询调用 `search_engine`，验证搜索引擎 API / 网络解锁器区域
2. 如果用户拥有 Pro 模式：确认 `scraping_browser_navigate` 可用

## 步骤 4：报告状态

告知用户：
- 哪些工具可用（免费层级与 Pro）
- 其区域名称（Unlocker 区域、Browser 区域）
- 缺少哪些配置，以及具体需要添加什么

如果一切正常，请确认：**Bright Data MCP 已连接并准备就绪。**