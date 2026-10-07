# 业务模块与数据归属图 v0.2

> 状态：已确认  
> 所属阶段：阶段 1 — ERP 地基  
> 目标：明确每类核心业务数据由哪个模块负责保存和修改，以及模块之间如何协作。

## 1. 基础原则

一个核心业务对象应有明确的数据负责人。

其他模块可以引用，但不能绕过负责人直接修改。

跨模块协作只允许两种基本方式：

1. 调用正式业务动作；
2. 通知另一个模块发生了某个业务事实。

技术上以后可以是普通函数、内部事件或消息队列，但业务边界不变。

---

## 2. 系统基础模块

负责整个 ERP 共用能力：

- Team；
- User；
- Group；
- Permission；
- Digital Employee；
- System Sync Service；
- Credential / Secret 安全保存；
- Actor / Audit；
- Async Job。

Store 只引用凭证，不直接散落保存 Client Secret、Token 等敏感值。

---

## 3. Store 模块

负责：

- Store；
- Platform State；
- ERP Enable State；
- Connection Health；
- Store Assignment History；
- Store Policy；
- Platform Resource Reference。

Walmart Platform Warehouse / Shipping Template 属于平台资源引用，不是 ERP 自有实体 Warehouse / WMS。

---

## 4. 商品模块

负责：

- Product；
- Product Data；
- Product Source；
- Product ↔ Source Relation History；
- Source Offer Snapshot；
- Primary / Backup Source；
- Product Audit / Audit History。

需要明确区分：

```text
Product Source
= 来源商品身份，例如 Amazon ASIN

Source Offer Snapshot
= 某个时间点的 Seller、Price、Stock、Fulfillment、Delivery 等采购条件

Purchase
= 最终真实发生的采购交易
```

Seller / Price / Stock / ETA 不构成 ASIN Source 的永久身份。

历史 Order Audit / Purchase 不通过当前 Source 最新值反推过去事实。

---

## 5. Risk 模块

负责：

- System Public Blacklist；
- Team Private Blacklist；
- Team Whitelist。

风险对象主要包括：

- Brand；
- Product / ASIN；
- Amazon Seller。

Blacklist Resolution 内部优先级：

```text
Team Whitelist
>
Team Private Blacklist
>
System Public Blacklist
```

该优先级只用于黑名单判断。

Team Whitelist 不能覆盖确认命中的 TRO、Target Platform 明确禁售等不可覆盖硬规则。

Risk 模块只提供风险事实 / Effective Blacklist Result，不直接修改 Product、Listing、Purchase。

---

## 6. Listing 模块

Listing 永久关联其来源 Product，但保存自己独立的最终发布资料。

负责：

- Listing；
- Final Listing Values；
- SKU / UPC / GTIN；
- Price / Inventory；
- Platform Resource References；
- Listing Validation；
- Submission / Submission History；
- Platform Listing State；
- Platform Error；
- Operation / Repair / Retry History。

Product Source 数据变化不能自动改写已发布 Listing。

---

## 7. Order 模块

负责：

- Sales Order；
- Order Line；
- Order Audit；
- Order Audit Evidence；
- Team Order Audit Rule / Policy Version Reference。

Order Line 必须保存平台下单当时的历史快照，例如：

- SKU；
- Product Name；
- Quantity；
- Sales Amount；
- Shipping / Customer Data；
- Platform Status。

Order Line 可以关联 Listing / Product，但这些关联不是订单入库的前提。

即使历史订单无法匹配 Listing，也必须正常保存。

---

## 8. Procurement 模块

负责：

- Procurement Task；
- Purchase；
- Purchase Snapshot；
- External Procurement Settlement。

Purchase 可以关联 Product Source，但关联是可选的。

无论是否关联已有 Product Source / Source Offer Snapshot，Purchase 都必须保存当时真实采购数据，例如：

- ASIN / Product Identifier；
- 具体 Variant；
- Seller；
- Fulfillment；
- Quantity；
- Actual Price；
- Tax；
- Shipping Cost；
- Actual Paid；
- Promise / Delivery；
- Purchase Time；
- Source Order ID。

多个 ERP Purchase 可以共享同一个外部 Source Order ID；外部订单号不是 ERP Purchase 的永久唯一身份。

历史 Purchase 不随当前 Product Source 变化。

---

## 9. Logistics 模块

负责：

- Platform Shipment；
- Source Shipment Observation；
- Tracking；
- Tracking Event；
- Delay；
- Lost；
- Logistics Exception。

Logistics 只维护物流事实。

例如 Tracking Delivered 后，不直接修改 Purchase / Order 的内部数据，而通过正式业务动作或通知让对应模块处理。

---

## 10. After-sales 模块

当前负责：

- Return；
- Refund；
- Pending After-sales；
- Refund Result。

关联 Sales Order / Order Line。

当前不把 Walmart Message / Email 完整沟通作为 ERP 核心售后数据。

After-sales 保存 Refund Fact，但不直接修改利润。

---

## 11. Finance 模块

所有利润统一由 Finance 计算。

主要输入：

- Order → Sales Amount；
- Procurement → Actual Procurement Cost；
- Logistics → Logistics Cost；
- After-sales → Refund Fact；
- Settlement → Actual Settled Amount。

负责输出：

- Estimated Profit；
- Current Reconciled Profit；
- Settlement Entry；
- Settlement Period；
- Store Total Payable；
- Cumulative Payout。

目标是保证整个 ERP 只有一套利润口径。

---

## 12. Platform Adapter

平台接口层不是独立业务数据模块，而是业务模块访问外部 Marketplace 的统一翻译层。

```text
业务模块
↓
Platform Adapter
↓
Walmart / Amazon / eBay / ...
```

它隔离平台认证、请求格式、状态、参数和 Capability 差异。

---

## 13. 整体数据归属图

```text
系统基础
├ Team / User / Group / Permission
├ Digital Employee / System Sync Service
├ Credential / Secret
├ Actor / Audit
└ Async Job

Store
├ Store
├ Assignment History
├ Store Policy
└ Platform Resource Reference

商品
├ Product
├ Product Source
├ Product-Source Relation History
├ Source Offer Snapshot
└ Product Audit History

风险
├ System Public Blacklist
├ Team Private Blacklist
└ Team Whitelist

Listing
├ Listing
├ Final Listing Data
├ Validation
├ Submission History
├ Operation History
└ Platform Error

订单
├ Sales Order
├ Order Line
└ Order Audit History / Evidence / Rule Version

采购
├ Procurement Task
├ Purchase
├ Purchase Snapshot
└ External Procurement Settlement

物流
├ Platform Shipment
├ Source Shipment Observation
├ Tracking
└ Logistics Exception

售后
├ Return
└ Refund

财务
├ Estimated Profit
├ Settlement Entry
├ Settlement Period
├ Current Reconciled Profit
└ Payout
```

---

## 14. 已确认原则

1. Store API 敏感凭证由系统安全模块专门保存，Store 只引用凭证。
2. Product、Product Source、Source Offer Snapshot 属于同一商品模块，但三者语义必须分开。
3. Product ↔ Source 的绑定 / 解绑历史必须保留。
4. Listing 保存自己独立的最终发布资料，并持续关联 Product。
5. Order Line 保存平台下单时快照；即使 Listing 删除或无法匹配，订单仍独立存在。
6. Product Audit / Order Audit 需要保存历史记录、证据与适用 Rule / Policy Version。
7. Purchase 保存采购当时的真实快照，不能依赖 Product Source 当前数据恢复历史。
8. 多个 ERP Purchase 可以共享同一个外部 Source Order ID。
9. 每个模块只直接维护自己拥有的数据。
10. 利润统一由 Finance 模块计算。
11. 模块之间通过正式业务动作或事实通知协作，不直接修改对方数据。
12. Job 是统一执行基础设施，不替代各业务模块的 Business Record。
13. User Delegated / Digital Employee Job 真正执行时重新检查当前 Authorization。
14. System Sync Service 与 User Delegation 分开，只在受限范围内维护 Platform Fact。
15. Platform Adapter 统一承接外部 Marketplace API。
16. 当前模块边界不等于微服务边界，也不要求独立数据库。

---

## 15. 后续衔接

本文件的数据归属结论已经继续展开到：

- 核心对象业务信息需求；
- 业务对象关系与开发交接约束图。

当前下一步已进入：

> 开发侧 PostgreSQL Logical / Physical Schema 设计与技术评审。

数据库实现由开发人员负责，但不得违反本文件及后续关系约束。
