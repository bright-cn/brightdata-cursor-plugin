---
name: bd-code
description: "Use when the user wants to WRITE CODE that integrates Bright Data APIs — Python or Node.js scripts, scrapers, data pipelines, or full applications using Web Unlocker, SERP API, Scraping Browser, or Web Scraper API / Dataset API. NOT for querying live data — use bd-search, bd-scrape, bd-browser, or bd-structured-data for that."
compatibility: No runtime dependencies required — generates code for the user to run.
allowed-tools: mcp(Bright Data:session_stats)
metadata:
  author: Bright Data
---

# Bright Data 代码生成

为以下内容生成集成代码：$ARGUMENTS

## 何时触发此技能

当用户提出以下类型的请求时触发此技能：
- “编写爬虫工具”“构建脚本”“集成 Bright Data”“代码示例”
- “如何在 Python / Node.js 中使用[产品]”
- “帮我构建”“创建项目”“搭建项目框架”
- 提及 `requests`、`playwright`、`puppeteer`、`selenium`、`brightdata-sdk`

当用户只是要求搜索或抓取实时数据时，**不要**使用此技能：请改用 `bd-search`、`bd-scrape`、`bd-browser` 或 `bd-structured-data`。

---

## 步骤 1：确定用户想要构建的内容

```
What does the user want to build?
│
├── Fetch a URL / bypass bot protection → Section 1 (Web Unlocker)
├── Search engine results at scale     → Section 2 (SERP API)
├── Interact with a page (click/type)  → Section 3 (Scraping Browser)
├── Pull structured platform data      → Section 4 (Web Scraper API / Dataset API)
└── Full project from scratch          → Section 8 (Scaffold) + relevant sections above
```

阅读用户请求，选择匹配的部分，并输出完整且可运行的代码。

---

## 第 1 部分：网络解锁器 API

### 1A. Python：`requests`（最简单）

```python
import requests

API_TOKEN = "YOUR_API_TOKEN"   # from brightdata.com/cp/setting/users
ZONE_NAME = "web_unlocker1"    # your Web Unlocker zone name

def unlock(url: str, country: str = None, data_format: str = "markdown") -> str:
    payload = {
        "zone": ZONE_NAME,
        "url": url,
        "format": "raw",
        "data_format": data_format,   # "html" | "markdown" | "screenshot"
    }
    if country:
        payload["country"] = country  # ISO 3166-1 e.g. "us", "gb", "de"

    response = requests.post(
        "https://api.brightdata.com/request",
        headers={
            "Content-Type": "application/json",
            "Authorization": f"Bearer {API_TOKEN}",
        },
        json=payload,
        timeout=30,
    )
    response.raise_for_status()
    return response.json()["body"]

if __name__ == "__main__":
    content = unlock("https://example.com", country="us")
    print(content)
```

### 1B. Python：官方 SDK（`brightdata-sdk`）

```bash
pip install brightdata-sdk
```

```python
import asyncio
from brightdata import BrightDataClient

async def main():
    async with BrightDataClient() as client:
        result = await client.scrape_url("https://example.com")
        print(result.data)

asyncio.run(main())
```

**批处理（并发）：**
```python
import asyncio
from brightdata import BrightDataClient

async def main():
    urls = [
        "https://example.com/page1",
        "https://example.com/page2",
        "https://example.com/page3",
    ]
    async with BrightDataClient() as client:
        tasks = [client.scrape_url(url) for url in urls]
        results = await asyncio.gather(*tasks)
        for r in results:
            print(r.data)

asyncio.run(main())
```

**异步模式（非阻塞，等待约 2 分钟）：**
```python
async with BrightDataClient() as client:
    result = await client.scrape_url(
        url="https://example.com",
        mode="async",
        poll_interval=5,
        poll_timeout=180,
    )
    print(result.data)
```

### 1C. Node.js：原生 `fetch`

```javascript
const API_TOKEN = process.env.BRIGHT_DATA_API_TOKEN;
const ZONE_NAME = "web_unlocker1";

async function unlock(url, { country, dataFormat = "markdown" } = {}) {
  const payload = { zone: ZONE_NAME, url, format: "raw", data_format: dataFormat };
  if (country) payload.country = country;

  const res = await fetch("https://api.brightdata.com/request", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${API_TOKEN}`,
    },
    body: JSON.stringify(payload),
  });

  if (!res.ok) {
    const text = await res.text();
    throw new Error(`Bright Data error ${res.status}: ${text}`);
  }

  const data = await res.json();
  return data.body;
}

unlock("https://example.com", { country: "us" })
  .then(console.log)
  .catch(console.error);
```

### 1D. 通过代理（替代方案：无需调用 REST API）

```python
import requests
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

ACCOUNT_ID  = "YOUR_ACCOUNT_ID"   # brd-customer-XXXXXXX
ZONE_NAME   = "web_unlocker1"
ZONE_PASS   = "YOUR_ZONE_PASSWORD"

proxy_url = f"http://brd-customer-{ACCOUNT_ID}-zone-{ZONE_NAME}:{ZONE_PASS}@brd.superproxy.io:33335"
proxies = {"http": proxy_url, "https": proxy_url}

response = requests.get("https://example.com", proxies=proxies, verify=False)
print(response.text)
```

### 1E. 响应结构

```json
{
  "status_code": 200,
  "headers": { "content-type": "text/html; charset=utf-8", "..." : "..." },
  "body": "<html>...</html>"
}
```

**常见错误代码：**
| 代码 | 含义 |
|------|---------|
| 200 | 成功 |
| 400 | 错误请求（载荷格式错误、缺少区域/URL） |
| 401 | 身份验证失败：检查 API 令牌 |
| 403 | 区域未获授权 |
| 429 | 超出速率限制 |
| 500 | 重试后仍无法访问目标网站 |

---

## 第 2 部分：搜索引擎 API

### 2A. Python：直接调用 API

```python
import requests

API_TOKEN = "YOUR_API_TOKEN"
ZONE_NAME = "serp_zone1"   # your SERP API zone name

def search_google(query: str, country: str = "us", lang: str = "en",
                  num: int = 10, parsed: bool = True) -> dict:
    """
    Returns parsed JSON results when parsed=True,
    or raw HTML string when parsed=False.
    """
    params = f"q={requests.utils.quote(query)}&gl={country}&hl={lang}&num={num}"
    if parsed:
        params += "&brd_json=1"

    response = requests.post(
        "https://api.brightdata.com/request",
        headers={
            "Content-Type": "application/json",
            "Authorization": f"Bearer {API_TOKEN}",
        },
        json={
            "zone": ZONE_NAME,
            "url": f"https://www.google.com/search?{params}",
            "format": "raw",
        },
        timeout=30,
    )
    response.raise_for_status()
    return response.json()

if __name__ == "__main__":
    results = search_google("bright data mcp", country="us")
    for r in results.get("organic", []):
        print(r["position"], r["title"], r["url"])
```

### 2B. Python：官方 SDK

```python
import asyncio
from brightdata import BrightDataClient

async def main():
    async with BrightDataClient() as client:
        result = await client.search.google(
            query="bright data web scraping",
            num_results=10,
        )
        for item in result.data:
            print(item["title"], item["link"])

asyncio.run(main())
```

### 2C. Node.js：直接调用 API

```javascript
const API_TOKEN = process.env.BRIGHT_DATA_API_TOKEN;
const ZONE_NAME  = "serp_zone1";

async function searchGoogle(query, { country = "us", lang = "en", num = 10, parsed = true } = {}) {
  const params = new URLSearchParams({ q: query, gl: country, hl: lang, num });
  if (parsed) params.set("brd_json", "1");

  const res = await fetch("https://api.brightdata.com/request", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${API_TOKEN}`,
    },
    body: JSON.stringify({
      zone: ZONE_NAME,
      url: `https://www.google.com/search?${params}`,
      format: "raw",
    }),
  });
  if (!res.ok) throw new Error(`SERP API error ${res.status}: ${await res.text()}`);
  return res.json();
}

searchGoogle("mcp servers 2025", { country: "us" })
  .then((data) => (data.organic || []).forEach((r) => console.log(r.position, r.title, r.url)))
  .catch(console.error);
```

### 2D. 通过代理（替代方案）

```python
import requests, urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxy_url = "http://brd-customer-ACCOUNT_ID-zone-ZONE_NAME:ZONE_PASS@brd.superproxy.io:33335"
proxies = {"http": proxy_url, "https": proxy_url}

response = requests.get(
    "https://www.google.com/search?q=bright+data&brd_json=1",
    proxies=proxies, verify=False,
)
print(response.json())
```

### 2E. 所有支持的搜索引擎和 URL 格式

| 引擎 | URL 格式 |
|--------|-----------|
| Google | `https://www.google.com/search?q={query}&brd_json=1` |
| Bing | `https://www.bing.com/search?q={query}` |
| Yandex | `https://yandex.com/search/?text={query}` |
| DuckDuckGo | `https://duckduckgo.com/?q={query}` |
| Yahoo | `https://search.yahoo.com/search?p={query}` |
| Baidu | `https://www.baidu.com/s?wd={query}` |
| Naver | `https://search.naver.com/search.naver?query={query}` |

### 2F. 所有 `brd_*` URL 参数

| 参数 | 值 | 说明 |
|-----------|--------|-------------|
| `brd_json=1` | `1`, `html` | `1` → 解析后的 JSON；`html` → JSON + 嵌套的原始 HTML |
| `gl` | ISO 国家/地区代码 | 按地理位置定位结果（例如 `gl=us`、`gl=de`） |
| `hl` | 语言代码 | 界面语言（例如 `hl=en`、`hl=ja`） |
| `num` | 整数 | 每页结果数（默认 10，最多 100） |
| `start` | 整数 | 分页偏移量（0、10、20……） |
| `tbm` | `shop`, `nws`, `isch`, `vid` | 搜索类型（购物、新闻、图片、视频） |
| `brd_mobile` | `1`, `ios`, `android` | 设备模拟 |
| `brd_browser` | `chrome`, `safari`, `firefox` | 浏览器指纹 |

### 2G. 解析后的 JSON 响应结构

```json
{
  "engine": "google",
  "query": "bright data mcp",
  "organic": [
    {
      "position": 1,
      "title": "Bright Data MCP Server",
      "url": "https://github.com/bright-cn/brightdata-mcp",
      "description": "..."
    }
  ],
  "ads": [],
  "shopping": [],
  "related_searches": []
}
```

---

## 第 3 部分：抓取浏览器

### 3A. 前提条件

```bash
# Puppeteer (Node.js)
npm install puppeteer-core

# Playwright (Node.js)
npm install playwright-core

# Playwright (Python)
pip install playwright
playwright install chromium

# Selenium (Node.js)
npm install selenium-webdriver

# Selenium (Python)
pip install selenium
```

### 3B. 凭据格式

```
brd-customer-<ACCOUNT_ID>-zone-<ZONE_NAME>:<ZONE_PASSWORD>
```

从以下位置获取凭据：`brightdata.com/cp/zones` → 选择你的 Browser 区域 → Overview 选项卡。

**国家/地区定位**：将 `-country-XX` 附加到用户名：
```
brd-customer-C1234567-zone-my_browser-country-us:mypassword
```

### 3C. Puppeteer（Node.js）：标准示例

```javascript
const puppeteer = require("puppeteer-core");

const SBR_WS = "wss://brd-customer-ACCOUNT_ID-zone-ZONE_NAME:ZONE_PASS@brd.superproxy.io:9222";

async function main() {
  const browser = await puppeteer.connect({ browserWSEndpoint: SBR_WS });
  try {
    const page = await browser.newPage();
    await page.goto("https://example.com", { waitUntil: "domcontentloaded" });
    const html = await page.content();
    console.log(html);
  } finally {
    await browser.close();
  }
}

main().catch((err) => console.error(err.stack || err));
```

### 3D. Playwright：Node.js

```javascript
const { chromium } = require("playwright-core");

const SBR_WS = "wss://brd-customer-ACCOUNT_ID-zone-ZONE_NAME:ZONE_PASS@brd.superproxy.io:9222";

async function main() {
  const browser = await chromium.connectOverCDP(SBR_WS);
  try {
    const context = browser.contexts()[0];
    const page = context.pages()[0];
    await page.goto("https://example.com", { waitUntil: "domcontentloaded" });
    console.log(await page.content());
  } finally {
    await browser.close();
  }
}

main().catch(console.error);
```

### 3E. Playwright：Python

```python
import asyncio
from playwright.async_api import async_playwright

SBR_WS = "wss://brd-customer-ACCOUNT_ID-zone-ZONE_NAME:ZONE_PASS@brd.superproxy.io:9222"

async def main():
    async with async_playwright() as pw:
        browser = await pw.chromium.connect_over_cdp(SBR_WS)
        try:
            page = await browser.new_page()
            await page.goto("https://example.com", wait_until="domcontentloaded")
            print(await page.content())
        finally:
            await browser.close()

asyncio.run(main())
```

### 3F. Selenium：Node.js

```javascript
const { Builder, Browser } = require("selenium-webdriver");

const SBR_WD = "https://brd-customer-ACCOUNT_ID-zone-ZONE_NAME:ZONE_PASS@brd.superproxy.io:9515";

async function main() {
  const driver = await new Builder()
    .forBrowser(Browser.CHROME)
    .usingServer(SBR_WD)
    .build();
  try {
    await driver.get("https://example.com");
    console.log(await driver.getPageSource());
  } finally {
    await driver.quit();
  }
}

main().catch(console.error);
```

### 3G. Selenium：Python

```python
from selenium import webdriver
from selenium.webdriver.chromium.remote_connection import ChromiumRemoteConnection

SBR_WD = "https://brd-customer-ACCOUNT_ID-zone-ZONE_NAME:ZONE_PASS@brd.superproxy.io:9515"

def main():
    conn = ChromiumRemoteConnection(SBR_WD, "goog", "chrome")
    driver = webdriver.Remote(conn, options=webdriver.ChromeOptions())
    try:
        driver.get("https://example.com")
        print(driver.page_source)
    finally:
        driver.quit()

main()
```

### 3H. 会话限制和最佳实践

| 限制 | 值 |
|-------|-------|
| 空闲超时 | 5 分钟 |
| 最长会话持续时间 | 30 分钟 |
| SSL 验证 | CDP WSS 连接不需要 |

**最佳实践：**
- 始终使用 `try/finally` 关闭浏览器：避免产生空闲超时费用
- 使用 `waitUntil: "domcontentloaded"` 而不是 `"networkidle"`：速度更快，足以满足大多数页面需求
- 对于 Playwright，`browser.contexts()[0]` 用于访问现有上下文（不要新建上下文）
- 国家/地区定位通过用户名字符串完成，而不是通过代理标头

---

## 第 4 部分：网页爬虫工具 API / 数据集 API

### 4A. 官方 Python SDK（推荐用于平台数据）

```bash
pip install brightdata-sdk
```

```python
import asyncio
from brightdata import BrightDataClient

async def main():
    async with BrightDataClient() as client:
        # Amazon product
        result = await client.scrape.amazon.products(
            url="https://www.amazon.com/dp/B0CRMZHDG8"
        )
        print(result.data)

        # Amazon reviews
        result = await client.scrape.amazon.reviews(
            url="https://www.amazon.com/dp/B0CRMZHDG8"
        )

        # LinkedIn
        result = await client.scrape.linkedin.profiles(
            url="https://www.linkedin.com/in/username"
        )

        # Instagram
        result = await client.scrape.instagram.profiles(
            url="https://www.instagram.com/username"
        )

asyncio.run(main())
```

**平台方法映射：**

| 平台 | SDK 调用 |
|----------|----------|
| Amazon 商品 | `client.scrape.amazon.products(url=...)` |
| Amazon 评论 | `client.scrape.amazon.reviews(url=...)` |
| Amazon 卖家 | `client.scrape.amazon.sellers(url=...)` |
| LinkedIn 个人资料 | `client.scrape.linkedin.profiles(url=...)` |
| LinkedIn 公司 | `client.scrape.linkedin.companies(url=...)` |
| LinkedIn 职位 | `client.scrape.linkedin.jobs(url=...)` |
| LinkedIn 帖子 | `client.scrape.linkedin.posts(url=...)` |
| Instagram 个人资料 | `client.scrape.instagram.profiles(url=...)` |
| Instagram 帖子 | `client.scrape.instagram.posts(url=...)` |
| Instagram Reels | `client.scrape.instagram.reels(url=...)` |
| Facebook 帖子 | `client.scrape.facebook.posts(url=...)` |

### 4B. 原始数据集 API：异步轮询模式

当 Python SDK 不支持你的平台，或者你需要使用 Node.js 时，请使用此方式。

**端点：**
```
POST https://api.brightdata.com/datasets/v3/trigger?dataset_id=<ID>&include_errors=true
GET  https://api.brightdata.com/datasets/v3/progress/<snapshot_id>
GET  https://api.brightdata.com/datasets/v3/snapshot/<snapshot_id>?format=json
```

**Python：**
```python
import requests, time

API_TOKEN   = "YOUR_API_TOKEN"
DATASET_ID  = "gd_YOUR_DATASET_ID"   # get from brightdata.com/cp/datasets

HEADERS = {
    "Authorization": f"Bearer {API_TOKEN}",
    "Content-Type": "application/json",
}

def trigger(urls: list[str]) -> str:
    body = [{"url": u} for u in urls]
    res = requests.post(
        f"https://api.brightdata.com/datasets/v3/trigger?dataset_id={DATASET_ID}&include_errors=true",
        headers=HEADERS, json=body, timeout=30,
    )
    res.raise_for_status()
    return res.json()["snapshot_id"]

def poll(snapshot_id: str, timeout: int = 600, interval: int = 5) -> None:
    url = f"https://api.brightdata.com/datasets/v3/progress/{snapshot_id}"
    deadline = time.time() + timeout
    while time.time() < deadline:
        res = requests.get(url, headers=HEADERS, timeout=10)
        res.raise_for_status()
        status = res.json().get("status")
        if status == "ready":
            return
        if status == "failed":
            raise RuntimeError(f"Collection failed: {snapshot_id}")
        time.sleep(interval)
    raise TimeoutError(f"Timed out after {timeout}s")

def download(snapshot_id: str) -> list:
    res = requests.get(
        f"https://api.brightdata.com/datasets/v3/snapshot/{snapshot_id}?format=json",
        headers=HEADERS, timeout=60,
    )
    res.raise_for_status()
    return res.json()

def collect(urls: list[str]) -> list:
    snap = trigger(urls)
    print(f"Triggered snapshot: {snap}")
    poll(snap)
    results = download(snap)
    print(f"Got {len(results)} records")
    return results

if __name__ == "__main__":
    data = collect(["https://www.amazon.com/dp/B0XXXXXX"])
    print(data[0])
```

**Node.js：**
```javascript
const API_TOKEN  = process.env.BRIGHT_DATA_API_TOKEN;
const DATASET_ID = "gd_YOUR_DATASET_ID";

const HEADERS = {
  Authorization: `Bearer ${API_TOKEN}`,
  "Content-Type": "application/json",
};

async function trigger(urls) {
  const res = await fetch(
    `https://api.brightdata.com/datasets/v3/trigger?dataset_id=${DATASET_ID}&include_errors=true`,
    { method: "POST", headers: HEADERS, body: JSON.stringify(urls.map((url) => ({ url }))) }
  );
  if (!res.ok) throw new Error(`Trigger failed: ${res.status}`);
  return (await res.json()).snapshot_id;
}

async function poll(snapshotId, timeoutMs = 600_000, intervalMs = 5_000) {
  const deadline = Date.now() + timeoutMs;
  while (Date.now() < deadline) {
    const res = await fetch(
      `https://api.brightdata.com/datasets/v3/progress/${snapshotId}`,
      { headers: HEADERS }
    );
    if (!res.ok) throw new Error(`Poll failed: ${res.status}`);
    const { status } = await res.json();
    if (status === "ready") return;
    if (status === "failed") throw new Error(`Collection failed: ${snapshotId}`);
    await new Promise((r) => setTimeout(r, intervalMs));
  }
  throw new Error(`Timed out waiting for ${snapshotId}`);
}

async function download(snapshotId) {
  const res = await fetch(
    `https://api.brightdata.com/datasets/v3/snapshot/${snapshotId}?format=json`,
    { headers: HEADERS }
  );
  if (!res.ok) throw new Error(`Download failed: ${res.status}`);
  return res.json();
}

async function collect(urls) {
  const snap = await trigger(urls);
  console.log(`Triggered snapshot: ${snap}`);
  await poll(snap);
  const results = await download(snap);
  console.log(`Got ${results.length} records`);
  return results;
}

collect(["https://www.amazon.com/dp/B0XXXXXX"])
  .then((data) => console.log(data[0]))
  .catch(console.error);
```

### 4C. SDK 数据集 API（预构建数据集，不基于 URL）

```python
import asyncio
from brightdata import BrightDataClient
from brightdata.datasets import export

async def main():
    async with BrightDataClient() as client:
        # Sample 5 records from a pre-built dataset
        snapshot_id = await client.datasets.imdb_movies.sample(records_limit=5)
        data = await client.datasets.imdb_movies.download(snapshot_id)
        export(data, "results.json")

asyncio.run(main())
```

---

## 第 5 部分：身份验证参考

### 何时使用哪种凭据类型

| 凭据类型 | 格式 | 用途 |
|-----------------|--------|----------|
| API 令牌（Bearer） | `Bearer eyJ...` | REST API（`/request`、数据集 API、MCP `API_TOKEN` 环境变量） |
| 区域凭据 | `brd-customer-ID-zone-NAME:PASS` | 直接代理连接（Puppeteer、Playwright、Selenium、代理路由） |

### 环境变量最佳实践

**Python `.env` / `os.environ`：**
```python
import os
from dotenv import load_dotenv  # pip install python-dotenv

load_dotenv()

API_TOKEN   = os.getenv("BRIGHT_DATA_API_TOKEN")
ZONE_NAME   = os.getenv("BRIGHT_DATA_UNLOCKER_ZONE", "web_unlocker1")
ACCOUNT_ID  = os.getenv("BRIGHT_DATA_ACCOUNT_ID")
ZONE_PASS   = os.getenv("BRIGHT_DATA_ZONE_PASS")
```

**Node.js：**
```javascript
import "dotenv/config";  // npm install dotenv

const API_TOKEN  = process.env.BRIGHT_DATA_API_TOKEN;
const ZONE_NAME  = process.env.BRIGHT_DATA_UNLOCKER_ZONE ?? "web_unlocker1";
const ACCOUNT_ID = process.env.BRIGHT_DATA_ACCOUNT_ID;
const ZONE_PASS  = process.env.BRIGHT_DATA_ZONE_PASS;
```

**`.env` 文件：**
```
BRIGHT_DATA_API_TOKEN=your_api_token_here
BRIGHT_DATA_UNLOCKER_ZONE=web_unlocker1
BRIGHT_DATA_SERP_ZONE=serp_zone1
BRIGHT_DATA_BROWSER_ZONE=mcp_browser
BRIGHT_DATA_ACCOUNT_ID=brd-customer-XXXXXXX
BRIGHT_DATA_ZONE_PASS=your_zone_password
```

---

## 第 6 部分：在代码中集成 MCP

适用于构建通过 MCP 使用 Bright Data 的 AI 智能体或工具的开发者：

### Cursor/Claude Desktop 配置（`mcp.json`）

**托管方式（最简单）：**
```json
{
  "mcpServers": {
    "Bright Data": {
      "url": "https://mcp.brightdata.com/mcp?token=YOUR_API_TOKEN"
    }
  }
}
```

**包含所有选项的本地方式：**
```json
{
  "mcpServers": {
    "Bright Data": {
      "command": "npx",
      "args": ["@brightdata/mcp"],
      "env": {
        "API_TOKEN": "YOUR_API_TOKEN",
        "WEB_UNLOCKER_ZONE": "mcp_unlocker",
        "BROWSER_ZONE": "mcp_browser",
        "PRO_MODE": "true",
        "POLLING_TIMEOUT": "600",
        "RATE_LIMIT": "100/1h"
      }
    }
  }
}
```

**选择性分组（比完整 PRO_MODE 更经济）：**
```json
"GROUPS": "browser,ecommerce,social"
```

有效的组值：`browser`、`ecommerce`、`social`、`business`、`finance`、`research`、`travel`、`app_stores`

---

## 第 7 部分：常见模式和误区

### 误区 1：mcp.json 中的环境变量名称错误

MCP 服务器使用 `API_TOKEN`（而不是 `BRIGHT_DATA_API_TOKEN`）。较长的名称是本地 `.env` 约定。

mcp.json 中的正确写法：
```json
{ "API_TOKEN": "..." }
```

mcp.json 中的错误写法：
```json
{ "BRIGHT_DATA_API_TOKEN": "..." }
```

### 误区 2：将 `requests` 用于 SERP 代理，出现 SSL 警告

使用代理模式时，始终应禁止或处理 SSL 警告：
```python
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
response = requests.get(url, proxies=proxies, verify=False)
```

### 误区 3：Playwright 上下文与新上下文

通过 CDP 连接抓取浏览器时，浏览器已经有一个上下文。请使用它，不要新建上下文：
```python
# CORRECT
context = browser.contexts[0]
page = context.pages[0]

# WRONG (creates extra context, wastes session time)
context = await browser.new_context()
```

### 误区 4：数据集 API 轮询超时

MCP 中的默认轮询时间为 600 秒。如果编写原始轮询代码，请与此设置保持一致或将其设为可配置：
```python
POLLING_TIMEOUT = int(os.getenv("POLLING_TIMEOUT", "600"))
poll(snapshot_id, timeout=POLLING_TIMEOUT)
```

### 误区 5：`brd_json=1` 与 `brd_json=html`

- `brd_json=1` → 仅返回解析后的 JSON（自然搜索结果、广告等）：适用于大多数情况
- `brd_json=html` → 相同的 JSON + 嵌套的原始 HTML：数据量大得多，仅在同时需要 HTML 时使用

### 误区 6：抓取浏览器，会话空闲超时

如果代码执行 `page.goto()` 后等待很长时间才进行下一项操作，会话可能会过期。始终确保交互间隔不超过 5 分钟。对于长时间等待，请使用 `page.waitForSelector()`，而不是 `setTimeout`。

---

## 第 8 部分：最小化入门项目

### 项目 A：简单 URL 解锁器（Python）

```
my-scraper/
├── .env
├── requirements.txt   # requests python-dotenv
└── scraper.py         # uses Section 1A pattern
```

### 项目 B：SERP 监控器（Node.js）

```
serp-monitor/
├── .env
├── package.json       # { "type": "module" }
└── index.js           # uses Section 2C pattern, saves to JSON
```

### 项目 C：Puppeteer 爬虫工具（Node.js）

```
browser-scraper/
├── .env
├── package.json
├── node_modules/      # puppeteer-core
└── index.js           # uses Section 3C pattern
```

### 项目 D：完整流程（Python、SDK）

```
data-pipeline/
├── .env
├── requirements.txt   # brightdata-sdk python-dotenv
└── pipeline.py        # SDK pattern from Section 4A
```

---

## 输出说明

1. 根据用户请求识别产品（网络解锁器 / SERP / 抓取浏览器 / 数据集 API）
2. 识别语言（Python 或 Node.js）：如果未指定，请询问或默认使用 Python
3. 输出上方匹配部分中的完整可运行代码块：不得截断
4. 在代码下方添加第 5 部分中相关的 `.env` 条目
5. 添加第 7 部分中任何适用的误区说明
6. 如果用户要求完整项目，请输出第 8 部分中的项目框架并填入实际代码

始终输出完整代码：切勿概述或省略代码行。
