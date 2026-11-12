# 供在此代码仓库中工作的编码智能体使用

更改任何内容前请先阅读本文。以下每一行均已于 2026-09-20 通过实时 API 或已安装的 SDK（`@brightdata/sdk` 1.2.0）验证。

## 会产生费用的调用

评论按每条评论一个额度计费，而且**此 SDK 中没有可用的数量上限**。

- API 接受 `max_reviews`，Bright Data 自己的文档示例将其设为 20。`AmazonCollectReviewsFilterSchema` 仅声明 `url` 和 `reviews_to_not_include`，且会进行严格校验，因此 `max_reviews` 会被拒绝。
- `limitPerInput` 是 SDK 自己的上限，但在评论数据集上返回**零行**。我们对两个商品测量了三次，结果均为 `success: true`、`status: "ready"`，且没有错误行。同一选项在商品数据集上表现正常。
- 因此，唯一能返回内容的评论调用是不设上限的调用。`B0CRMZHDG8` 显示有 205,036 条评论。这只是一次调用。

问题记录在 [sdk-js#37](https://github.com/brightdata/sdk-js/issues/37) 中。在修复前，README 中的代码块不得调用 `collectReviews` 或 `reviews`，并且有测试会强制执行此规则。如果需要有界的数据切片，请直接使用 REST 端点。

## 一般上限

`limitPerInput` 是发现调用的唯一数量限制，但除非同时传入 `async: true`，否则它会被静默移除。选项 schema 是以 `async` 为键的联合类型，同步调用部分会丢弃未知键，因此上限会在不报错的情况下消失。类型化的 `DiscoverOptions` 会提示你省略 `async`，这意味着类型正确的调用反而是不设上限的调用。问题记录在 [sdk-js#36](https://github.com/brightdata/sdk-js/issues/36) 中。始终同时传入两者。

已验证：使用 `limitPerInput: 3` 的 `discoverProductsByKeyword` 恰好返回了 3 行。

## 身份验证

- SDK 依次读取 `apiKey`、`BRIGHTDATA_API_TOKEN` 或 `BRIGHTDATA_API_KEY`，然后读取已存储的凭据 `bdata login`。
- 此 SDK 不会读取 `.env` 文件。Node 20 会加载该文件：`node --env-file=.env`。
- 始终使用 `new bdclient({ autoCreateZones: false })`。创建区域需要付款方式，而此代码仓库从不使用区域。

## API 的行为方式

- Amazon 位于 `client.scrape.amazon`。商品数据集是 `gd_l7q7dkf244hwjntr0`，这是控制面板显示的 ID，包含约 3 亿条记录。
- 轮询选项以毫秒为单位。`pollTimeout` 默认为 600000。
- API 预计每个输入约需 7 秒，单个商品通常会在 12 至 25 秒内返回。关键词发现的耗时远难预测：同一调用一次耗时 32 秒，另一次则耗时 547 秒。因此，`live.yml` 允许代码块运行 15 分钟。
- 所有 ASIN 都会放入同一个任务。API 按记录而非按任务计费。
- 批处理返回的行没有固定顺序。`attribute()` 会根据每行自身的 `asin` 将其与对应 ASIN 匹配；如果没有，则回退到 `input_url`、`url` 或 `input.url` 中的 ASIN。绝不能按位置匹配。
- 失效的 ASIN 会作为一行返回，其错误内容为“The navigation resulted in a dead page (404 status code)”。请匹配消息，而不是错误代码。
- 每条记录一个额度。每月免费提供 5,000 个额度，商品爬虫工具的价格为每 1,000 条记录 1.50 美元。

## SDK 未公开的功能

API 接受的筛选条件比 SDK 允许的更多。已根据 `GET /datasets/v3/scrapers?domain=amazon.com` 检查：

- 商品 `collect_by_url` 接受 `url`、`asin`、`origin_url`、`zipcode`、`language`、`all_variations`。SDK 筛选器仅接受 `url`、`zipcode`、`language`，并拒绝另外三个。因此，ASIN 必须写成 `/dp/` URL，`productUrl()` 正是这样做的。
- 还有第五个 Amazon 抓取工具，即商品全球数据集（`gd_lwhideng15g8jg63s7`），支持按**卖家**和**品牌**发现商品，并可使用 `domain` 参数指定其他 Amazon 站点。两个 SDK 都未公开它。
- 此 SDK 可访问的四个数据集：商品 `gd_l7q7dkf244hwjntr0`（119 个字段）、评论 `gd_le8e811kzy4ggddlq`（31 个）、卖家 `gd_lhotzucw1etoe5iw1k`（29 个）、商品搜索 `gd_lwdb4vjm1ehb499uxs`（30 个）。

## schema

- 绝不要硬编码字段列表。商品只会携带适用于它的字段：README 中的两个商品分别返回了 119 个字段中的 99 个和 100 个。
- API 将 `seller_name`、`zipcode` 和 `coupon` 标记为 `pii: true`。卖家数据集还会标记 `email` 和 `seller_phone_number`，因此处理该数据集时要比处理商品数据集更加谨慎。
- 请使用原始元数据端点读取 schema，就像 `live.yml` 中生成字段表的步骤所做的那样。
- 完整文档索引：https://docs.brightdata.com/llms.txt

## API 会周期性卡住

2026-09-16，在构建 Instagram 对应项目时，API 经历了数次持续约一小时的异常期，其间大多数任务一直停滞到轮询截止时间，然后返回没有任何行的 `status: "timeout"`。如果实时检查失败且唯一症状是超时，很可能只是遇到了异常期：先重新运行，再排查代码更改。

## 在此开发

- `npm test` 可离线运行且无需令牌。`npm run lint` 必须通过。
- CI 会在空白机器上使用 README 自己的命令安装。每周工作流会针对真实 API 执行 README 中每个带代码围栏的代码块。
- 字段表和“最后验证”徽章会由每周运行自动重写。不要手动编辑它们。
- 保持项目精简：`src/` 中包含 15 个文件和约 350 行 JavaScript。不要添加重试、去重、调度、数据库或并发功能。
