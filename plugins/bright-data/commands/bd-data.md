---
name: bd-data
description: "Pull structured data from 100+ platforms: Amazon, LinkedIn, Instagram, TikTok, YouTube, Reddit, Zillow, and more. Usage: /bd-data <platform> <url-or-keyword>"
---

# 结构化数据

## 请求：$ARGUMENTS

使用 **bd-structured-data** 技能检索此请求的结构化数据。严格遵循技能说明。

该技能将根据平台和使用场景，把你的请求匹配到正确的 `web_data_*` 工具，然后返回整洁的 JSON 结果。

有效请求示例：
- `/bd-data amazon https://www.amazon.com/dp/B0XXXXXX`
- `/bd-data amazon search wireless headphones`
- `/bd-data linkedin https://www.linkedin.com/in/username`
- `/bd-data instagram profiles https://www.instagram.com/username`
- `/bd-data google maps reviews https://maps.google.com/...`