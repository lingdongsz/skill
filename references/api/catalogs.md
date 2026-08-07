# 目录链接接口

用于 Mercado Libre 目录链接、官链、跟卖商品、BSR 和目录每日数据分析。

## 目录链接相关接口选择规则

`catalog_search.py` 封装 3 个目录关联接口：

- `/catalogs/search`：目录链接列表搜索（按关键词/类目/销量/BSR/价格/评分/重量/跟卖状态筛选）。
  - **什么时候调用**：用户要"找官链机会""找目录链接""看跟卖品池""按 BSR 找品""筛选高销量目录商品"。
  - **什么时候不调用**：用户只问单个目录链接的详情、跟卖结构、BSR 趋势；不需要列表页。
  - **不要默认全查**：按需设置 `pageSize`，不要一上来拉超大分页。

- `/catalog/info`：单个目录链接详情 + 跟卖商品列表（`followItems`）。
  - **什么时候调用**：用户问某个目录链接里有哪些卖家在跟卖、跟卖价格/库存/销量对比、谁是跟卖头部。
  - **什么时候不调用**：只看目录搜索列表、只看 BSR 日趋势。

- `/catalog/daily`：目录链接每日 BSR、销量、价格趋势。
  - **什么时候调用**：用户问某个目录链接 BSR 是否稳定、销量是否在掉、价格变动趋势。
  - **什么时候不调用**：只看静态详情、只看跟卖结构。

### 组合调用规则

- 只有当用户要求"完整分析这个目录链接""把这个官链从跟卖到 BSR 都扒一遍"时，才同时查 `/catalog/info` + `/catalog/daily`。
- 普通目录选品：先 `/catalogs/search` 筛出候选链接，再对个别链接按需查 info 或 daily，不要默认每个链接都查详情+每日。
- 判断是否适合跟卖：优先 `/catalog/info` 看跟卖结构和价格分布；再按需用 `/catalog/daily` 验证稳定性。

## `/catalogs/search` 目录链接搜索

> ⚡ **积分消耗**：每次调用消耗 **4 积分**

返回目录链接列表。适合按关键词、类目、销量、BSR、价格、评分、重量、跟卖状态等筛选官链机会。

### ProductItem

- `id`: 商品 ID
- `title`: 商品标题
- `siteId`: 站点 ID
- `categoryId`: 类目 ID
- `price`: 价格
- `basePrice`: 原价
- `currencyId`: 币种 ID
- `availableQuantity`: 可用库存
- `soldQuantity`: 总销量
- `sale7` / `sale30d` / `sale60` / `sale90`: 各周期销量
- `sale714` / `sale3060`: 间隔期销量
- `gmv7` / `gmv30` / `gmv60` / `gmv90`: 各周期 GMV
- `sa7` / `sa30` / `sa60` / `sa90`: SA 指标
- `bsr`: BSR 排名
- `health`: 健康度
- `sellerId`: 卖家 ID
- `sellerName`: 卖家名称
- `sellerType`: 卖家类型
- `permalink`: 店铺链接
- `url`: 商品 URL
- `picSmall` / `picBig`: 图片
- `startTime` / `dateCreated` / `lastUpdated`: 时间信息
- `catalogListing`: 是否目录商品
- `isUsaFull`: 是否美国 FULL 仓
- `productId` / `familyId`: 产品 ID / Family ID
- `rating`: 评分对象
- `shipping`: 物流对象
- `pathFromRoot`: 类目路径

## `/catalog/info` 目录链接详情

> ⚡ **积分消耗**：每次调用消耗 **1 积分**（默认）

### ProductInfoData

- `productId`: 产品 ID
- `siteId`: 站点 ID
- `dateCreated`: 创建时间
- `name`: 产品名称
- `permalink`: 产品链接
- `pictures`: 图片列表
- `soldQuantity`: 销售数量
- `status`: 状态
- `categoryId`: 类目 ID
- `followCount`: 关注数量
- `followItems`: 跟卖商品列表，array<FollowItem>

### FollowItem

- `id` / `itemId`: 商品 ID
- `sellerId`: 卖家 ID
- `sellerName`: 卖家名称
- `price`: 价格
- `sale7` / `sale30d` / `sale60` / `sale90`: 销量
- `availableQuantity`: 库存
- `brandId`: 品牌 ID
- `picBig`: 图片
- `sellerType`: 卖家类型

## `/catalog/daily` 目录链接每日数据

> ⚡ **积分消耗**：每次调用消耗 **2 积分**

- `date`: 日期
- `bsr`: BSR 排名
- `soldQuantity`: 销售数量
- `price`: 销售价格

## 使用提示

- 找跟卖机会：关注高销量、BSR 靠前、评分一般、跟卖结构清晰的目录链接。
- 做差异化：看 `followItems` 里的价格、库存、卖家类型、品牌和销量。
- 看趋势：用 `/catalog/daily` 观察 BSR、销量、价格是否稳定。
