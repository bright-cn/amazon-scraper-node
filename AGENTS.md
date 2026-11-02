# 面向在此仓库中工作的编程智能体

在进行任何更改之前，请先阅读本文。以下每一行均已于 2026-09-20 针对实时 API 或已安装的 SDK（`@brightdata/sdk` 1.2.0）完成验证。

## 需要付费的部分

评论按每条评论扣除一个积分，且**此 SDK 中没有可用的数量上限**。

- API 接受 `max_reviews`，Bright Data 自己的文档示例将其设为 20。`AmazonCollectReviewsFilterSchema` 仅声明了 `url` 和 `reviews_to_not_include`，而且校验严格，因此会拒绝 `max_reviews`。
- `limitPerInput` 是 SDK 自身的上限，但在评论数据集上会返回**零行数据**。已针对两个商品测量三次，结果均为 `success: true`、`status: "ready"`，且没有错误行。同一选项在商品数据集上表现正常。
- 因此，唯一能返回任何数据的评论调用是不设上限的调用。`B0CRMZHDG8` 报告有 205,036 条评论。这只算一次调用。

问题记录在 [sdk-js#37](https://github.com/brightdata/sdk-js/issues/37)。在修复之前，README 中的任何代码块都不得调用 `collectReviews` 或 `reviews`，并且有测试会强制检查这一点。如果你需要有数量上限的数据切片，请直接使用 REST 端点。

## 一般的数量上限

`limitPerInput` 是发现调用的唯一数量限制，但除非同时传入 `async: true`，否则它会被静默移除。选项 schema 是以 `async` 为判别键的联合类型，而同步分支会丢弃未知键，因此数量上限会在不报错的情况下消失。带类型的 `DiscoverOptions` 会提示你省略 `async`，这意味着类型正确的调用反而是不设上限的调用。问题记录在 [sdk-js#36](https://github.com/brightdata/sdk-js/issues/36)。始终同时传入这两个选项。

已验证：使用 `limitPerInput: 3` 的 `discoverProductsByKeyword` 恰好返回了 3 行数据。

## 身份验证

- SDK 依次读取 `apiKey`、`BRIGHTDATA_API_TOKEN` 或 `BRIGHTDATA_API_KEY`，然后读取 `bdata login` 存储的凭据。
- 此 SDK 不会读取 `.env` 文件。Node 20 可以加载该文件：`node --env-file=.env`。
- 始终使用 `new bdclient({ autoCreateZones: false })`。创建区域需要付款方式，而此仓库从不使用区域。

## API 的行为方式

- Amazon 位于 `client.scrape.amazon`。商品数据集是 `gd_l7q7dkf244hwjntr0`，这也是控制面板显示的 ID，其中包含约 3 亿条记录。
- 轮询选项以毫秒为单位。`pollTimeout` 默认为 600000。
- API 给出的预估时间约为每个输入 7 秒，而单个商品通常会在 12 到 25 秒内返回。关键词发现的耗时远难预测：同一个调用有一次耗时 32 秒，另一次耗时 547 秒。因此，`live.yml` 为相应代码块预留了 15 分钟。
- 每个 ASIN 都会放入同一个任务。API 按记录计费，而不是按任务计费。
- 批量调用返回的数据行不保证顺序。`attribute()` 使用每行自身的 `asin` 将其与对应的 ASIN 匹配；如果没有该字段，则回退到 `input_url`、`url` 或 `input.url` 中的 ASIN。切勿按位置匹配。
- 失效的 ASIN 会返回一条错误内容为 "The navigation resulted in a dead page (404 status code)" 的数据行。请匹配消息，而不是错误代码。
- 每条记录消耗一个积分。每月免费提供 5,000 个积分，商品抓取工具的价格为每 1,000 条记录 1.50 美元。

## SDK 未暴露的功能

API 接受的筛选条件比 SDK 允许的更多。已针对 `GET /datasets/v3/scrapers?domain=amazon.com` 检查：

- 商品 `collect_by_url` 接受 `url`、`asin`、`origin_url`、`zipcode`、`language`、`all_variations`。SDK 筛选器接受 `url`、`zipcode`、`language`，并拒绝其余三个。因此，ASIN 必须写成 `/dp/` URL，`productUrl()` 正是这样做的。
- 还有第五个 Amazon 爬虫工具，即商品全球数据集（`gd_lwhideng15g8jg63s7`），支持按**卖家**和**品牌**发现数据，并可通过 `domain` 参数指定其他 Amazon 站点。两个 SDK 都未暴露该功能。
- 此 SDK 可以访问以下四个数据集：商品 `gd_l7q7dkf244hwjntr0`（119 个字段）、评论 `gd_le8e811kzy4ggddlq`（31 个）、卖家 `gd_lhotzucw1etoe5iw1k`（29 个）、商品搜索 `gd_lwdb4vjm1ehb499uxs`（30 个）。

## Schema

- 切勿硬编码字段列表。每个商品仅包含适用于自身的字段：README 中的两个商品分别返回了 119 个字段中的 99 个和 100 个。
- API 将 `seller_name`、`zipcode` 和 `coupon` 标记为 `pii: true`。卖家数据集还将 `email` 和 `seller_phone_number` 标记为该类型，因此处理卖家数据集时要比商品数据集更加谨慎。
- 请通过原始元数据端点读取 schema，就像 `live.yml` 中生成字段表的步骤一样。
- 完整文档索引：https://docs.brightdata.com/llms.txt

## API 会周期性停滞

2026-09-16，在构建对应的 Instagram 项目时，API 经历了多次持续约一小时的异常时段，其间大多数任务会一直停留到轮询截止时间，并返回 `status: "timeout"`，且没有任何数据行。如果实时检查失败且唯一症状是超时，很可能只是遇到了异常时段：请先重新运行，再检查是否发生了代码更改。

## 在此仓库中工作

- `npm test` 可离线运行，且不需要令牌。`npm run lint` 必须通过。
- CI 会在空白机器上使用 README 自身提供的命令进行安装。每周工作流会针对真实 API 执行 README 中的每个围栏代码块。
- 字段表和“最后验证”徽章会由每日任务重写。请勿手动编辑两者。
- 保持项目精简：共 15 个文件，`src/` 中约 350 行 JavaScript。不要添加重试、去重、调度、数据库或并发功能。
