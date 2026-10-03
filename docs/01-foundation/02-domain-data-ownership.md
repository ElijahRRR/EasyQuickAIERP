# 业务模块与数据归属图 v0.1

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
- Primary / Backup Source；
- Product Audit。

Product Source 表示当前来源状态，例如：

- ASIN；
- Seller；
- Price；
- Stock；
- Shipping；
- Delivery；
- Source Product Data。

历史采购事实不回读当前 Product Source 作为历史成本。

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

正式优先级：

```text
Team Whitelist
>
Team Private Blacklist
>
System Public Blacklist
```

Risk 模块只提供风险判断，不直接修改 Product、Listing、Purchase。

---

## 6. Listing 模块

Listing 永久关联其来源 Product，但保存自己独立的最终发布资料。

负责：

- Listing；
- Final Listing Values；
- SKU / UPC / GTIN；
- Price / Inventory；
- Platform Resource References；
- Submission；
- Platform Listing State；
- Platform Error；
- Repair / Retry History。

Product Source 数据变化不能自动改写已发布 Listing。

---

## 7. Order 模块

负责：

- Sales Order；
- Order Line；
- Order Audit。

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

无论是否关联已有 Product Source，Purchase 都必须保存当时真实采购数据，例如：

- ASIN / Product Identifier；
- Seller；
- Quantity；
- Actual Price；
- Shipping Cost；
- Purchase Time；
- Source Order ID。

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
└ Product Audit

风险
├ System Public Blacklist
├ Team Private Blacklist
└ Team Whitelist

Listing
├ Listing
├ Final Listing Data
├ Submission
└ Platform Error

订单
├ Sales Order
├ Order Line
└ Order Audit

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
2. Product 与 Product Source 属于同一商品模块。
3. Listing 保存自己独立的最终发布资料，并持续关联 Product。
4. Order Line 保存平台下单时快照；即使 Listing 删除或无法匹配，订单仍独立存在。
5. Purchase 保存采购当时的真实快照，不能依赖 Product Source 当前数据恢复历史。
6. 每个模块只直接维护自己拥有的数据。
7. 利润统一由 Finance 模块计算。
8. 模块之间通过正式业务动作或事实通知协作，不直接修改对方数据。
9. Platform Adapter 统一承接外部 Marketplace API。
10. 当前模块边界不等于微服务边界，也不要求独立数据库。

---

## 15. 下一步

下一步进入：

> 核心对象与字段需求设计

这里先确认“业务上必须保存什么信息”，再由开发人员设计：

- 数据库表；
- 字段名称；
- 数据类型；
- 索引；
- 外键；
- 唯一约束；
- JSON / 关系型拆分；
- 迁移策略。

业务字段需求与物理数据库字段设计必须分层确认。
