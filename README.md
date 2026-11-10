# Cursor 插件模板

通过单个仓库构建并发布 Cursor Marketplace 插件。

包含三个插件：

- **starter-simple**：仅包含规则和技能
- **starter-advanced**：包含规则、技能、智能体、命令、钩子、MCP 和脚本
- **bright-data**：由 Bright Data 提供支持的网页搜索、抓取、浏览器自动化和结构化数据

## Bright Data 插件

`plugins/bright-data/` 插件将 [Bright Data MCP 服务器](https://github.com/bright-cn/brightdata-mcp)集成到 Cursor 中。它涵盖四款产品：

| 产品 | 技能 | 命令 |
|---------|-------|---------|
| 搜索引擎 API（Google、Bing 等） | `bd-search` | `/bd-search` |
| 网络解锁器 | `bd-scrape`, `bd-batch-scrape` | `/bd-scrape` |
| 抓取浏览器 | `bd-browser` | `/bd-browser` |
| 网页爬虫工具 / 数据集 API | `bd-structured-data` | `/bd-data` |
| 代码生成（Python 和 Node.js） | `bd-code` | `/bd-code` |

**技能**：AI 用来查询实时数据或生成代码：
- `bd-search`：通过搜索引擎 API 进行实时网页搜索
- `bd-scrape`：通过网络解锁器提取单个 URL
- `bd-batch-scrape`：并行提取多个 URL
- `bd-browser`：通过抓取浏览器（CDP）实现交互式浏览器自动化
- `bd-structured-data`：结构化平台数据（Amazon、LinkedIn、Instagram 等）
- `bd-code`：为任意 Bright Data 产品生成完整的 Python 或 Node.js 集成代码

**命令**：调用技能的斜杠命令：
- `/bd-setup`：配置并验证 MCP 连接
- `/bd-search <query>`：搜索网页
- `/bd-scrape <url>`：提取 URL
- `/bd-browser <url>`：打开页面并与之交互
- `/bd-data <platform> <url>`：获取结构化平台数据
- `/bd-status`：检查 MCP 连接状态
- `/bd-code <product> <language> [use-case]`：生成集成代码

### 快速设置

1. 从 [www.bright.cn/cp/setting/users](https://www.bright.cn/cp/setting/users) 获取你的 API 令牌。
2. 将 MCP 服务器添加到 Cursor 的 `mcp.json`：

```json
{
  "mcpServers": {
    "Bright Data": {
      "command": "npx",
      "args": ["@brightdata/mcp"],
      "env": {
        "API_TOKEN": "YOUR_API_TOKEN"
      }
    }
  }
}
```

3. 在 Cursor 中运行 `/bd-setup` 以验证连接。

