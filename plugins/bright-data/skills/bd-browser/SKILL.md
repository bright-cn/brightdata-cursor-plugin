---
name: bd-browser
description: "Real browser automation for JS-heavy, interactive, or login-gated pages. Use when the task requires clicking, typing, scrolling, filling forms, or taking screenshots. Supports a Puppeteer/Playwright-style flow via MCP tools. Requires Browser zone (PRO tier). Use bd-scrape for static pages that do not need interaction."
compatibility: Requires Bright Data MCP server with API_TOKEN and BROWSER_ZONE configured (PRO tier).
allowed-tools: mcp(Bright Data:scraping_browser_navigate), mcp(Bright Data:scraping_browser_snapshot), mcp(Bright Data:scraping_browser_click_ref), mcp(Bright Data:scraping_browser_type_ref), mcp(Bright Data:scraping_browser_get_text), mcp(Bright Data:scraping_browser_get_html), mcp(Bright Data:scraping_browser_screenshot), mcp(Bright Data:scraping_browser_scroll), mcp(Bright Data:scraping_browser_scroll_to_ref), mcp(Bright Data:scraping_browser_wait_for_ref), mcp(Bright Data:scraping_browser_go_back), mcp(Bright Data:scraping_browser_go_forward), mcp(Bright Data:scraping_browser_network_requests)
metadata:
  author: Bright Data
---

# 浏览器自动化

为以下任务自动执行浏览器交互：$ARGUMENTS

## 会话限制

- <strong>空闲超时：</strong>5 分钟，持续执行交互以避免会话过期
- <strong>最长持续时间：</strong>每个会话 30 分钟
- 会话在空闲超时或达到最长持续时间时会自动终止

## 步骤 1：导航到页面

```
scraping_browser_navigate({ url: "<target url>" })
```

## 步骤 2：检查页面结构

拍摄 ARIA 快照，以识别交互元素及其引用：

```
scraping_browser_snapshot()
```

快照会列出所有按钮、输入框、链接及其他元素及其 `ref` 标识符。在所有后续交互中使用这些引用。

## 步骤 3：按需交互

**点击按钮或链接：**
```
scraping_browser_click_ref({ ref: "<element ref>" })
```

**在字段中输入内容（可选择按 Enter 提交）：**
```
scraping_browser_type_ref({ ref: "<input ref>", text: "<value>", pressEnter: true })
```

**滚动到页面底部：**
```
scraping_browser_scroll()
```

**将特定元素滚动到可见区域：**
```
scraping_browser_scroll_to_ref({ ref: "<element ref>" })
```

**等待元素出现：**
```
scraping_browser_wait_for_ref({ ref: "<element ref>", timeout: 5000 })
```

**浏览历史记录：**
```
scraping_browser_go_back()
scraping_browser_go_forward()
```

## 步骤 4：提取内容

**获取所有可见文本：**
```
scraping_browser_get_text()
```

**获取页面 HTML（除非必要，否则避免使用 full_page：其内容量很大）：**
```
scraping_browser_get_html({ full_page: false })
```

**拍摄截图以进行视觉验证：**
```
scraping_browser_screenshot()
```

**检查页面发出的网络请求：**
```
scraping_browser_network_requests()
```

## 推荐流程

```
navigate → snapshot → [interact: click / type / scroll] → snapshot again → extract
```

每次交互后重新拍摄快照，以便在继续操作前查看更新后的页面状态。

## 如果 PRO 工具不可用

如果 `scraping_browser_navigate` 不可用，则 Browser 区域可能尚未配置，或账户可能处于免费层级。告知用户：

1. 浏览器自动化需要 Bright Data Browser 区域（Pro 层级）
2. 将 `BROWSER_ZONE` 添加到 `mcp.json` 环境变量，并确保 `PRO_MODE=true` 或 `GROUPS=browser`
3. 运行 `/bd-setup` 以验证连接