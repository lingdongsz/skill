# 行业趋势接口

用于类目机会、价格带、销量历史、新品机会、仓储结构、竞争店铺、竞争商品和竞争品牌分析。

## 对应脚本路径

- 行业趋势查询：`scripts/trends.py`
- 通用请求、鉴权和格式化工具：`scripts/utils.py`

## 行业趋势相关接口选择规则

`trends.py` 通过 `--type` 参数封装 9 个子接口：

### 核心三件套（类目机会判断默认只查这 3 个）

- `/trends/statistical`（`--type statistical`）：行业统计概览——总商品数、活跃数、总销量、总销售额、均价、月/日环比。
  - **什么时候调用**：用户问"这个类目怎么样""类目大盘数据""市场容量大不大""最近增长还是下滑"。
  - **什么时候不调用**：只要具体商品/店铺/品牌排行，只要价格带，只要销量曲线。
- `/trends/sold/his`（`--type sold_his`）：销量历史曲线——每日销量、销售额、单价趋势。
  - **什么时候调用**：用户问"这个类目最近销量走势如何""类目有没有季节性""最近 30/60/90 天类目趋势"。
  - **什么时候不调用**：只要静态快照、只要价格带、只要竞争排行。
- `/trends/brand/top/sellers`（`--type top_sellers`）：竞争店铺排行——类目 Top 店铺及其商品数/均价/销量/类型。
  - **什么时候调用**：用户问"这个类目都有哪些头部店铺""本土/跨境店铺格局""头部店铺销量集中度"。
  - **什么时候不调用**：只要品牌排行、只要商品排行、只要价格带。

### 按需补充的 6 个接口（不要默认全查）

- `/trends/price/list`（`--type price_list`）：价格区间分布——各价格段商品数/数值/价格。
  - **什么时候调用**：用户问"什么价格好卖""价格带分布""定价建议""哪个价格段竞争少"。
  - **什么时候不调用**：不涉及定价策略时不查。
- `/trends/brand/top/brands`（`--type top_brands`）：竞争品牌排行——类目 Top 品牌及其商品数/均价/销量。
  - **什么时候调用**：用户问"品牌集中度高吗""有哪些头部品牌""白牌有机会吗""品牌垄断判断"。
  - **什么时候不调用**：不涉及品牌格局时不查。
- `/trends/brand/top/items`（`--type top_items`）：竞争商品排行——类目 Top 商品及其品牌/价格/销量。
  - **什么时候调用**：用户问"这个类目爆款有哪些""头部商品长什么样""代表商品"。
  - **什么时候不调用**：不看具体爆款列表时不查（可用 `/items/search` 按类目+销量排序替代，不要重复查）。
- `/trends/sale/list`（`--type sale_list`，需 `--month`）：子类目销量分布——各子类目销量排行。
  - **什么时候调用**：用户问"一级类目下哪个子类目好卖""子类目机会判断""细分类目销量对比"。
  - **什么时候不调用**：没有子类目拆分需求时不查。
- `/trends/store/inventoryType`（`--type inventory_type`，需 `--month`）：仓储类型分布——FBM/FULL/CBT/本土仓销量和占比。
  - **什么时候调用**：用户问"本土/跨境格局""自发货还是海外仓""仓储模式判断""美国仓占比"。
  - **什么时候不调用**：不涉及仓储/本土跨境策略时不查。
- `/trends/new/items`（`--type new_items`）：新品机会指数——新品销量占比、每日/每周/每月上架趋势。
  - **什么时候调用**：用户问"新品好做吗""最近上架趋势""新品占比高不高""上新节奏判断"。
  - **什么时候不调用**：不涉及新品维度时不查。

### 组合调用规则

- **默认不要把 9 个接口全查一遍**。绝大多数"类目能不能做"场景，只查前三件套（statistical + sold\_his + top\_sellers）已经足够给业务结论。
- **完整类目调研**（用户明确说"完整类目调研""把这个类目所有维度都分析"）才把 9 个接口都跑一遍。
- **价格带专项**：statistical + price\_list + 再用 `/items/search` 验证具体商品形态。
- **品牌格局专项**：statistical + top\_brands + （必要时）top\_sellers。
- **新品机会专项**：statistical + new\_items + （必要时）price\_list。
- 不要与其他脚本重复做同一件事：判断本土/跨境格局，用 `inventory_type` 或 `top_sellers` 二选一，不要再加 `/seller/search` 做第三次。

## `/trends/statistical` 统计概览

> ⚡ **积分消耗**：每次调用消耗 **2 积分**

- `total`: 总商品数
- `hbTotal`: 月环比增长
- `rhbTotal`: 日环比增长
- `itemCount`: 近 30 天活跃商品数
- `soldTotal`: 总销量
- `amountTotal`: 总销售额
- `avgSold`: 近 30 天日平均销量
- `avgAmount`: 平均成交价
- `monthGrowth`: 月销量环比增长

## `/trends/sold/his` 销量历史

> ⚡ **积分消耗**：每次调用消耗 **2 积分**

- `dataList`: 销量数据，array<DateItem>
- `currencyId`: 币种

### DateItem

- `date`: 日期，`YYYYMMDD`
- `sold`: 销量
- `amount`: 销售额
- `price`: 销售额均价(销售额/销量)

## `/trends/sale/list` 销量分布

> ⚡ **积分消耗**：每次调用消耗 **2 积分**

- `soldTotal`: 总销量
- `categoryId`: 类目 ID
- `name`: 类目名称
- `nameZn`: 类目中文名称

## `/trends/price/list` 价格分布

> ⚡ **积分消耗**：每次调用消耗 **2 积分**

- `key`: 价格区间
- `count`: 商品数量
- `value`: 近30天销量
- `price`: 平均价格

## `/trends/store/inventoryType` 仓储类型分布

> ⚡ **积分消耗**：每次调用消耗 **2 积分**

- `fbm`: FBM 占比
- `all`: 全部统计，通常含 `amount` / `sale30` / `avgPrice` / `totalItems`
- `full`: FULL 仓统计
- `cbt`: CBT 统计
- `localFull`: 本土 FULL 统计
- `cbtFull`: 跨境 FULL 统计
- `cbtNotFull`: 跨境非 FULL 统计
- `localNotFull`: 本土非 FULL 统计
- `usa`: 美国仓统计，如 `usaCbtCount` / `usaCbtSale` / `usaFullSale`

## `/trends/new/items` 新品机会

> ⚡ **积分消耗**：每次调用消耗 **2 积分**

- `sale30` / `sale60` / `sale90` / `sale180`: 各周期总销量
- `newSale30` / `newSale60` / `newSale90` / `newSale180`: 新品各周期销量
- `newRate30` / `newRate60` / `newRate90` / `newRate180`: 新品占比
- `dailySaleList`: 每日上架趋势，`date` / `value`
- `weeklySaleList`: 每周上架趋势，`week` / `value`
- `monthlySaleList`: 每月上架趋势，`month` / `value`

## `/trends/brand/top/sellers` 竞争店铺

> ⚡ **积分消耗**：每次调用消耗 **2 积分**

- `categoryName`: 类目名称
- `categoryNameCn`: 类目中文名称
- `allCount`: 总商品数
- `brandCount`: 品牌数
- `sellerCount`: 店铺总数
- `count`: 活跃店铺数
- `sale30`: 该类目近30 天总销量
- `sumSale30`: 前Top10店铺近30 天销量
- `currencyId`: 币种
- `topList`: 店铺排行，array<TopSeller>

### TopSeller

- `id`: 店铺 ID
- `key`: 店铺名称
- `count`: 商品数
- `price`: 商品均价
- `volume`: 销量
- `url`: 店铺链接
- `sellerType`: 店铺类型

## `/trends/brand/top/items` 竞争商品

> ⚡ **积分消耗**：每次调用消耗 **2 积分**

结构与竞争店铺类似，`topList` 为 TopItem。

### TopItem

- `id`: 商品 ID
- `key`: 品牌或聚合键
- `price`: 价格
- `volume`: 销售额
- `sale30`: 30 天销量
- `url`: 商品链接
- `title`: 商品标题

## `/trends/brand/top/brands` 竞争品牌

> ⚡ **积分消耗**：每次调用消耗 **2 积分**

结构与竞争店铺类似，`topList` 为 TopBrand。

### TopBrand

- `key`: 品牌名
- `count`: 商品数
- `price`: 商品均价
- `volume`: 销售额
- `url`: 链接
- `value`: 近30天销量

## 使用提示

- 类目能不能做：先看统计概览、销量历史、竞争店铺、竞争品牌。
- 定价：看价格分布，再用商品搜索验证具体商品形态。
- 新品机会：看新品销量占比和上架趋势。
- 本土/跨境格局：看仓储类型分布和竞争店铺类型。

## 本模块接口积分明细表

> 下表为本模块包含的所有接口及其单次积分消耗，与 SKILL.md 总表 / users.md 全表完全一致。
> 同一个接口调用多次需累计计算（例如完整类目调研 9 个接口各跑一次 = 2 × 9 = 18 积分）。

| 接口路径 | 功能描述 | 消耗积分 |
|:---|:---|---:|
| `GET /trends/price/list` | 行业价格分布 | 2 |
| `GET /trends/new/items` | 新品机会指数 | 2 |
| `GET /trends/brand/top/brands` | 竞争品牌（Top Brands） | 2 |
| `GET /trends/brand/top/sellers` | 竞争店铺（Top Sellers） | 2 |
| `GET /trends/sale/list` | 行业销量分布 | 2 |
| `GET /trends/sold/his` | 行业销量历史 | 2 |
| `GET /trends/statistical` | 行业趋势统计概览 | 2 |
| `GET /trends/store/inventoryType` | 仓储类型分布（FBA/FBM） | 2 |
| `GET /trends/brand/top/items` | 竞争商品（Top Items） | 2 |

