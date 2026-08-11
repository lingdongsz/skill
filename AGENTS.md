## 项目概述

Mercado Libre 选品调研工具集（蓝鲸选品），通过 Python 脚本调用蓝鲸选品 API，支持类目树查询、关键词研究、竞品分析、店铺分析、品牌格局、目录链接机会、价格带分析、利润测算等功能。以 Skill 形式交付，供 Agent 按标准 9 步流程调用。

## 技术栈

- 语言：Python 3.12（仅标准库，无第三方依赖）
- API 基础：`https://xpskills.lingdongsz.com/api`
- 认证：环境变量 `LJXP_TOKEN`
- 支持站点：MLM（墨西哥）、MLB（巴西）、MLC（智利）、MLA（阿根廷）、MCO（哥伦比亚）

## 目录结构

```
/workspace/projects/
├── .coze                          # 项目配置
├── AGENTS.md                      # 本文件
├── ljxp-skills/
│   ├── SKILL.md                   # Skill 定义（标准 9 步流程、功能清单、参数说明）
│   ├── references/
│   │   ├── api_reference.md       # API 总览
│   │   ├── search_results_template.html  # 搜索结果 HTML 展示模板
│   │   └── api/                   # 各模块 API 参考文档
│   │       ├── category.md
│   │       ├── items.md
│   │       ├── keywords.md
│   │       ├── sellers.md
│   │       ├── trends.md
│   │       ├── catalogs.md
│   │       ├── rate-shipping.md
│   │       └── users.md
│   └── scripts/                   # Python 脚本（CLI 工具）
│       ├── utils.py               # 通用模块（HTTP、鉴权、格式化）
│       ├── category_tree.py       # 类目树查询
│       ├── search_items.py        # 商品搜索/竞品分析
│       ├── search_keywords.py     # 关键词研究
│       ├── search_sellers.py      # 店铺搜索
│       ├── trends.py              # 趋势/价格带/品牌分析
│       ├── catalog_search.py      # 目录链接搜索
│       └── get_item_info.py       # 商品详情/利润测算
```

## 关键入口 / 核心模块

- **SKILL.md**：Skill 入口定义，包含标准 9 步流程、10 类业务功能、参数清单
- **scripts/utils.py**：所有脚本的公共依赖，封装 HTTP 请求、Token 鉴权、结果格式化
- **scripts/*.py**：各业务功能脚本，通过 CLI 参数调用 API

## 运行与预览

- 本项目为脚本/工具集合，非 web/小程序/App 产物，不支持预览（`preview_enable = "disabled"`）
- 脚本执行方式：`python3 ljxp-skills/scripts/<脚本名>.py --site <站点> [其他参数]`
- 需要设置环境变量 `LJXP_TOKEN` 提供 API 认证

## 用户偏好与长期约束

- API 调用消耗积分（1~10 积分/次），必须先澄清意图再调用
- 不要在回复中暴露 Token
- 类目树查询的 `--depth`/`--search`/`--refresh` 是脚本本地参数，不能传给后端

## 常见问题和预防

- 类目 ID 格式如 `MLM458037`，只有品类名时先用 `category_tree.py --search` 定位
- 类目树缓存 7 天，`--refresh` 可强制刷新
- 所有脚本仅依赖 Python 标准库，无需 pip install
