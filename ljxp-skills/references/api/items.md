# 商品接口

用于商品搜索、商品详情、评价、每日/月度数据、流量词反查。

## 对应脚本路径

- 商品搜索：`scripts/search_items.py`
- 商品详情、每日数据、每月数据、评价、流量词反查：`scripts/get_item_info.py`
- 通用请求、鉴权和格式化工具：`scripts/utils.py`

## 商品相关接口选择规则

商品相关接口由 2 个脚本封装：

### `search_items.py` 封装 1 个搜索接口：

- `/items/search`：多维度商品列表搜索（按关键词/类目/店铺/价格/销量/评分等筛选）。
  - **什么时候调用**：用户要"找商品""看竞品列表""按条件筛选商品""某类目下有什么好卖的""某店铺有哪些商品"。
  - **什么时候不调用**：用户只问单个商品详情、评价、趋势；不需要列表页。
  - **不要默认全查**：按需设置 `pageSize`，不要一上来拉取超大分页。

### `get_item_info.py` 封装 5 个商品关联接口：

- `/item/info`：基础信息、价格、店铺、类目、物流、佣金、变体、评分。
  - **什么时候调用**：用户问单个商品详情、价格、店铺是谁、类目、物流、佣金、变体情况、评分概览。用户没有指定查哪方面时，优先只查这个。
  - **什么时候不调用**：用户只要评价内容、只要价格走势、只要流量词，此时不需要查 info。
- `/item/daily`：日级销量、价格、库存、访问量趋势。
  - **什么时候调用**：用户问最近走势、最近 7/14/30 天价格变化、库存是否在掉、日销量波动。
  - **什么时候不调用**：只看静态详情、只看月度汇总。
- `/item/monthly`：月度销量、销售额、访问量。
  - **什么时候调用**：用户问长期趋势、月度表现、季节性规律、月度 GMV。
  - **什么时候不调用**：只看最近几天波动、只看静态详情。
- `/item/review`：评价内容、评分、用户痛点。
  - **什么时候调用**：用户问差评原因、口碑怎么样、产品有什么问题/优点、用户反馈。
  - **什么时候不调用**：只要销量价格数据、不需要看用户文字评论。
- `/item/keyword/reverse`：流量来源、关键词排名、SEO/广告词。
  - **什么时候调用**：用户问关键词、流量来源、标题怎么优化、竞品在投什么词、商品靠什么词出单。
  - **什么时候不调用**：只要商品基础信息、不要关键词分析。

### 组合调用规则

- 只有当用户明确要求"完整竞品拆解""全量商品分析""把这个商品彻底分析一遍""从销量到评价到流量都看看"时，才组合查询以上 5 个接口（即脚本 `--all` 参数）。
- 普通商品详情查询不要默认全查。优先 `/item/info`，根据用户追问再补 daily/monthly/review/keyword/reverse。
- 先 `/items/search` 锁定商品池，再对个别商品用 `get_item_info.py` 深挖，不要对搜索结果里每个商品都调用 5 个子接口。

## `/items/search` 商品搜索

> ⚡ **积分消耗**：每次调用消耗 **5 积分**

返回分页商品列表。

### 顶层字段

- `code`: 响应码，integer
- `pageNo`: 当前页码，integer
- `pageSize`: 每页条数，integer
- `total`: 总记录数，integer
- `totalMore`: 总数是否更多，`Eq` / `Lt` / `Gt`
- `costTime`: 耗时毫秒，integer
- `msg`: 响应消息，string
- `data`: 商品详情列表，array<ItemDetail>

### ItemDetail 基本信息

- `id`: 商品 ID
- `title`: 商品标题
- `titleCn`: 商品标题中文
- `price`: 价格
- `priceUsd`: 美元价格
- `basePrice`: 原价
- `currencyId`: 货币 ID
- `siteId`: 站点 ID
- `itemStatus`: 商品状态
- `permalink` / `url`: 商品链接
- `listingType`: 流量类型
- `categoryId`: 分类 ID
- `brandId`: 品牌 ID
- `domainId`: 域名 ID

### ItemDetail 店铺信息

- `sellerId`: 店铺 ID
- `sellerName`: 店铺名称
- `sellerTitle`: 店铺头衔
- `sellerTitleValue`: 店铺头衔值
- `sellerLevel`: 店铺等级
- `sellerType`: 店铺类型，通常为 `LOCAL` / `CBT`
- `registrationDate`: 店铺注册日期
- `sellerTotal`: 卖家商品总数

### ItemDetail 时间信息

- `dateCreated`: 创建日期
- `startTime`: 上架时间
- `lastUpdated`: 最后更新时间
- `bsrDate`: BSR 日期
- `loadDate`: 更新日期

### ItemDetail 库存与销售

- `availableQuantity`: 库存
- `soldQuantity`: 已售数量
- `variationsCount`: 变体数量
- `freeShipping`: 是否包邮
- `bsr`: BSR 排名
- `health`: 健康度
- `isUsaFull`: 是否美国直邮
- `followItem`: 关注商品
- `catalogListing`: 是否目录链接
- `productId`: 目录商品 ID
- `promotion`: 促销

### ItemDetail 销售周期字段

- `sale7`: 7 天销量
- `gmv7` / `salesAmount7`: 7 天销售额
- `saleRate7`: 7 天销售率
- `sale714`: 7-14 天销量
- `sale30`: 30 天销量
- `gmv30`: 30 天销售额
- `saleRate30`: 30 天销售率
- `visit30`: 30 天访问量
- `sale3060`: 30-60 天销量
- `sale60`: 60 天销量
- `gmv60`: 60 天销售额
- `sale90`: 90 天销量
- `gmv90`: 90 天销售额
- `visitTotal`: 总访问量

### ItemDetail 图片与物流

- `picBig`: 大图
- `picSmall`: 小图
- `logisticType`: 物流类型
- `shippingHtmlType`: 配送 HTML 类型
- `packetLength`: 包裹长度
- `packetWidth`: 包裹宽度
- `packetHeight`: 包裹高度
- `packetWeight`: 包裹重量

### ItemDetail 子对象

- `shipping.mode`: 配送模式
- `shipping.freeShipping`: 是否包邮
- `shipping.logisticType`: 物流类型
- `shipping.tags`: 标签列表
- `rating.amount`: 评分数量
- `rating.stars`: 星级
- `pathFromRoot[].name`: 分类名称
- `pathFromRoot[].id`: 分类 ID

## `/item/info` 商品详情

> ⚡ **积分消耗**：每次调用消耗 **1 积分**（默认）

返回单个商品的完整信息。

### ItemInfoData 基本信息

- `id`: 商品 ID
- `siteId`: 站点
- `title`: 标题
- `uddt`: 更新时间
- `sellerId`: 店铺 ID
- `domainId`: 类目领域 ID
- `categoryId`: 类目 ID
- `categoryName`: 分类名称
- `categoryNameZn`: 分类中文名
- `price`: 单价
- `avgPrice`: 平均单价
- `minPrice`: 最小单价
- `maxPrice`: 最大单价
- `basePrice`: 原价
- `currencyId`: 币种
- `pictures`: 图片列表

### ItemInfoData 销售数据

- `sale7`: 7 天销量
- `sale30`: 30 天销量
- `sale714`: 7-14 天销量
- `sale3060`: 30-60 天销量
- `sale60`: 60 天销量
- `sale90`: 90 天销量
- `saleRate7`: 7 天转换率
- `saleRate30`: 30 天转换率
- `gmv7`: 7 天销售额
- `gmv30`: 30 天销售额
- `gmv60`: 60 天销售额
- `gmv90`: 90 天销售额

### ItemInfoData 库存与状态

- `availableQuantity`: 库存
- `initialQuantity`: 期初数量
- `soldStock`: 销量概数
- `soldQuantity`: 销量
- `itemStatus`: 状态
- `listingMode`: 曝光级别
- `variationsCount`: 变体数量

### ItemInfoData 其他字段

- `url`: 商品链接
- `picSmall` / `picBig`: 商品图片
- `brandId`: 品牌
- `startTime`: 上架时间
- `health`: 健康度
- `packetWeight`: 装箱重量
- `catalogListing`: 是否目录链接
- `cbtItem`: 是否 CBT 商品
- `visitTotal`: 总访问量
- `isUsaFull`: 支持美国转运仓
- `dateCreated`: 创建时间
- `lastUpdated`: 更新时间
- `promotion`: 秒杀
- `promotionLd`: 限时秒杀
- `sellerName`: 店铺名称
- `shippingHtmlType`: 物流类型
- `logisticType`: 仓储类型
- `saleFeeAmount`: 佣金

### ItemInfoData 子对象

- `rating.amount`: 评价数量
- `rating.stars`: 星级评分
- `pathFromRoot[].name`: 类目名称
- `pathFromRoot[].id`: 类目 ID
- `shipping.mode`: 运输模式
- `shipping.freeShipping`: 是否包邮
- `shipping.logisticType`: 物流类型
- `shipping.tags`: 标签列表
- `variations[].id`: 变体 ID
- `variations[].availableQuantity`: 变体库存
- `variations[].price`: 变体单价
- `variations[].sale30`: 变体 30 天销量
- `variations[].soldQuantity`: 变体销量
- `variations[].pictureIds`: 变体图片 ID
- `variations[].attrs[].name`: 属性名称
- `variations[].attrs[].id`: 属性 ID
- `variations[].attrs[].valueId`: 属性值 ID
- `variations[].attrs[].valueName`: 属性值名称
- `sellerInfo.id`: 店铺 ID
- `sellerInfo.name`: 店铺名称
- `sellerInfo.siteId`: 站点 ID
- `sellerInfo.permalink`: 店铺链接
- `sellerInfo.sellerStatus`: 状态
- `sellerInfo.levelId`: 级别
- `sellerInfo.countryId`: 国家 ID
- `sellerInfo.saleTotal`: 近一年销量
- `sellerInfo.itemTotal`: 店铺商品数量
- `sellerInfo.brands`: 品牌列表
- `sellerInfo.registrationDate`: 注册日期
- `sellerInfo.sellerType`: 店铺类型
- `rateSite` / `rateRmb`: 汇率对象
- `weightInfo.finalFreight`: 最终运费
- `weightInfo.weight`: 重量
- `weightInfo.additionalCost`: 附加费

## `/item/review` 商品评价

> ⚡ **积分消耗**：每次调用消耗 **1 积分**（默认）

### ReviewData

- `pageNo`: 当前页
- `pageSize`: 总页数
- `total`: 总记录数
- `reviewList`: 评论列表，array<Review>

### Review

- `itemId`: 商品 ID
- `productId`: 目录链接 ID
- `content`: 评论内容
- `contentZh`: 中文翻译
- `rate`: 评分
- `createTime`: 评论时间

## `/item/daily` 商品每日数据

> ⚡ **积分消耗**：每次调用消耗 **2 积分**

- `date`: 日期
- `availableQuantity`: 库存
- `currencyId`: 币种
- `price`: 单价
- `soldQuantity`: 当日销量
- `visit`: 当天访问量

## `/item/monthly` 商品每月数据

> ⚡ **积分消耗**：每次调用消耗 **2 积分**

- `month`: 月份
- `amount`: 销售额
- `currencyId`: 币种
- `soldQuantity`: 当月销量
- `visit`: 月访问量

## `/item/keyword/reverse` 商品流量词反查

> ⚡ **积分消耗**：每次调用消耗 **2 积分**

### KeywordReverseData

- `itemInfo`: 商品信息
- `keyList`: 关键词列表，array<KeyItem>

### itemInfo 常用字段

- `id`: 商品 ID
- `title`: 标题
- `price`: 价格
- `sale30`: 30 天销量
- `siteId`: 站点
- `rating.stars`: 评分
- `visit30`: 30 天访问量
- `picSmall`: 图片
- `url`: 链接

### KeyItem

- `key`: 关键词
- `keyCn`: 关键词中文
- `totalItem`: 总商品数
- `keyCount`: 关键词数量
- `sale30`: 30 天销量
- `bgl`: 流量占比
- `paiming` / `ranking`: 排名
- `visit30`: 30 天访问量
- `runDate`: 时间
- `history`: 历史数据

## 本模块接口积分明细表

> 下表为本模块包含的所有接口及其单次积分消耗，与 SKILL.md 总表 / users.md 全表完全一致。
> 同一个接口调用多次需累计计算（例如商品搜索后对 5 个竞品各查一次详情 + 一次每日数据 = 5 + (1 + 2) × 5 = 20 积分）。
> 普通商品详情优先只查 `/item/info`，用户明确要求「全量竞品拆解」时再用 `--all` 组合调用剩余 5 个接口，避免浪费积分。

| 接口路径 | 功能描述 | 消耗积分 |
|:---|:---|---:|
| `GET /items/search` | 商品查询（分页列表） | 5 |
| `GET /item/daily` | 商品每日历史数据 | 2 |
| `GET /item/monthly` | 商品每月历史数据 | 2 |
| `GET /item/keyword/reverse` | 商品流量词反查 | 2 |
| `GET /item/info` | 商品详情概要 | 1 |
| `GET /item/review` | 商品评价信息 | 1 |

