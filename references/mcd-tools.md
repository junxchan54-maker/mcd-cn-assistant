# 麦当劳中国 MCP 工具速查

来源：<https://github.com/M-China/mcd-mcp-server>（版本日志至 1.0.9，2026-09-10）
服务地址：`https://mcp.mcd.cn`（Streamable HTTP，`Authorization: Bearer <你的 MCP Token>`）
调用名前缀：`mcp__mcd-mcp__<tool-name>`
**工具总数：35**（2026-10-09 对线上服务 `tools/list` 实测逐一核对；本文清单即为该次实测结果，与官方 README 的文字描述可能因版本推进存在差异，以线上实测为准）

## 活动 / 券 / 账户

| Tool name | 用途 | 写操作 |
|---|---|---|
| `now-time-info` | 获取当前完整时间信息，供推理当前日期 | 否 |
| `campaign-calendar` | 活动日历查询，返回当月进行中 / 往期 / 未来活动 | 否 |
| `available-coupons` | 麦麦省可领取的优惠券列表 | 否 |
| `auto-bind-coupons` | 麦麦省一键领券（自动领取所有可领券） | **是** |
| `query-my-coupons` | 我的优惠券查询（账户下全部可用券） | 否 |
| `query-my-account` | 我的积分查询（可用 / 累计 / 冻结 / 即将过期） | 否 |
| `query-store-coupons` | 指定门店下可用的优惠券列表 | 否 |
| `query-survey-coupon` | 按订单号查询本人订单的 CSAT 满意度答卷及关联奖券（标题 / 核销时间 / 核销状态 / 点餐方式） | 否 |

## 点餐主链路

| Tool name | 用途 | 写操作 |
|---|---|---|
| `query-nearby-stores` | 按地址查询附近可用门店 | 否 |
| `query-meals` | 查询当前门店可售卖餐品菜单（分类 / 编码 / 标签 / 优惠价） | 否 |
| `query-meal-detail` | 按餐品编码查详情、套餐组成、可替换/换购选项 | 否 |
| `calculate-price` | 按选购商品列表（可含优惠券）计算金额、配送费、优惠、应付总价 | 否 |
| `create-order` | 创建订单，返回订单详情与支付链接 | **是** |
| `query-order` | 查询订单状态 / 内容 / 配送信息 | 否 |
| `order-list` | 查询近期到店 / 外送历史订单（非商城订单） | 否 |
| `cancel-order` | 取消点餐订单 | **是** |
| `list-nutrition-foods` | 餐品营养信息（能量、蛋白质、脂肪、碳水、钠、钙） | 否 |
| `query-meal-assistance` | 企业团餐场景下的助餐服务查询（**仅企业团餐 beType=6**） | 否 |
| `query-promotions` | 企业团餐场景下的促销规则查询（满减 / 满折，**仅企业团餐 beType=6**） | 否 |

## 外送地址

| Tool name | 用途 | 写操作 |
|---|---|---|
| `delivery-query-addresses` | 查询用户已创建的配送地址列表 | 否 |
| `delivery-create-address` | 新增配送地址 | **是** |
| `delivery-query-stores` | 按收货地址查询可配送门店 | 否 |

## 积分商城

| Tool name | 用途 | 写操作 |
|---|---|---|
| `mall-points-products` | 麦麦商城商品列表（积分兑换 / 现金购买） | 否 |
| `mall-product-detail` | 商城商品详情（图片、积分、有效期、说明） | 否 |
| `mall-create-order` | 积分兑换商品下单，返回兑换订单号与券码 | **是** |
| `mall-order-list` | 麦麦商城近一年订单列表 | 否 |
| `mall-order-detail` | 商城订单详情（支付积分、支付金额、状态） | 否 |

## 积分抽奖

| Tool name | 用途 | 写操作 |
|---|---|---|
| `query-lottery-info` | 抽奖活动信息（状态、奖品、消耗规则、可用资源） | 否 |
| `draw-lottery` | 执行一次积分抽奖 | **是** |
| `query-my-prizes` | 我的奖品记录（分页、按中奖时间倒序） | 否 |

## 主题活动（派对 / 品鉴会）

| Tool name | 用途 | 写操作 |
|---|---|---|
| `query-party-city` | 主题活动可参与城市列表 | 否 |
| `query-party-store` | 指定城市下可参与门店列表 | 否 |
| `query-party-store-date` | 指定门店可预约日期 | 否 |
| `query-party-store-session` | 指定门店 + 日期下可预约场次 | 否 |
| `party-order-create` | 主题活动订单创建 | **是** |

## 典型调用顺序

**活动推送**：`now-time-info` → `campaign-calendar` →（可选）`available-coupons` / `query-my-coupons`

**最优惠点餐（到店）**：
`now-time-info` → `query-nearby-stores` → `available-coupons` →（用户同意）`auto-bind-coupons` → `query-my-coupons` + `query-store-coupons` → `query-meals` →（可选）`query-meal-detail` → `calculate-price`（多方案）→ 用户确认 → `create-order`

**最优惠点餐（外送）**：
`now-time-info` → `delivery-query-addresses` → `delivery-query-stores` → 其余同上（`calculate-price` 会含配送费）

## 错误码

| code | 原因 | 处理建议 |
|---|---|---|
| 401 | MCP Token 无效或已过期 | 检查 `Authorization` 请求头与 `~/.workbuddy/mcp.json` 配置，重新复制 Token |
| **403 + `校验鉴权authToken必填!`** | **`Authorization` 值缺 `Bearer ` 前缀**（只填了 Token 本身） | 值必须是 `Bearer <你的 MCP Token>`（Bearer 后一个空格）。**注意返回 403 而非 401**，容易漏查 |
| 429 | 触发限流（超过 600 次/分钟） | 降低请求频率，合理控制调用间隔 |

## 版本演进备忘

| 日期 | 版本 | 新增 |
|---|---|---|
| 2025-12-09 | 1.0.0 | 麦麦日历、麦麦省领券 |
| 2026-01-23 | 1.0.1 | 餐品营养信息列表 |
| 2026-02-13 | 1.0.2 | 麦乐送点餐、积分兑换券 |
| 2026-04-02 | 1.0.3 | 到店取餐、团餐 |
| 2026-05-21 | 1.0.4 | 积分兑换实物、商城订单、得来速车道、全部场景预约 |
| 2026-06-16 | 1.0.5 | 套餐内商品更换组合、部分餐品特制 |
| 2026-07-16 | 1.0.6 | 历史订单查询、餐品优惠价展示、随单购买麦金卡/早餐卡 |
| 2026-07-29 | 1.0.7 | 派对 / 品鉴会等主题活动查询、预约、下单 |
| 2026-08-27 | 1.0.8 | 积分抽奖（活动信息、抽奖、我的奖品） |
| 2026-09-10 | 1.0.9 | 取消订单、餐具选择、麦乐送备注、堂食外带取餐柜二维码 |
