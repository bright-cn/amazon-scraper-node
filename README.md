[![使用 Amazon 爬虫工具 API 抓取 Amazon 数据：商品、评论、卖家和搜索结果。按 ASIN、关键词和 UPC 采集或发现数据。免费开始使用。](.github/banner.png)](https://www.bright.cn/products/web-scraper/amazon?utm_source=github&utm_medium=referral&utm_content=amazon-scraper-node)

# 适用于 Node.js 和 JavaScript 的 Amazon 抓取工具

使用 Node.js 和 JavaScript 将 Amazon 商品、评论、卖家和搜索结果抓取为 JSON。无需 Product Advertising API 密钥，无需登录。基于 [Bright Data Amazon 爬虫工具 API](https://www.bright.cn/products/web-scraper/amazon?utm_source=github&utm_medium=referral&utm_content=amazon-scraper-node) 构建。新账户每月可获得 5,000 个免费额度。

[![实时检查](https://github.com/bright-cn/amazon-scraper-node/actions/workflows/live.yml/badge.svg)](https://github.com/bright-cn/amazon-scraper-node/actions/workflows/live.yml) [![最后验证](https://img.shields.io/badge/last%20verified-11%20Nov%202026-brightgreen)](https://github.com/bright-cn/amazon-scraper-node/actions/workflows/live.yml) <!-- verified: rewritten by the weekly run -->

[快速开始](#如何使用-nodejs-抓取-amazon-商品数据) · [无需 PA-API](#无需-product-advertising-api-即可抓取-amazon) · [CLI](#批量-amazon-爬虫-cli) · [端点](#抓取-amazon-商品评论卖家和搜索结果) · [数据](#amazon-商品数据字段json) · [错误](#amazon-爬虫错误) · [MCP 和 Claude Code](#amazon-mcp-服务器和-claude-code-技能) · [免费额度](#这个-amazon-爬虫工具免费吗) · [文档](https://docs.brightdata.com/products/scrapers/amazon/introduction) · [支持](#支持)

使用 JavaScript 将 Amazon 商品、评论、卖家和搜索结果获取为 JSON。无需登录，无需浏览器。基于 [Bright Data Amazon 爬虫工具 API](https://www.bright.cn/products/web-scraper/amazon?utm_source=github&utm_medium=referral&utm_content=amazon-scraper-node) 构建。

使用 [Bright Data JavaScript SDK](https://github.com/bright-cn/sdk-js)。完整 API 文档：[Amazon 爬虫工具 API](https://docs.brightdata.com/products/scrapers/amazon/introduction)。

此外还提供：用于商品抓取的一条命令 CLI，以及完全不需要 JavaScript 的 [Bright Data CLI](#amazon-mcp-服务器和-claude-code-技能)。

## 如何使用 Node.js 抓取 Amazon 商品数据

需要 Node 20 或更高版本。此软件包使用 ESM，因此请使用 `import`，不要使用 `require`。

```bash
npm install @brightdata/sdk
export BRIGHTDATA_API_TOKEN=YOUR_API_KEY
```

从 [Bright Data 控制面板](https://www.bright.cn/cp/setting/users)获取令牌。此 SDK 不会读取 `.env` 文件；Node 可通过 `node --env-file=.env yourscript.mjs` 为其加载该文件。

你也可以不用令牌。运行一次 `npx -p @brightdata/cli bdata login`：它会打开浏览器，此后 SDK 会自动找到已存储的凭据，供你以及在该终端中工作的任何编码智能体使用。智能体无法点击完成登录流程，因此请先自行登录。

还没有账户？[创建账户](https://www.bright.cn/cp/start)；新账户[每月可获得 5,000 个免费额度](https://docs.brightdata.com/general/account/billing-and-pricing/free-tier)。

```javascript
import { bdclient } from "@brightdata/sdk";

const client = new bdclient({ autoCreateZones: false });
const job = await client.scrape.amazon.collectProducts(
  ["https://www.amazon.com/dp/B0CRMZHDG8"],
  { async: true, includeErrors: true },
);
const result = await job.toResult({ pollTimeout: 600_000 });
if (!result.success) throw new Error(`${result.status}: ${result.error}`);
const [product] = result.data;
console.log(product.title);
console.log(product.final_price, product.currency, "|", product.rating, "stars |", product.reviews_count, "reviews");
await client.close();
```

```
STANLEY Quencher H2.0 Flow State Tumbler, 40 oz, Fuchsia
39.95 USD | 4.7 stars | 205036 reviews
```

每个商品消耗一个[额度](https://www.bright.cn/pricing/web-scraper)，API 预计每个输入约需 7 秒。

该代码片段中的三项设置都不能省略。

`autoCreateZones: false` 会阻止 SDK 在启动时创建区域。这些区域用于网络解锁器和 SERP，它们是此爬虫工具完全不会使用的另外两款 Bright Data 产品。在未设置付款方式的账户中，创建区域会失败。

`includeErrors: true` 会让 API 将失效的 ASIN 作为一行报告。没有它，该行会被丢弃，你将得不到任何结果。

`async: true` 会让 `collectProducts` 返回一个任务，而不是通过会在一分钟后放弃的同步端点处理。

`pollTimeout` 的单位是毫秒，而不是秒。从 Python 示例复制的数值会在第一次状态检查前就超时。

SDK 的商品筛选器接受 `url`、`zipcode` 和 `language`。API 还接受 `asin`、`origin_url` 和 `all_variations`，但筛选器会拒绝这三项，因此请像上面那样将 ASIN 作为 `/dp/` URL 传入。

## 无需 Product Advertising API 即可抓取 Amazon

Amazon 爬虫工具 API 不需要 Amazon 账户、卖家账户或 Product Advertising API（PA-API）访问密钥。唯一需要的凭据是 Bright Data API 密钥，可将其设置为 `BRIGHTDATA_API_TOKEN` 或通过 `bdata login` 存储。

## 批量 Amazon 爬虫 CLI

此代码仓库中的命令可同时对多个 ASIN 运行 Node.js 快速开始示例，并写入一个 JSON 文件。

```bash
npm install -g github:bright-cn/amazon-scraper-node
amazon-scraper B0CRMZHDG8 B085DVHQ57
```

```
Fetching 2 Amazon products: B0CRMZHDG8, B085DVHQ57
One job for all of them. One credit per product.
asking  2 products...
got     B0CRMZHDG8: 99 fields (STANLEY Quencher H2.0 Flow State Tumbler, 40 oz,)
got     B085DVHQ57: 100 fields (Owala FreeSip Stainless Steel Water Bottle 32 oz)

Saved 2 of 2 products as JSON to amazon.json
```

它既可接受商品 URL，也可接受其中的 ASIN，因此 `amazon-scraper https://www.amazon.com/dp/B0CRMZHDG8` 的效果相同。相同 ASIN 出现两次时只会抓取一次，因此重复项不会产生费用。

在终端中，`asking` 行会被以下内容原地替换，让你可以看到程序正在运行以及已经运行了多久：

```
⠹ 2 products 0:00:11
```

```
--out PATH   output file, default amazon.json
```

你请求的每个 ASIN 都会放入同一个任务。API 按记录而非按任务计费，因此十个商品的等待时间成本与一个商品相同。

也可以导入它而不是直接运行，从而为每个 ASIN 获取 `ok` 和 `error`，而不是原始行。`scrape` 不会因为某个错误的 ASIN 而拒绝整个请求；读取 `product` 前请先检查 `ok`：

```javascript
import { scrape } from "@brightdata/amazon-scraper-node";

for (const outcome of await scrape(["B0CRMZHDG8", "B0ZZZZZZZZ"])) {
  if (outcome.ok) {
    console.log(`${outcome.asin}: ${outcome.product.title}`);
  } else {
    console.log(`${outcome.asin} failed: ${outcome.error}`);
  }
}
```

```
B0CRMZHDG8: STANLEY Quencher H2.0 Flow State Tumbler, 40 oz, Fuchsia
B0ZZZZZZZZ failed: The navigation resulted in a dead page (404 status code)
```

## 抓取 Amazon 商品、评论、卖家和搜索结果

该命令覆盖下表第一行。此处的每个代码片段都完整无缺，只需要 `@brightdata/sdk`：直接按原样粘贴即可。所有代码片段都会在每周一通过 Actions 运行，每隔一天还会运行一次较小规模的检查，顶部徽章显示最新结果。

| 你已有 | 你想要 | 调用 |
| --- | --- | --- |
| ASIN 或商品 URL | 该商品 | `collectProducts([url], { async: true, includeErrors: true })` |
| 关键词 | 匹配的商品 | `discoverProductsByKeyword([{ keyword }], { async: true, includeErrors: true, limitPerInput: 3 })` |
| 分类 URL | 其中的商品 | `discoverProductsByCategoryURL([{ url }], …)` |
| 畅销商品 URL | 其中的商品 | `discoverProductsByBestSellerURL([{ category_url }], …)` |
| UPC | 匹配的商品 | `discoverProductsByUPC([{ upc }], …)` |
| 卖家 URL | 该卖家 | `collectSellers([url], { async: true, includeErrors: true })` |
| 关键词和商店 URL | 搜索页面 | `collectProductSearch([{ keyword, url }], …)` |
| 商品 URL | 其评论 | `collectReviews([url], …)`，每条评论消耗一个额度，详情见下文 |

它们都挂载在 `client.scrape.amazon` 下。

### 抓取 Amazon 评论，以及为什么这里不运行该调用

评论按每条评论一个额度计费，而且无法通过此 SDK 有效地请求更少评论。

API 接受 `max_reviews` 输入，Bright Data 自己的文档示例将其设为 20。SDK 的评论筛选器仅声明 `url` 和 `reviews_to_not_include`，并拒绝 `max_reviews`。SDK 自己的上限 `limitPerInput` 会在评论数据集上返回零行；我们对两个商品测量了三次，结果均是如此，但它在商品数据集上表现正常（[sdk-js#37](https://github.com/brightdata/sdk-js/issues/37)）。

因此，唯一能返回内容的评论调用是不设上限的调用。此 README 的[快速开始](#如何使用-nodejs-抓取-amazon-商品数据)中使用的商品显示有 205,036 条评论。这只是一次调用。

请先检查商品上的 `reviews_count`；在 SDK 支持 `max_reviews` 前，如果需要有界的数据切片，请直接调用 REST 端点。

### 上限，以及会静默移除上限的标志

`limitPerInput` 用于限制发现调用的数量。它仅在同时传入 `async: true` 时有效：如果没有该键，选项会匹配 SDK 选项 schema 的同步调用部分，而该部分会丢弃不认识的键，因此上限会在不报错的情况下消失（[sdk-js#36](https://github.com/brightdata/sdk-js/issues/36)）。SDK 的 `DiscoverOptions` 类型会提示你省略 `async`，所以类型正确的调用反而是不设上限的调用。

此处的每个发现代码片段都会同时传入两者。

### 按关键词抓取 Amazon 商品

```javascript
import { bdclient } from "@brightdata/sdk";

const client = new bdclient({ autoCreateZones: false });
const job = await client.scrape.amazon.discoverProductsByKeyword(
  [{ keyword: "stainless steel water bottle" }],
  { async: true, includeErrors: true, limitPerInput: 3 },
);
const result = await job.toResult({ pollTimeout: 900_000 });
if (!result.success) throw new Error(`${result.status}: ${result.error}`);
for (const product of result.data) {
  console.log(product.asin, "|", product.final_price, product.currency, "|", product.title.slice(0, 40));
}
await client.close();
```

```
B0D2W1MKZX | 14.24 USD | Fijinhom Insulated Water Bottle with Han
B085DVHQ57 | 29.99 USD | Owala FreeSip Stainless Steel Water Bott
B0D8J2ZB8P | 14.24 USD | POWCAN 26 oz Insulated Water Bottle with
```

如果没有 `limitPerInput`，该调用会返回所有匹配项，每项按一个额度计费。

### 在一个任务中抓取多个 Amazon ASIN

URL 数组只会创建一个任务，而不是每个商品一个任务。

```javascript
import { bdclient } from "@brightdata/sdk";

const client = new bdclient({ autoCreateZones: false });
const job = await client.scrape.amazon.collectProducts(
  ["https://www.amazon.com/dp/B0CRMZHDG8", "https://www.amazon.com/dp/B085DVHQ57"],
  { async: true, includeErrors: true },
);
const result = await job.toResult({ pollTimeout: 600_000 });
if (!result.success) throw new Error(`${result.status}: ${result.error}`);
for (const product of result.data) {
  console.log(product.asin, "|", product.brand, "|", product.final_price, product.currency);
}
await client.close();
```

```
B085DVHQ57 | Owala | 29.99 USD
B0CRMZHDG8 | STANLEY | 39.95 USD
```

返回的行没有固定顺序。请使用每行自身的 `asin` 将其与你请求的内容匹配，不要按位置匹配。

### 立即触发，稍后获取

对于超过几个商品的任务，不要让进程阻塞一小时。先触发任务，保存快照 ID，待任务就绪后再获取。快照可在 30 天内下载。

```javascript
import { bdclient } from "@brightdata/sdk";

const client = new bdclient({ autoCreateZones: false });
const job = await client.scrape.amazon.collectProducts(
  ["https://www.amazon.com/dp/B0CRMZHDG8"],
  { async: true, includeErrors: true },
);
console.log("snapshot:", job.snapshotId);
await job.wait({ pollInterval: 5_000, pollTimeout: 600_000 });
console.log("status:", await job.status());
const [record] = await job.fetch();
console.log("fetched:", record.asin, "|", record.title.slice(0, 40));
await client.close();
```

```
snapshot: sd_mu9ycsw7zyq04t1uw
status: ready
fetched: B0CRMZHDG8 | STANLEY Quencher H2.0 Flow State Tumbler
```

## Amazon 商品数据字段（JSON）

大多数人需要的商品字段：

```
asin  title  brand  final_price  currency  rating  reviews_count  availability
```

代码没有硬编码任何字段列表。API 返回的所有内容都会进入 `result.data` 以及该命令生成的文件。

API 在 schema 中使用 `pii: true` 将其中 3 个字段标记为个人数据：`seller_name`、`zipcode` 和 `coupon`。

<!-- fields:start -->
<details>
<summary>全部 122 个字段及其类型和描述</summary>

每周通过 `client.datasets.amazonProducts.getMetadata()` 根据数据集 schema 重新生成，因此不会过时。商品只会携带适用于它的字段：示例文件包含这 122 个字段中的 97 个，另加 schema 中未列出的 `timestamp` 和 `input`。

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `title` | 文本 | 商品标题 |
| `seller_name` | 文本 | 个人数据。卖家名称 |
| `brand` | 文本 | 商品品牌 |
| `description` | 文本 | 商品的简短描述 |
| `initial_price` | 价格 | 初始价格 |
| `currency` | 文本 | 商品货币 |
| `availability` | 文本 | 商品库存状态 |
| `reviews_count` | 数字 | 评论数量 |
| `categories` | 数组 | 商品分类 |
| `parent_asin` | 文本 | 商品的父 ASIN |
| `asin` | 文本 | 每个商品的唯一标识符 |
| `buybox_seller` | 文本 | 购物车中的卖家 |
| `number_of_sellers` | 数字 | 商品卖家数量 |
| `root_bs_rank` | 数字 | 大类中的畅销商品排名 |
| `ISBN10` | 文本 | 图书的 ISBN-10 标识符 |
| `answered_questions` | 数字 | 已回答问题的数量 |
| `domain` | URL | 商品域名的 URL |
| `images_count` | 数字 | 图片数量 |
| `url` | URL | 直接链接到商品的 URL |
| `video_count` | 数字 | 视频数量 |
| `image_url` | URL | 直接链接到商品图片的 URL |
| `item_weight` | 文本 | 商品重量 |
| `rating` | 数字 | 商品评分 |
| `product_dimensions` | 文本 | 商品尺寸 |
| `seller_id` | 文本 | 每个卖家的唯一标识符 |
| `image` | URL | 直接链接到商品图片的 URL |
| `date_first_available` | 文本 | 商品首次上架的日期 |
| `discount` | 文本 | 商品折扣信息 |
| `model_number` | 文本 | 商品型号 |
| `manufacturer` | 文本 | 商品制造商 |
| `department` | 文本 | 商品所属部门 |
| `plus_content` | 布尔值 | 表示是否存在其他内容的布尔值 |
| `upc` | 文本 | 通用商品代码 |
| `video` | 布尔值 | 表示是否存在视频的布尔值 |
| `top_review` | 文本 | 商品的热门评论 |
| `final_price_high` | 价格 | 最终价格为范围时的最高值 |
| `final_price` | 价格 | 商品最终价格 |
| `variations` | 数组 | 同一商品不同变体的详细信息 |
| `delivery` | 数组 | 配送相关信息 |
| `features` | 数组 | 商品特性 |
| `format` | 数组 | 图书格式相关信息 |
| `buybox_prices` | 对象 | 商品价格详情 |
| `input_asin` | 文本 | 输入的 ASIN（当前未启用） |
| `ingredients` | 文本 | 商品配料，主要适用于食品 |
| `origin_url` | URL | 用于提取此记录的源页面 URL |
| `bought_past_month` | 数字 | 过去一个月的购买量（由 Amazon 显示） |
| `is_available` | 布尔值 | 商品是否仍然有货 |
| `root_bs_category` | 文本 | 畅销商品根分类 |
| `bs_category` | 文本 | 畅销商品分类 |
| `bs_rank` | 数字 | 特定分类中的畅销商品排名 |
| `badge` | 文本 | 商品徽章。例如：畅销商品第 1 名或 Amazon 精选 |
| `subcategory_rank` | 数组 | 按子分类划分的畅销商品排名条目 |
| `amazon_choice` | 布尔值 | 指定商品是否为 Amazon 精选 |
| `images` | 数组 | 商品图片的 URL |
| `product_details` | 数组 | 完整商品详情 |
| `prices_breakdown` | 对象 | 标价/常规价格及优惠状态的明细 |
| `country_of_origin` | 文本 | 商品原产国 |
| `from_the_brand` | 数组 | 页面上显示的品牌宣传媒体 |
| `product_description` | 数组 | 嵌入商品描述部分的媒体 |
| `seller_url` | URL | Amazon 上的卖家店面/资料 URL |
| `customer_says` | 文本 | 顾客评价 |
| `sustainability_features` | 数组 | 带参考资料的可持续性徽章/认证 |
| `climate_pledge_friendly` | 布尔值 | 商品是否显示 Climate Pledge Friendly 徽章 |
| `videos` | 数组 | 商品视频的 URL |
| `other_sellers_prices` | 数组 | 其他卖家提供的同一商品 |
| `downloadable_videos` | 数组 | 直接媒体 URL |
| `editorial_reviews` | 数组 | 图书的编辑评论 |
| `about_the_author` | 文本 | 作者简介 |
| `zipcode` | 文本 | 个人数据。用于配送和库存估算的 ZIP/邮政编码 |
| `coupon` | 文本 | 个人数据。优惠券 |
| `sponsered` | 布尔值 | 赞助 |
| `store_url` | URL | 商品的商店 URL |
| `ships_from` | 文本 | 商品发货地 |
| `city` | 文本 | 与配送、卖家或位置上下文相关的城市 |
| `customers_say` | 对象 | 从评论中提取的 Amazon 顾客评价摘要 |
| `max_quantity_available` | 数字 | 允许加入购物车的最大数量 |
| `variations_values` | 数组 | 变体及其可能的值 |
| `language` | 文本 | 商品页面/内容的语言 |
| `return_policy` | 文本 | 商品页面显示的退货政策文本 |
| `inactive_buy_box` | 对象 | 购物车不可用/未启用时的价格信息 |
| `buybox_seller_rating` | 数字 | 购物车卖家的评分 |
| `premium_brand` | 布尔值 | 是否为高端品牌 |
| `amazon_prime` | 布尔值 | 是否支持 Amazon Prime 配送 |
| `coupon_description` | 文本 | 优惠券描述 |
| `all_badges` | 数组 | 所有徽章 |
| `sponsored` | 布尔值 | Amazon 赞助标志 |
| `variant_id` | 文本 | 特定变体的唯一标识符 |
| `product_category` | 文本 | 使用分隔符连接的完整面包屑路径 |
| `category_tree` | 数组 | 由名称和 URL 对象组成的分类层级数组 |
| `availability_date` | 文本 | 缺货商品的预计到货日期 |
| `listing_has_variations` | 布尔值 | 商品信息是否包含多个变体 |
| `variant_attributes` | 数组 | 以名称/值对表示的当前变体属性 |
| `variants` | 数组 | 按类型分组的结构化变体选项 |
| `seller_privacy_policy` | 文本 | 卖家隐私政策 URL |
| `seller_tos` | 文本 | 卖家服务条款 URL |
| `return_window` | 数字 | 退货期限天数 |
| `target_countries` | 数组 | 商品可配送至的国家/地区 |
| `store_country` | 文本 | 商店所在国家/地区 |
| `category_urls` | 数组 | 分类面包屑 |
| `all_variations` | 布尔值 | 用于采集所有变体的输入字段 |
| `safety_information` | 文本 | 安全信息 |
| `subcategory_link` | 数组 | 按子分类划分的畅销商品链接条目 |
| `all_inactive_buy_box` | 数组 | 所有未启用的购物车信息 |
| `is_frequently_returned_item_badge` | 布尔值 | 表示是否存在该徽章 |
| `frequently_returned_item_message` | 文本 | 警告框中显示的文本 |
| `is_customers_usually_keep` | 布尔值 | 表示顾客通常是否会保留此商品 |
| `title_badge` | 文本 | 徽章标题 |
| `review_images` | 数组 | 评论图片 |
| `review_videos` | 数组 | 评论视频 |
| `also_viewed` | 数组 | 顾客还浏览了 |
| `similar_items` | 数组 | 考虑类似商品 |
| `bought_past_month_text` | 文本 | 过去一个月的购买量（由 Amazon 显示），文本格式。 |
| `is_high_price` | 布尔值 | 表示商品是否被标记为高价。 |
| `title_highlight` | 文本 | 记录 Amazon 在商品页面上紧接商品标题之后直接呈现的补充营销/亮点文本 |
| `title_clean` | 文本 | 卖家/品牌定义的实际商品标题，不包括 Amazon 在页面标题旁边或之后呈现的任何亮点文本 |
| `customers_say_topics` | 数组 | “顾客评价”部分的结构化明细 |
| `brand_url` | URL | 商品品牌 URL |
| `variant_condition` | 文本 | 商品变体的状况（例如：全新、翻新、优质翻新）。 |
| `is_aplus_premium` | 布尔值 | 表示商品是否为 Premium A+ |
| `buying_option_type` | 文本 | 购买选项类型 |
| `amazon_checkout_promotion` | 对象 | 商品结账促销 |
| `selected_format` | 文本 | 已选格式 |

</details>
<!-- fields:end -->

<details>
<summary>真实输出文件的开头，来自 <code>amazon-scraper B0CRMZHDG8</code></summary>

```json
{
  "generated_at": "2026-09-20T15:10:50.120Z",
  "products": [
    {
      "asin": "B0CRMZHDG8",
      "product": {
        "title": "STANLEY Quencher H2.0 Flow State Tumbler, 40 oz, Fuchsia",
        "seller_name": "Avrix Brands",
        "brand": "STANLEY",
        "description": "Constructed of recycled stainless steel for sustainable sipping, our 40 oz Quencher H2.0 offers maximum hydration with fewer refills. Commuting, studio workouts, day trips or your front porch—you’ll want this tumbler by your side. Thanks to Stanley’s vacuum insulation, your water will stay ice-cold, hour after hour. The advanced FlowState™ lid features a rotating cover with three positions: a straw opening designed to resist splashes while holding the reusable straw in place, a drink opening, and a full-cover top. The ergonomic handle includes comfort-grip inserts for easy carrying, and the narrow base fits just about any car cup holder.",
        "initial_price": 45,
        "currency": "USD",
        "availability": "In Stock",
        "reviews_count": 205036,
        "categories": [
          "Home & Kitchen",
          "Kitchen & Dining",
          "Storage & Organization",
          "Thermoses",
  ...
```

包含一个商品全部字段的完整文件位于 [examples/sample_output.json](examples/sample_output.json)。

</details>

## Amazon 爬虫错误

| 你看到的内容 | 含义 |
| --- | --- |
| `API token required but not found.` | 在发出任何请求前以状态 2 退出。请设置令牌。 |
| `failed  ASIN: ...` | 以状态 1 退出。没有该商品，通常是 ASIN 输入错误。 |
| `failed  ASIN: the API returned no row for this ASIN` | 以状态 1 退出。任务返回时缺少该输入对应的行。请重新运行。 |
| `failed  ASIN: Polling timed out after 605s for sd_...` | 以状态 1 退出。请求在 600 秒后放弃。请重新运行。 |

任何失败都会以状态 1 退出，因此可以安全地使用运行结果控制脚本是否继续。

在 SDK 中，相同情况如下所示：

| 你看到的内容 | 含义 |
| --- | --- |
| `AuthenticationError: No API token found.` | 任何位置都没有令牌：选项、环境变量和 CLI 登录中均不存在。 |
| `APIError`，状态为 401 | 令牌已设置，但不正确。 |
| `result.success` 为 `false`，`result.status` 为 `"timeout"` | SDK 已放弃等待。请增大以毫秒为单位的 `pollTimeout`，或重新运行。 |
| `result.data` 中带有 `error` 键的一行 | 当你请求 `includeErrors` 时，API 对某个输入给出的响应。其他行不受影响。 |
| 成功结果包含零行 | 在评论数据集中，这是由 `limitPerInput` 导致的。请移除它，并留意费用。 |

## Amazon MCP 服务器和 Claude Code 技能

无需 JavaScript，也无需安装任何内容。粘贴以下两行；第一行会打开一次浏览器，也可以在 SSH 和 CI 环境中使用 `bdata login --device`：

```bash
npx -p @brightdata/cli bdata login
npx -p @brightdata/cli bdata pipelines run gd_l7q7dkf244hwjntr0 collect_by_url "https://www.amazon.com/dp/B0CRMZHDG8"
```

这是 CLI 0.4.0 的语法；旧版 `bdata pipelines amazon_product` 形式已在该版本中移除。`bdata pipelines search amazon` 用于查找爬虫工具 ID。Amazon 对应的 ID 为 `gd_l7q7dkf244hwjntr0`（商品）、`gd_le8e811kzy4ggddlq`（评论）和 `gd_lwdb4vjm1ehb499uxs`（搜索），它们均使用 `collect_by_url` 方法。`bdata pipelines info <id> collect_by_url` 会列出抓取工具的输入字段。每个工具都接受 URL、输出 JSON，并按每条记录一个额度计费。

`npx skills add brightdata/skills` 会向 Claude Code、Cursor 和 Codex 提供这些命令和文档，因此之后可以直接使用自然语言。完整指南：[面向编码智能体的 Bright Data](https://docs.brightdata.com/quickstart-coding-agent)。

要为无终端的托管助手使用？[Bright Data MCP 服务器](https://github.com/bright-cn/brightdata-mcp#which-tool-to-use)的 `ecommerce` 组中提供了 Amazon 工具，除非你主动请求，否则该组默认关闭：

    https://mcp.brightdata.com/mcp?token=YOUR_API_TOKEN&groups=ecommerce

智能体也可以自行开设账户，无需填写注册表单：[智能体注册](https://brightdata.com/auth.md)。Bright Data 支持的其他所有集成（从 LangChain 到 Zapier 和 n8n）：[集成](https://docs.brightdata.com/integrations/introduction)。

## 这个 Amazon 爬虫工具免费吗？

新 Bright Data 账户[每月可获得 5,000 个免费额度](https://docs.brightdata.com/general/account/billing-and-pricing/free-tier)。商品和搜索结果按每条记录一个额度计费，评论按每条评论一个额度计费，因此免费额度每月可覆盖 5,000 个商品。[定价](https://www.bright.cn/pricing/web-scraper)中列出了超出后的费率。

## 支持

此代码仓库中的错误：[提交 issue](https://github.com/bright-cn/amazon-scraper-node/issues)，[CONTRIBUTING.md](CONTRIBUTING.md) 中说明了需要提供哪些信息。有关 API、你的账户或额度的任何问题，请联系 [Bright Data 支持团队](https://brightdata.zendesk.com/hc/en-us/requests/new)。

## 许可证

[Apache 2.0](LICENSE)。

## 适用于 Python 的 Amazon 爬虫工具

同一款 Amazon 爬虫工具也提供 Python 版本：[bright-cn/amazon-scraper-python](https://github.com/bright-cn/amazon-scraper-python)。

## Bright Data 提供的更多开源爬虫工具

其他以相同方式构建且示例经过实时验证的 Node.js 抓取工具：

- [适用于 Node.js 的 URL 转 Markdown（Fetch API）](https://github.com/bright-cn/fetch-api-scraper-node)
- [适用于 Node.js 的 TikTok 爬虫工具](https://github.com/bright-cn/tiktok-scraper-node)
- [适用于 Node.js 的 ChatGPT 爬虫工具](https://github.com/bright-cn/chatgpt-scraper-node)
