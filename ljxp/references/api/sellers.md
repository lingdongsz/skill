# 店铺接口

用于店铺搜索、卖家画像、店铺风控和对标店铺池构建。

## 对应脚本路径

- 店铺搜索：`scripts/search_sellers.py`
- 通用请求、鉴权和格式化工具：`scripts/utils.py`

## 店铺相关接口选择规则

`search_sellers.py` 封装 1 个搜索接口：

- `/seller/search`：多维度店铺列表搜索（按站点/店铺类型/等级/优质卖家级别筛选）。
  - **什么时候调用**：用户要"找本土对标店铺""找跨境头部卖家""找绿级/铂金店铺""看某类目下都有什么店""店铺风控画像"。
  - **什么时候不调用**：用户只要商品列表、只要商品详情；不涉及店铺维度筛选。
  - **不要默认全查**：
    - 按需选 `sellerType`（LOCAL/CBT），不要一次把所有类型都跑一遍。
    - 按需选 `levelId`（5\_green/4\_light\_green/3\_yellow）和 `powerType`（platinum/gold/silver），默认只要绿级+铂金即可，不要默认把所有等级和贵金属级别都轮询。
    - 先拿前 50 个看是否够用，不要超大分页。

### 调用边界与组合

- **判断本土/跨境竞争格局**：用 `/seller/search` 分别筛 `LOCAL` 和 `CBT` 两类店铺数量和销量占比（或用 `trends.py` 的 `inventory_type` + `top_sellers`，不要两个工具重复做同一件事）。
- **看店铺商品矩阵**：拿到店铺 `id` 后，再用 `/items/search` 按 `sellerId` 搜索该店铺商品；但不要对搜索结果里每个店铺都跑商品搜索。
- **找高质量对标店铺**：优先 `LOCAL` + `5_green` + `platinum`，不要把黄级和白银级混进对标池除非用户明确要求。

## `/seller/search` 店铺搜索

> ⚡ **积分消耗**：每次调用消耗 **2 积分**

返回符合筛选条件的店铺列表。

### SellerDetail 基本字段

- `id`: 卖家 ID
- `name`: 卖家名称
- `permalink`: 店铺永久链接
- `remark`: 备注
- `site_id`: 站点 ID
- `country_id`: 国家 ID
- `time_zone`: 时区
- `registration_date`: 注册日期
- `seller_status`: 卖家状态
- `power_seller_status`: 优质卖家状态
- `level_id`: 等级 ID
- `seller_type`: 卖家类型
- `item_total`: 商品总数
- `sale_total`: 近一年销售总数
- `sale_completed`: 60 天完成销售数
- `sell_completed`: 作为卖家完成销售数
- `completed60`: 60 天完成数
- `sell_canceled`: 取消销售数
- `sale_cancel`: 销售取消数
- `cancel60` / `cancel60_rate`: 60 天取消数 / 取消率
- `delayed60` / `delayed60_rate`: 60 天延迟数 / 延迟率
- `claims60` / `claims60_rate`: 60 天投诉数 / 投诉率
- `sa30` / `sa60` / `sa90`: SA 指标
- `uddt`: 更新时间

### SellerAddress

- `city`: 城市
- `state`: 州/省

### SellerRatings

- `positive`: 好评数
- `neutral`: 中评数
- `negative`: 差评数

## 使用提示

- 找高质量本土对标店铺时，优先筛选 `seller_type=LOCAL`、绿级或浅绿级、铂金/黄金优质卖家。
- 判断跨境竞争时，关注 `seller_type=CBT` 及相关 CBT 类型。
- 风险判断优先看取消率、延迟率、投诉率和 SA 指标。
- 拿到 `id` 后，可用商品搜索按 `sellerId` 查看店铺商品矩阵。

## 本模块接口积分明细表

> 下表为本模块包含的所有接口及其单次积分消耗，与 SKILL.md 总表 / users.md 全表完全一致。
> 同一个接口调用多次需累计计算（例如按店铺类型分两页查询 = 2 × 2 = 4 积分）。

| 接口路径 | 功能描述 | 消耗积分 |
|:---|:---|---:|
| `POST /seller/search` | 店铺查询（分页列表） | 2 |

