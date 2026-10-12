# MCP 接入说明（MCP_INTEGRATION）

本文档说明「麦麦饭搭子」实际使用的麦当劳 MCP Server、Tool 清单、调用流程与业务价值。**所有示例数据均为真实调用返回**（截自 2026-10-11 的实跑记录，见 [docs/demo-真实输出.md](docs/demo-真实输出.md)）。

## 1. 使用的 MCP Server

| 项目 | 值 |
|---|---|
| 服务名称 | 麦当劳中国 MCP 服务（mcd-mcp） |
| 接入地址 | `https://mcp.mcd.cn/mcp-servers/mcd-mcp` |
| 传输协议 | Streamable HTTP（JSON-RPC 2.0），支持 SSE 响应 |
| 鉴权方式 | HTTP Header：`Authorization: Bearer <MCP_TOKEN>` |
| 协议版本 | 2025-06-18 |
| Token 申请 | https://open.mcd.cn/mcp（登录 → 控制台 → 激活） |
| 会话模式 | 无状态（服务端未返回 Mcp-Session-Id，每次请求独立） |
| 频控 | 每 Token 每分钟 600 次请求 |

## 2. Tool 全景与本项目选型

服务端实际开放 **35 个工具**（远超公开资料中流传的 5 个）。本项目按「决策型助手」定位，选用其中 11 个：

| 分类 | 工具 | 本项目用途 |
|---|---|---|
| 时间 | `now-time-info` | 判断饭点、计算券到期紧迫度 |
| 活动 | `campaign-calendar` | 营销活动日历（支持 `specifiedDate`） |
| 券 | `query-my-coupons` | 卡包券资产与有效期 |
| 券 | `available-coupons` | 麦麦省可领券列表 |
| 券 | `auto-bind-coupons` | 一键领取全部可领券（需确认） |
| 券 | `query-store-coupons` | 指定门店+订单类型可用券 |
| 营养 | `list-nutrition-foods` | 160+ 款餐品的热量/蛋白/脂肪/碳水/钠/钙 |
| 门店 | `query-nearby-stores` | 可点餐门店与 storeCode |
| 餐品 | `query-meals` / `query-meal-detail` | 门店餐品列表与详情 |
| 价格 | `calculate-price` | 含优惠价格计算（省钱承诺的验证依据） |
| 下单 | `create-order` | 下单（需用户确认） |

> 未使用：派对预约、团餐促销、积分商城、抽奖、满意度问卷、订单历史等（与本项目定位无关；本项目不做任何未经确认的下单动作）。

## 3. 核心调用流程：饭点推荐（多工具编排）

```
[触发] "今天麦麦怎么吃" 或 WorkBuddy 饭点自动化(11:00 / 17:00)
   │
   ├─① now-time-info ─────► 2026-10-11 19:36:17 (SUNDAY)
   │                         → 时段判定：夜宵窗口，下一个饭点=明日早餐
   │
   ├─② campaign-calendar ─► 当日活动：
   │                         · 超值 9.9 元早餐两件套（10/8–10/21）
   │                         · 韩式风味蘸酱上新
   │                         · 麦当劳 × PEACEMINUSONE 联名…
   │
   ├─③ query-my-coupons ──► 卡包 2 张：
   │                         · 新人专享-中杯美式 7 折（¥13.3，到店，有效期至 11/9）
   │                         · 新人专享-中杯奶铁 7 折（¥14.7，到店）
   │
   ├─④ list-nutrition-foods► 160 款餐品营养数据（可按热量/蛋白约束筛选）
   │
   ▼
[决策引擎]
   - 防浪费优先：临期券优先安排
   - 叠加最优：券 × 当日活动组合
   - 营养约束：如用户给了热量/蛋白目标，在营养库内求解
   │
   ▼
[输出] 一句话决策 + 理由 + 预计节省 + 临期提醒
```

## 4. 差异化流程：热量预算搭餐

```
用户："500 大卡以内高蛋白吃什么"
   │
   ├─ list-nutrition-foods（160 款，字段 energyKcal/protein/fat/carbohydrate/sodium/calcium）
   ├─ 约束求解：kcal ≤ 500，按 protein 降序
   ├─ 可选深化：query-nearby-stores → query-meals → calculate-price
   ▼
输出：Top5 营养对比表 + 一句"在预算内最扛饿"的决策
```

实测（2026-10-11）：600 kcal 预算内高蛋白 Top1 = **果然多肉（478 kcal / 36g 蛋白）**，其后为绝代双翅（469/32）、双层猪柳蛋麦满分（513/29）。

## 5. 差异化流程：算真账（可验证省钱）

```
query-nearby-stores(beType=1) → storeCode
   → query-meals(storeCode, orderType, beType) → 选品
   → calculate-price(storeCode, beType, orderType, items) → 含优惠实付价
   → 输出明细与实付金额
```

## 6. 业务价值

1. **决策成本归零**：用户不用在 App 里翻活动、翻券，饭点一句话得到答案。
2. **减少券浪费**：临期券前置提醒 + 围绕临期券优化点单，把"囤了不用"变成"顿顿吃在折扣上"。
3. **开拓营养维度**：把麦当劳的营养数据变成"控卡也能吃麦麦"的决策引擎，覆盖减脂/健身人群这一尚未被充分服务的需求。
4. **活动曝光增量**：营销活动通过饭点推送触达，提升活动参与度。
5. **MCP 编排示范**：展示了用单一 MCP Server 的多工具做**多源融合决策**的完整范式（时间 × 活动 × 券 × 营养 × 价格），可作为其他 MCP 开发者的参考样例。

## 7. 安全与合规

- Token 仅存于本地客户端配置；仓库内所有配置仅含 `YOUR_MCP_TOKEN` 占位符（见 [mcp-config.example.json](mcp-config.example.json)）。
- 不做任何未经用户确认的写操作（领券、下单）。
- 不缓存、不上传用户账户数据；所有查询即时执行、即时呈现。
- 遵守《麦当劳 MCP 服务规则》与频控限制。
