# 汇率与运费接口

用于汇率换算、运费查询和利润测算。

## 对应脚本路径

- 汇率 / 运费查询：通过 `scripts/get_item_info.py` 返回的 `rateSite` / `rateRmb` / `weightInfo` 子对象直接读取（推荐，无需额外请求）
- 单独调用汇率 / 运费接口：使用通用请求工具 `scripts/utils.py` 组装请求
- 利润测算场景：优先用 `scripts/get_item_info.py` 的内置汇率和运费字段，自定义重量/售价时再通过 `utils.py` 补查

## 汇率与运费相关接口选择规则

说明：`utils.py` 只提供通用 HTTP 工具，不直接单独封装汇率/运费脚本。汇率和运费通常有两种获取方式：
1. 优先从 `/item/info` 返回体里的 `rateSite` / `rateRmb` / `weightInfo` 子对象直接读取（大多数场景够用，不要额外发请求）。
2. 单独调用以下两个接口（用户只问汇率或运费、没有商品详情上下文时使用）。

### 接口列表与调用边界

- `/rate/{siteId}`：指定站点的汇率（美元、人民币）。
  - **什么时候调用**：用户只问"墨西哥比索汇率""MLM 汇率""巴西雷亚尔兑人民币"；或没有商品详情、需要独立查汇率。
  - **什么时候不调用**：已经查过 `/item/info` 且返回里有 `rateSite` / `rateRmb`，直接复用数据即可，不要再单独调汇率接口。

- `/cost/weight/*`：运费查询（按站点、重量区间、价格区间、卖家等级匹配运费）。
  - **什么时候调用**：用户要算利润、估运费、测算毛利；且 `/item/info` 里的 `weightInfo.finalFreight` 不能满足（例如需要批量测算、或用户自定义重量/价格/卖家等级参数）。
  - **什么时候不调用**：`/item/info` 已经返回了 `weightInfo.finalFreight` 且用户没有自定义参数。

### 组合调用与不要全查

- **利润测算组合**：先用 `/item/info` 拿价格、重量、币种、店铺类型、佣金、`rateSite`、`rateRmb`、`weightInfo.finalFreight`。90% 的情况这些字段已够用，不需要单独调汇率和运费接口。
- **只有当**：用户自定义了采购成本、自定义重量/售价/卖家等级，或 `/item/info` 里没有 `weightInfo` 时，才补调 `/rate/{siteId}` 和 `/cost/weight/*`。
- **不要默认全查**：不要一上来就汇率+运费两个接口都调，先看 `/item/info` 返回里有没有现成字段。

## `/rate/{siteId}` 汇率

> ⚡ **积分消耗**：每次调用消耗 **1 积分**（默认）

### RateSearchData

- `usdRate`: 美金汇率
- `cnyRate`: 人民币汇率
- `updateDate`: 更新时间

## `/cost/weight/*` 运费

> ⚡ **积分消耗**：
> - `/cost/weight/list/cbt` 自发货运费重量列表：每次调用消耗 **8 积分**（最高）
> - `/cost/weight/filter` 运费查询：每次调用消耗 **1 积分**（默认）

### WeightData

- `id`: ID
- `siteId`: 站点
- `country`: 国家
- `fromWeight`: 起始重量，单位 KG
- `toWeight`: 小于重量，单位 KG
- `fromPrice`: 售价大于
- `toPrice`: 售价小于
- `sellerLevel`: 店铺级别
- `label`: 二手商品描述
- `value`: 运费

## 使用提示

- 先从商品详情里拿价格、重量、币种、店铺类型和佣金。
- 再查汇率，把站点币种换算成美元或人民币。
- 最后按站点、重量、卖家等级、售价匹配运费。
- 用户没有提供采购成本时，只能做销售侧费用估算，不能给出完整净利。

## 本模块接口积分明细表

> 下表为本模块包含的所有接口及其单次积分消耗，与 SKILL.md 总表 / users.md 全表完全一致。
> 同一个接口调用多次需累计计算（例如汇率 + 运费列表 + 运费查询三个接口各跑一次 = 1 + 8 + 1 = 10 积分）。
> 利润测算时优先复用 `/item/info` 已返回的 `rateSite / rateRmb / weightInfo.finalFreight`，可省下这几笔。

| 接口路径 | 功能描述 | 消耗积分 |
|:---|:---|---:|
| `GET /cost/weight/list/cbt` | 自发货运费重量列表 | 8 |
| `GET /cost/weight/filter` | 运费查询 | 1 |
| `GET /rate/{siteId}` | 站点汇率查询 | 1 |
