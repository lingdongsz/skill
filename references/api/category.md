# 类目接口

用于查询站点全量类目树，为下游趋势/搜索/价格等接口提供 `--category-id`。

## 【⚠️ 极其重要：后端 vs 脚本边界】

- **真实后端 `GET /category/tree` 只接受 1 个参数：`siteId`**。绝对没有 `depth`/`keyword`/`search`/`page` 等参数，**不要**把这些传给后端。
- 「关键词搜索」「按深度截断」「JSON 导出」「缓存刷新」全部是 **Python 脚本本地后处理能力**，不发额外请求、不额外扣积分。

## 对应脚本

- `scripts/category_tree.py`：同站点**首次请求 10 积分**，命中本地缓存**免费**（过期判定：`cacheLastModified + 7 天`）

## 接口选择规则

| 场景 | 是否请求后端 | 是否消耗积分 |
|---|:-:|:-:|
| 首次查某站点（无缓存文件） | ✅ | 10 积分，写入 `cache/category_tree_<site>.json` |
| 有缓存且未过期、无 `--refresh`（即使有 `--search`/`--depth`） | ❌（只读本地） | 0 积分，可省略 `--token` |
| 加了 `--refresh` 强制刷新 | ✅（重新拉取覆盖缓存） | 10 积分 |
| 用户已给出明确类目 ID（如 `MLM458037`） | ❌（直接跳过脚本） | 0 积分 |

缓存文件位置：`skills/ljxp-skills/cache/category_tree_<站点>.json`

---

## GET /category/tree 站点类目树

> ⚡ 请求后端 **10 积分** ｜ 命中缓存 **免费**

### 请求参数（后端只接这一个！）

| 必填 | 参数 | 说明 |
|:-:|---|---|
| ✅ | siteId | `MLM` / `MLB` / `MLC` / `MLA` / `MCO` |

### 脚本本地后处理参数（只影响展示，不发后端）

| 参数 | 说明 |
|---|---|
| `--search "关键词"` | 本地过滤：匹配 `zhName`/`name`/`id`，保留命中节点+完整祖先链 |
| `--depth N` | 本地打印截断：默认 3，`0` 不限 |
| `--output json` | 本地输出 JSON 格式 |
| `--refresh` | 忽略缓存，强制重拉后端覆盖缓存（本次 10 积分） |
| `--cache-ttl 秒数` | 本地 TTL：默认 604800=7 天，`0` 永不过期 |
| `--token` | 仅真实请求后端时需要；命中缓存可省略 |

### 返回字段

- `data.list[]`：递归类目树，每个节点：`id`（类目ID，如 `MLM458037`，传给下游的 `--category-id`）｜ `name`（英文）｜ `zhName`（中文）｜ `children[]`（递归）
- `data.cacheLastModified`：后端类目 JSON 最后修改时间（`yyyy-MM-dd HH:mm:ss`），过期判定基准

---

## 使用提示

1. 默认 `--depth 3` 防止树太大 Markdown 放不下；用户要完整结构才 `--depth 0`
2. 定位类目优先用 `--search <关键词>`（本地免费），同时匹配中英文和ID
3. 拿到目标 `id` 后传给下游：`trends.py / search_items.py / catalog_search.py` 等
4. 同站点第一次付费后，后续所有关键词搜索/深度浏览都免费；`--refresh` 只在 `cacheLastModified` 距今超 7 天或平台公告改版时用
5. 跨站点对比：不同站点类目 ID **不通用**，每个站点单独首次付费
6. Token 只在首次或 `--refresh` 时需要

---

## 本模块积分明细表

| 接口 | 功能 | 积分 |
|---|---|---:|
| `GET /category/tree` | 站点类目树（首次 10 / 命中缓存免费；过期判定 `cacheLastModified + 7 天`） | 10 |
