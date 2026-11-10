---
name: bd-structured-data
description: "Pull pre-structured, cleaned data from 100+ platforms including Amazon, LinkedIn, Instagram, TikTok, YouTube, Reddit, Zillow, Crunchbase, and more. Returns clean JSON — no parsing required. Requires PRO_MODE=true or GROUPS config. Always prefer this over bd-browser for supported platforms — it is faster and cheaper."
compatibility: Requires Bright Data MCP server with API_TOKEN configured and PRO_MODE=true or relevant GROUPS enabled.
allowed-tools: mcp(Bright Data:web_data_amazon_product), mcp(Bright Data:web_data_amazon_product_search), mcp(Bright Data:web_data_amazon_product_reviews), mcp(Bright Data:web_data_walmart_product), mcp(Bright Data:web_data_walmart_seller), mcp(Bright Data:web_data_ebay_product), mcp(Bright Data:web_data_homedepot_products), mcp(Bright Data:web_data_zara_products), mcp(Bright Data:web_data_etsy_products), mcp(Bright Data:web_data_bestbuy_products), mcp(Bright Data:web_data_linkedin_person_profile), mcp(Bright Data:web_data_linkedin_company_profile), mcp(Bright Data:web_data_linkedin_job_listings), mcp(Bright Data:web_data_linkedin_posts), mcp(Bright Data:web_data_linkedin_people_search), mcp(Bright Data:web_data_instagram_profiles), mcp(Bright Data:web_data_instagram_posts), mcp(Bright Data:web_data_instagram_reels), mcp(Bright Data:web_data_instagram_comments), mcp(Bright Data:web_data_facebook_posts), mcp(Bright Data:web_data_facebook_marketplace_listings), mcp(Bright Data:web_data_facebook_company_reviews), mcp(Bright Data:web_data_facebook_events), mcp(Bright Data:web_data_tiktok_profiles), mcp(Bright Data:web_data_tiktok_posts), mcp(Bright Data:web_data_tiktok_shop), mcp(Bright Data:web_data_tiktok_comments), mcp(Bright Data:web_data_x_posts), mcp(Bright Data:web_data_youtube_profiles), mcp(Bright Data:web_data_youtube_videos), mcp(Bright Data:web_data_youtube_comments), mcp(Bright Data:web_data_reddit_posts), mcp(Bright Data:web_data_google_maps_reviews), mcp(Bright Data:web_data_google_shopping), mcp(Bright Data:web_data_google_play_store), mcp(Bright Data:web_data_apple_app_store), mcp(Bright Data:web_data_zillow_properties_listing), mcp(Bright Data:web_data_booking_hotel_listings), mcp(Bright Data:web_data_crunchbase_company), mcp(Bright Data:web_data_zoominfo_company_profile), mcp(Bright Data:web_data_reuter_news), mcp(Bright Data:web_data_yahoo_finance_business), mcp(Bright Data:web_data_github_repository_file)
metadata:
  author: Bright Data
---

# 结构化数据提取

检索以下内容的结构化数据：$ARGUMENTS

## 步骤 1：确定正确的工具

使用下表将用户请求与对应平台和工具进行匹配：

### 电商
| 平台 | 使用场景 | 工具 |
|----------|----------|------|
| Amazon | 商品详情（使用 /dp/ URL） | `web_data_amazon_product` |
| Amazon | 按关键词搜索结果 | `web_data_amazon_product_search` |
| Amazon | 商品评论 | `web_data_amazon_product_reviews` |
| Walmart | 商品详情（使用 /ip/ URL） | `web_data_walmart_product` |
| Walmart | 卖家资料 | `web_data_walmart_seller` |
| eBay | 商品信息 | `web_data_ebay_product` |
| Best Buy | 电子产品目录 | `web_data_bestbuy_products` |
| Home Depot | 家装产品 | `web_data_homedepot_products` |
| Etsy | 市场商品信息 | `web_data_etsy_products` |
| Zara | 时尚商品库存 | `web_data_zara_products` |
| TikTok Shop | 电商商品 | `web_data_tiktok_shop` |

### 专业与商业
| 平台 | 使用场景 | 工具 |
|----------|----------|------|
| LinkedIn | 个人资料 | `web_data_linkedin_person_profile` |
| LinkedIn | 公司页面 | `web_data_linkedin_company_profile` |
| LinkedIn | 职位列表 | `web_data_linkedin_job_listings` |
| LinkedIn | 帖子与讨论 | `web_data_linkedin_posts` |
| LinkedIn | 人员搜索 | `web_data_linkedin_people_search` |
| Crunchbase | 初创公司 / 公司信息 | `web_data_crunchbase_company` |
| ZoomInfo | 商业情报 | `web_data_zoominfo_company_profile` |
| Yahoo Finance | 公司财务数据 | `web_data_yahoo_finance_business` |

### 社交媒体与内容
| 平台 | 使用场景 | 工具 |
|----------|----------|------|
| Instagram | 账户资料 | `web_data_instagram_profiles` |
| Instagram | 帖子 | `web_data_instagram_posts` |
| Instagram | Reels | `web_data_instagram_reels` |
| Instagram | 评论 | `web_data_instagram_comments` |
| TikTok | 创作者资料 | `web_data_tiktok_profiles` |
| TikTok | 帖子元数据 | `web_data_tiktok_posts` |
| TikTok | 评论 | `web_data_tiktok_comments` |
| Facebook | 帖子 | `web_data_facebook_posts` |
| Facebook | 市场商品信息 | `web_data_facebook_marketplace_listings` |
| Facebook | 公司评论 | `web_data_facebook_company_reviews` |
| Facebook | 活动 | `web_data_facebook_events` |
| X / Twitter | 帖子 | `web_data_x_posts` |
| YouTube | 频道资料 | `web_data_youtube_profiles` |
| YouTube | 视频元数据 | `web_data_youtube_videos` |
| YouTube | 评论（默认：10 条） | `web_data_youtube_comments` |
| Reddit | 帖子 | `web_data_reddit_posts` |

### 搜索、地图与服务
| 平台 | 使用场景 | 工具 |
|----------|----------|------|
| Google Shopping | 商品搜索 | `web_data_google_shopping` |
| Google Maps | 地点评论 | `web_data_google_maps_reviews` |
| Google Play | Android 应用详情 | `web_data_google_play_store` |
| Apple App Store | iOS 应用详情 | `web_data_apple_app_store` |
| Zillow | 房地产信息 | `web_data_zillow_properties_listing` |
| Booking.com | 酒店信息 | `web_data_booking_hotel_listings` |
| Reuters | 新闻文章 | `web_data_reuter_news` |
| GitHub | 仓库文件内容 | `web_data_github_repository_file` |

## 步骤 2：调用工具

根据特定工具的要求传入平台 URL 或搜索关键词。示例：

```
web_data_amazon_product({ url: "https://www.amazon.com/dp/B0XXXXXX" })
web_data_linkedin_person_profile({ url: "https://www.linkedin.com/in/username" })
web_data_amazon_product_search({ keyword: "wireless headphones" })
```

## 步骤 3：展示结果

向用户返回结构化 JSON 数据。清晰总结关键字段，并在需要时询问用户是否希望深入了解特定属性。

## 如果 PRO 工具不可用

如果 `web_data_*` 工具不可用，则账户可能需要启用 PRO_MODE。告知用户：

1. 结构化数据工具需要 PRO_MODE 或 GROUPS 配置
2. 将 `"PRO_MODE": "true"` 添加到 `mcp.json` 中的 `env` 部分，或设置 `"GROUPS": "ecommerce,social,business"` 以启用选择性访问
3. 运行 `/bd-setup` 以验证更新后的配置