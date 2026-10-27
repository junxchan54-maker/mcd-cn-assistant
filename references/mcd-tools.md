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

> ⚠️ **券 ID 只有 `query-store-coupons` 返回**（2026-10-16 实测）：`query-my-coupons` 与 `available-coupons` 只给标题 / 有效期 / 状态 / 标签，**没有 `couponId` / `couponCode`**，喂不进 `calculate-price` 的券入参。要"带券算价"必须先调 `query-store-coupons`（顺带拿 `products[].productCode` 知道券适用哪些商品）。另注意：带券算价返回的 `discount` 是相对**券的原价口径**算的（实测冰美式中杯 `productOriginalPrice 2300` vs 菜单促销价 1190，`discount 1310`），**别把 `discount` 直接当"省了多少"**。

## 点餐主链路

| Tool name | 用途 | 写操作 |
|---|---|---|
| `query-nearby-stores` | 按地址查询附近可用门店（返回 `businessStatus` 与营业时段） | 否 |
| `query-meals` | 查询当前门店可售卖餐品菜单（分类 / 编码 / 标签 / 优惠价） | 否 |
| `query-meal-detail` | 按餐品编码查详情、套餐组成、可替换/换购选项、**特调项（`modification`）** | 否 |
| `calculate-price` | 按选购商品列表（可含优惠券、**特调 `modification`**）计算金额、配送费、打包费、优惠、应付总价 | 否 |
| `create-order` | 创建订单，返回订单详情与支付链接；支持 **`remark` 订单备注（仅 `orderType=2`、≤50 字）** 与 **`modification` 特调** | **是** |
| `query-order` | 查询订单状态 / 内容 / 配送信息 | 否 |
| `order-list` | 查询近期到店 / 外送历史订单（非商城订单） | 否 |
| `cancel-order` | 取消点餐订单 | **是** |
| `list-nutrition-foods` | 餐品营养信息（`energyKcal / protein / fat / carbohydrate / sodium / calcium`；**无入参，一次返回全表约 180 项**） | 否 |
| `query-meal-assistance` | 企业团餐场景下的助餐服务查询（**仅企业团餐 beType=6**） | 否 |
| `query-promotions` | 企业团餐场景下的促销规则查询（满减 / 满折，**仅企业团餐 beType=6**） | 否 |

## 外送地址

| Tool name | 用途 | 写操作 |
|---|---|---|
| `delivery-query-addresses` | 查询用户已创建的配送地址列表 | 否 |
| `delivery-create-address` | 新增配送地址 | **是** |
| `delivery-query-stores` | 按收货地址查询可配送门店（返回 `businessStatus`、营业时段与 `reservationTimeOptions` 时段档） | 否 |

## 积分商城

| Tool name | 用途 | 写操作 |
|---|---|---|
| `mall-points-products` | 麦麦商城商品列表（积分兑换 / 现金购买） | 否 |
| `mall-product-detail` | 商城商品详情（图片、积分、有效期、说明） | 否 |
| `mall-create-order` | 积分兑换商品下单，返回兑换订单号与券码 | **是** |
| `mall-order-list` | 麦麦商城近一年订单列表 | 否 |
| `mall-order-detail` | 商城订单详情（支付积分、支付金额、状态） | 否 |

> **`mall-points-products` 类目 `2>8`（周边产品）换不到联名周边**（2026-10-13 实测）：该类目下仅返回 3 项且**全部已过期**——罗技键鼠套装（8-17~8-21）、Nike Book 2 麦当劳特别联名款（6-05 14:00~16:00）、酸黄瓜吧唧礼盒（8-14）。
>
> ⚠️ **但联名周边本身是能买的**（2026-10-16 修正）：周边以**普通可售商品**出现在 `query-meals` 菜单里，`discountType="随单购麦金卡优惠"`、带 `withOrder: {membershipCode:"106", specId:"880"}`。实测编码（深圳金丰城 · 外送）：`521954` 联名复古棒球帽（¥39.90 / 麦金卡价 ¥34.90）、`521955` 联名徽章、`521743` 薯条脆卜卜毛绒周边（¥29.90 / ¥24.90）。走 `query-meals` → `calculate-price` → `create-order` 主链路即可，**不需要积分商城**。加购 `521234 麦金卡体验卡（15 天）¥9.90` 后周边降 ¥5、配送费与打包费归零，**几乎总是更省**（实测净省 ¥1.10～¥12.10）。周边**随门店和渠道变化**（同一时刻上海某店到店菜单没有帽/徽章，只有 `521946 旋旋星愿贴 ¥3.9`），**必须先扫再下结论**。

## 2026-10-16 新增实测要点

| 主题 | 实测结论 |
|---|---|
| **`create-order.remark`** | 订单备注，**仅 `orderType=2`（外送）场景可用，最多 50 字**；到店自取（`orderType=1`）**没有备注入口**。免费配件（番茄酱/糖/胡椒粉/空杯/奶球）只能走这里，或到店给话术 |
| **`modification` 特调** | 商品级与套餐子项级都有。key 规则：选中用 `selectedKey`，未选且该组有 `unselectedKey` 的**也必须一并传**（用 `unselectedKey`）；某组 `minValues=maxValues=1` 且仅 1 项时为必选，不能取消。实测可取消酱料（巨无霸 `100116`、麦香鱼 `100126`）、改冰量（标准/去冰/多冰/少冰）、换燕麦奶（`120146`，+¥2）、加奶油球/白砂糖（`120519`/`120520`，price 0） |
| **特调生效性** | `calculate-price` **不回显特调结果**，只能确认不报错 → 不得向用户断言"已生效" |
| **外送菜单编码 ≠ 到店** | 同一餐品两套编码：巨无霸 到店 `1100` / 外送 `901100`；麦香鸡 `1450` / `901450`；麦香鸡套餐 `9900005453` / `9900003041`。**用错编码会导致 `calculate-price` 返回 `price:0`、`productList` 为空**（不是下架）→ 按当前 `orderType` 重新 `query-meals` 取码 |
| **到店单无满减档位** | 实测到店 ¥94 篮子 `discount:0`、无 `enjoyable`。满减主要在外送 / 团餐场景 |
| **`withOrder` 自动带入** | 传 `withOrder: {membershipCode:"106", membershipSpecId:"880"}` 后，`calculate-price` 会自动把 `521234 麦金卡体验卡（15天）` 和归零的配送/打包费写进 `productList`，**照抄返回值，不要自己拼价格** |
| **`list-nutrition-foods` 无入参** | 一次返回全表（约 180 项），字段 `energyKj / energyKcal / protein / fat / carbohydrate / sodium / calcium`。表按**标准份**计，改特调不会自动重算 |
| **无库存 / 无天气接口** | 35 个工具中**没有任何门店库存查询工具，也没有天气工具**。"这家店还有货吗""今天下雨吗"均无法通过 MCP 回答 |
| **深夜到店门店大面积歇业**（2026-10-28 实测） | 凌晨 00:05 定位杭州西湖区，`query-nearby-stores` 返回 5 家门店**全部 `businessStatus:false`**（最近 2.5km 的灵隐景区餐厅 07:00-19:00；最晚的西湖银泰 06:30-23:00）。同一时刻 `delivery-query-stores`（外送）3 家**全部营业**（00:00-23:59）→ 展示门店必须逐家标 `businessStatus` 与营业时段；全关时如实告知并说明外送是可选通道。另：到店预约时段里的「宵夜」档可能 22:46 就结束（西湖银泰「宵夜 22:14 至 22:46」），**别被档位名误导** |
| **深夜外送专属菜单分类**（2026-10-28 实测） | 外送菜单在深夜出现「**麦麦深夜送**」分类：`9900010321` 四拼小食可乐餐 ¥49.9（原价 ¥81）、`9900005128` 深夜吃鸡堡堡桶 ¥88（原价 ¥140）。另有「多人分享餐」现成 N 人餐（`9900013002` 畅享吃鸡三人餐 ¥86 vs 三人各自点 ¥135.5）→ 多人代录前先提示。**时段空档**：外送 `reservationTimeOptions` 可能不覆盖当前时刻（10-28 夜市档「00:30 至 05:29」，00:00-00:29 为空档）；**立即点单不受影响**（实测可正常算价），仅预约需告知最早可约时段 |
| **社媒链接抓不到** | 实测抓取 `xiaohongshu.com/explore/...` 只返回登录墙（页面仅含"取消 / 允许"）→ 引导用户粘贴文案或发截图 |

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
`now-time-info` → `delivery-query-addresses` → `delivery-query-stores` → 其余同上（`calculate-price` 会含**配送费 + 打包费**；外送菜单价与到店不同，**编码也可能不同**，务必按 `orderType=2` 重新取 `query-meals`）

**隐藏吃法还原（能力五）**：
解析文案/截图 → `query-meals`（当前渠道取码）→ `query-meal-detail`（读 `modification`；含 `rounds` 的商品改用 `roundList` 传选配项）→ `calculate-price`（带特调）→ 用户确认 → `create-order`（外送单带 `remark` 写免费配件）

**穷鬼套餐演算（能力六）**：
`query-store-coupons`（拿 `couponId`/`couponCode` + 适用商品）→ `query-meals` → 枚举 2~5 候选 → **每个候选** `calculate-price` → 以返回值排序 → 输出最低方案（**约束无解就明说，不硬凑**）

**周边抢货雷达（能力七）**：
`query-nearby-stores` / `delivery-query-stores` → 逐门店 `query-meals` 扫周边编码 → 选定后 `calculate-price`（**带与不带 `withOrder` 各算一次**）→ 用户确认 → `create-order`

**参数化营养搭配（能力八）**：
`list-nutrition-foods`（一次拉全表）→ 本地按热量/蛋白约束筛 → `query-meals` 确认在售 → `query-meal-detail`（去酱/换无糖）→ `calculate-price`

**晒单卡片（能力九）**：无 MCP 调用，用已获得的订单与营养数据本地生成 SVG / HTML

**场景感应推送（能力十）**：`now-time-info` →（可选）`campaign-calendar` / `available-coupons` → 本地判断时段与星期 → 组织推送内容

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
