# 业务对象关系与开发交接约束图 v0.1

> 状态：已确认  
> 所属阶段：阶段 1 — ERP 地基  
> 目标：把已经确认的业务模型转换为开发人员设计 PostgreSQL Schema 时不可违反的逻辑关系与数据约束。  
> 注意：本文不是数据库 DDL，不规定具体表名、字段名、数据类型、索引算法或 ORM 实现。

## 1. 关系表示

本文使用：

```text
1 → N   = 一个对象可以关联多个
0..1    = 可以没有，最多一个
0..N    = 可以没有，也可以有多个
```

开发可以采用不同物理表设计，但不能改变这里确认的业务基数与历史语义。

---

## 2. 总体关系图

```text
Team
├── User / Group / Permission
├── Digital Employee
├── Store
├── Product
└── Job

Product
├── 0..N Product Source Relation
│       └── Product Source
│               └── 0..N Source Offer Snapshot
├── 0..N Product Audit History
└── 0..N Listing
        ├── 0..N Listing Validation
        ├── 0..N Submission Attempt
        ├── 0..N Platform Error
        └── 0..N Operation History

Store
├── 0..N Listing
├── 0..N Sales Order
├── 0..N Store Assignment History
├── 0..N Store Configuration History
└── 0..N Settlement Period

Sales Order
└── 1..N Order Line
        ├── 0..N Order Audit History
        ├── 0..N Procurement Task
        │       └── 0..N Purchase
        ├── 0..N Platform Shipment
        │       └── 0..N Tracking Reference
        │               └── 0..N Tracking Event
        ├── 0..N Return
        ├── 0..N Refund
        └── 0..N Settlement Entry

Settlement Period
└── 0..N Settlement Entry
```

Finance 基于这些事实形成 Estimated Profit、Current Reconciled Profit 和 Store-level Payout。

---

## 3. Team / User / Group / Permission

### 3.1 Team 与 User

```text
Team 1 → N User
User → Team = exactly 1
```

当前不支持普通 User 同时属于多个 Team。

### 3.2 User 与 Group

```text
User N ↔ N Group
```

但：

```text
Group Membership ≠ Permission Grant
```

Group 是组织关系，不自动产生权限。

### 3.3 Actor

Human User、Digital Employee、System Sync Service 都是可识别 Actor，但权限语义不同。

用户委托 / Digital Employee 执行真实 Business Operation 时必须检查当前权限；System Sync Service 只在受限范围内同步 Platform Fact。

---

## 4. Team / Store

```text
Team 1 → N Store
Store → Team = exactly 1
```

Store 至少拥有独立的：

- Assignment History；
- Configuration History；
- Platform Resource current data / operation references。

### 4.1 Store Assignment History

当前负责人变化不能覆盖历史责任区间。

历史 Sales Order / Order Line 的运营责任归属按照下单时 Store Assignment 固定。

### 4.2 Store Configuration History

Store Policy / 经营参数修改后需要保留历史生效区间，以支持按当时经营条件分析历史表现。

### 4.3 Store Unbind 不是历史删除

正常业务中的 Store Delete / Unbind 只能解除当前连接或停用。

不得级联删除：

- Listing history；
- Order / Order Line；
- Purchase；
- Shipment / Tracking；
- Return / Refund；
- Settlement；
- Profit / historical attribution；
- Audit History。

---

## 5. Product / Product Source

### 5.1 基本关系

```text
Product 1 → 0..N Product Source Relation
Product Source Relation → Product Source
```

一个 Product 可以人工关联多个 Amazon ASIN。

这些 ASIN 必须被确认能够履约同一个 Product，尤其要确认关键 Variant / Spec，而不是仅凭标题相似。

### 5.2 Product ↔ Source Relation History

绑定 / 解绑 Product Source 都是重要 Product Event。

当前有效关系可以失效，但历史关系永久保留。

至少要能够还原：

- Product；
- Source / ASIN；
- Bound At / By；
- Unbound At / By；
- Reason。

解绑当前关系不得删除历史 Purchase。

---

## 6. Product Source / Source Offer Snapshot

### 6.1 Product Source 身份

Amazon Product Source 按 ASIN 级保持稳定。

Seller 变化不会创建新的 Product Source。

### 6.2 Offer Snapshot

```text
Product Source 1 → 0..N Source Offer Snapshot
```

Source Offer Snapshot 表示某个时间点的采购条件，包括：

- ASIN / Variant；
- Seller / Seller ID；
- Price；
- Shipping；
- Stock；
- Fulfillment；
- ETA / Promise；
- Ship-to Context；
- Observed Time。

因此：

```text
Product Source ≠ Seller Offer
```

开发不得把 Seller / Price / Stock / ETA 当作 ASIN Source 的永久身份。

### 6.3 Amazon Variant

需要能够表达 Parent ASIN、实际可购买 Child ASIN 以及 Color / Size / Pack 等关键 Variant。

Purchase 应尽可能记录实际购买的 Child ASIN / Variant。

---

## 7. Product Audit

```text
Product 1 → 0..N Product Audit
```

Product Audit History 只能追加，不能用一个当前字段覆盖历史。

每次 Audit 必须绑定：

- Product；
- Target Platform；
- Content Version；
- Evidence / Source Version；
- Rule / Policy Version；
- Result；
- Audit Time；
- Actor / Trigger。

Audit Result 只有 Pass / Reject / Pending。

Audit Validity 与 Result 分开。历史 Pass 可以因内容、规则或关键证据变化变为 Stale，但不能把旧 Pass 原地改写成 Reject。

---

## 8. Product / Listing

```text
Product 1 → 0..N Listing
```

数据库不得硬编码一个 Product 只能有一个 Listing。

“一个 Product 只能进入一个 Store”等限制属于 Team Policy，不是全局关系约束。

### 8.1 Listing Draft → Formal Listing

Draft 与正式 Listing 使用同一个 ERP Listing Internal ID。

```text
Draft:
Platform = required
Store = optional

Before Submission:
Platform = required
Store = required
```

Draft 绑定 Store 后不创建新的 Listing Identity。

AI Suggested、Human Edited、Final Listing Values、Validation、Submission 和 Operation History 都沿同一 Listing 连续保留。

---

## 9. Listing History

```text
Listing 1 → 0..N Submission Attempt
Listing 1 → 0..N Platform Error
Listing 1 → 0..N Listing Validation
Listing 1 → 0..N Operation History
```

Submission Attempt 失败后再次提交必须新增历史 Attempt，不能只保留最终 Published。

Listing 当前 Price / Inventory / State 可以更新，但重要变化通过 Operation History / Product Event 保留。

Store 解绑、Listing Retire/Delete 均不得删除历史 Listing。

---

## 10. Store / Sales Order / Order Line

```text
Store 1 → 0..N Sales Order
Sales Order 1 → 1..N Order Line
```

Order / Order Line 的 Platform Snapshot 是订单存在的基础。

客户、地址、邮编、金额、SKU 等历史快照不能依赖未来平台数据重建。

---

## 11. Order Line / Listing / Product

```text
Order Line → Listing = 0..1
Order Line → Product = 0..1
```

Listing / Product 无法匹配时，Order Line 仍必须正常入库。

开发不得将 Listing ID / Product ID 设计成订单入库的业务必填关系。

删除 / Retire Listing 不得删除历史 Order Line。

---

## 12. Order Audit

```text
Order Line 1 → 0..N Order Audit
```

Audit 是可选 Team 能力，因此可以没有审核记录。

每次审核历史必须保存：

- Result；
- Pending Reason / Resolution；
- Source；
- Source Offer Snapshot / Evidence；
- Seller；
- Variant；
- Price；
- Stock；
- Fulfillment；
- ETA / Promise；
- Ship-to Context；
- Team Order Audit Policy / Rule Version；
- Audit Time；
- Actor / Trigger。

多次 Pending / Recheck / Pass / Reject 全部保留。

不能只保留最终结果。

---

## 13. Order Line / Procurement Task

```text
Order Line 1 → 0..N Procurement Task
```

原因包括：

- Initial Fulfillment；
- Lost Replacement；
- After-sales Replacement；
- 其他重新采购。

补发 / 丢件重购需要新的 Procurement Task，不能继续修改已完成的原采购任务。

---

## 14. Procurement Task / Purchase

```text
Procurement Task 1 → 0..N Purchase
```

Procurement Task 是工作任务；Purchase 是实际采购交易。

一个 Qty=3 的 Task 可以由多个 Purchase 覆盖。

### 14.1 ERP Purchase 与外部 Source Order ID

ERP Purchase 保持 Procurement Task / Order Line 粒度。

多个 ERP Purchase 允许共享同一个 Amazon / Source Order ID。

```text
Amazon Order 114-ABC
├ Purchase 001 → Walmart Order Line A
└ Purchase 002 → Walmart Order Line B
```

Source Order ID 不得被设计为 ERP Purchase 的全局唯一业务身份。

---

## 15. Purchase / Product Source / Offer Snapshot

```text
Purchase → Product Source = 0..1
Purchase → Source Offer Snapshot = 0..1
```

临时采购允许没有现有 Product Source，也允许没有完整的采购前 Offer Snapshot。

但 Purchase 自身必须保存完整交易快照，包括：

- 实际 ASIN / Variant；
- Seller；
- Fulfillment；
- Price；
- Quantity；
- Product Amount；
- Tax；
- Shipping；
- Actual Paid；
- Purchase Time；
- Promise / Expected Delivery；
- Source Order ID。

因此 Source / Offer 后续变化或解绑，不能让历史 Purchase 失去可解释性。

---

## 16. Purchase 是历史交易事实

Purchase 原始支付不能被 Cancel / Refund 无痕覆盖。

例如：

```text
Original Paid = 50
Refund = 50
Net Procurement Cost = 0
```

原始 Purchase 与后续 Refund / Adjustment 均保留。

Finance 可以计算净成本，但不能改写原始交易事实。

---

## 17. Source Shipment 与 Platform Shipment 分开

```text
Purchase / Source
→ Source Shipment Observation

Sales Order Line
→ Platform Shipment
```

两者不存在 ERP 全局固定先后依赖。

Source Shipped 不自动等于 Walmart Confirm Shipment；Walmart Shipment 也不自动把 Purchase 改为 Shipped。

如需联动必须由 Team Workflow 调用明确 Business Operation。

---

## 18. Logistics Relations

概念关系：

```text
Purchase 0 → N Source Shipment Observation
Order Line 0 → N Platform Shipment
Shipment 0 → N Tracking Reference
Tracking 1 → 0..N Tracking Event
```

实际获取到的 Tracking Event History 全部保留。

Tracking 被替换时旧 Tracking 仍永久保留。

### 18.1 Tracking Number 不假设全局唯一

开发不得简单把 Tracking Number 作为 ERP 全局唯一键。

需要考虑：

- Carrier；
- Source / Platform context；
- Shipment relation；
- 历史号码重复；
- 一个包裹覆盖多个业务对象。

具体 Unique Constraint 由开发根据上述业务语义设计。

---

## 19. Return / Refund

```text
Order Line 1 → 0..N Return
Order Line 1 → 0..N Refund
Return 1 → 0..N Refund
Refund → Return = 0..1
```

Refund 可以是 Refund-only，因此 Return Reference 不能作为 Refund 必填条件。

同一个 Order Line 可以有多次独立 Refund。

Return 当前不建设复杂全状态历史，但 Current Status 与 Final Result 必须保存。

---

## 20. Settlement

```text
Store 1 → 0..N Settlement Period
Settlement Period 1 → 0..N Settlement Entry
Order Line 1 → 0..N Settlement Entry
Settlement Entry → Order Line = 0..1
```

无法匹配 Order Line 的 Store-level Fee、Adjustment、Period Charge 等 Settlement Entry 仍必须保留。

同一个 Order Line 可以跨多个 Settlement Period 产生 Entry。

不得设计为“一条 Order Line 只有一个 Settlement”。

正式 Cumulative Payout：

```text
Σ Settlement Period.Total Payable
```

不得通过每日 Pending Payout 累计计算。

---

## 21. Profit

Finance 统一计算：

- Estimated Profit；
- Current Reconciled Profit；
- Store / Period Payout。

订单、采购、物流、售后模块提供事实，但不能分别维护不同利润真值。

Current Reconciled Profit 可以随新的 Settlement Entry / Refund / Adjustment 继续变化。

---

## 22. Job 与业务对象

Job 可以关联 Product、Listing、Order、Settlement 等业务 Target，但：

> Job 只是后台执行过程，不是业务记录的父对象。

```text
Job ≠ Business Record
```

删除 / 清理 Job Log 不得删除正式业务数据。

每个 Domain 的实际 Business Detail 仍归各自 Domain 保存。

---

## 23. Job Actor 与执行时授权

Job 至少区分：

```text
Requested By
Execute As
```

User Delegated / Digital Employee Job 在真正执行 Business Operation 时必须重新检查 Execute As 的当前权限。

```text
Queued Permission ≠ Execution Permission
```

长任务执行过程中权限被撤销：

- 已成功完成部分保留；
- 未执行部分停止；
- Job 可记录 Partial Success / Authorization Revoked；
- 旧的一次性任务在后来重新授权后不自动恢复。

System Sync Service 与 User Delegation 分开，只能在授权的同步范围内维护 Platform Fact。

---

## 24. Current Value 与 Historical Fact 必须分开

开发不得为了去重只保存最新值。

例如：

```text
Source Current Offer
≠
Order Audit Historical Offer Evidence

Listing Current Price
≠
Listing Price Operation History

Store Current Operator
≠
Store Assignment History
```

业务上“当前状态”和“历史发生过什么”是两种不同数据。

---

## 25. 原则上只能追加、不能无痕覆盖的数据

至少包括：

- Store Assignment History；
- Store Configuration History；
- Product ↔ Source Relation History；
- Source Offer Snapshot；
- Product Audit History；
- Order Audit History；
- Listing Submission History；
- Listing Operation History；
- Purchase Original Transaction Fact；
- Purchase Cancel / Refund / Adjustment History；
- Tracking Event History；
- Refund Records；
- Settlement Entries；
- Settlement Period Historical Snapshots；
- Audit Log。

如需纠错，应通过 New Version / Correction / Void / Adjustment 等可追踪方式处理，而不是无痕覆盖历史。

---

## 26. 允许维护当前值的数据

例如：

- Store Current Configuration；
- Store Connection Health；
- Product Current Primary Source；
- Source Current Offer Summary；
- Listing Current Price；
- Listing Current Inventory；
- Listing Current Platform State；
- Order Current Platform State；
- Return Current Status；
- Tracking Current Status；
- Job Progress。

重要变化仍应按相应业务规则留下 Operation / Event / Audit History。

---

## 27. 正常业务操作禁止级联删除历史事实

普通 Delete / Unbind / Retire 不得级联删除业务历史。

明确禁止：

```text
Store Unbind → Delete Orders
Listing Delete → Delete Order Lines
Product Source Unbind → Delete Purchases
User Disabled → Delete Actor History
Workflow Delete → Delete Historical Audit
Job Cleanup → Delete Business Records
```

真正的物理删除 / 合规删除 / 数据修复属于专门平台管理流程，不属于普通业务 Delete。

---

## 28. Platform Fact 与 ERP Decision 分开

### Platform Fact

例如：

- Marketplace Order Status；
- Listing Platform State；
- Platform Return / Refund；
- Platform Shipment；
- Settlement Entry；
- Platform Warehouse / Shipping Template；
- Platform Assigned IDs。

这些来自平台真实数据，ERP 不能凭空伪造。

### ERP Decision

例如：

- Product Audit；
- Order Audit；
- Primary Source；
- Store Recommendation；
- Procurement Task；
- Risk Resolution；
- Estimated Profit；
- Workflow Decision。

不能把 Platform Fact 与 ERP Decision 压缩成一个统一 status。

---

## 29. 开发人员可自由决定的内容

在不违反本文业务关系的前提下，开发人员自行决定：

- PostgreSQL 表拆分；
- 字段名称；
- UUID / ULID 等 ID 技术；
- 数据类型；
- 外键实现；
- 索引；
- 唯一约束；
- JSONB / 关系表选择；
- Partition；
- ORM；
- Migration；
- 性能优化；
- Cache；
- Queue / Worker 技术。

业务方不需要逐项批准这些实现细节。

---

## 30. 开发交付时必须能够回答的问题

Schema / Technical Design Review 时，开发应能说明：

1. 每个核心对象的稳定 ERP Internal ID 在哪里；
2. Team / Tenant 隔离如何强制；
3. 哪些关系允许为空；
4. 哪些对象不能因为父对象解绑而级联删除；
5. 历史快照如何保留；
6. Current Value 与 Historical Fact 如何区分；
7. Audit Rule Version / Evidence Version 如何关联；
8. Product Source、Offer Snapshot、Purchase 如何区分；
9. Draft → Formal Listing 如何保持同一 Identity；
10. 一个外部 Amazon Order 如何关联多个 ERP Purchase；
11. Refund-only 如何存在；
12. 无法匹配 Order Line 的 Settlement Entry 如何保留；
13. Job Log 清理为什么不会影响 Business Record；
14. Requested By / Execute As 和执行时权限校验如何实现；
15. Platform Fact 与 ERP Decision 如何避免混在同一状态模型。

---

## 31. 当前核心不变量

```text
Product ≠ Product Source ≠ Source Offer Snapshot ≠ Purchase

Product ≠ Listing

Listing Draft → Formal Listing
保持同一 ERP Listing Identity

Order Platform Snapshot
不依赖 Listing / Product 关联才能存在

Product Audit / Order Audit
历史只能追加，不用最终结果覆盖过去

Queued Permission ≠ Execution Permission

Job ≠ Business Record

Source Shipment ≠ Platform Shipment

Return ≠ Refund

Estimated Profit ≠ Current Reconciled Profit ≠ Payout

Platform Fact ≠ ERP Decision

普通 Delete / Unbind / Retire
≠ 历史数据物理删除
```

---

## 32. 阶段结果

本文确认后：

> EasyQuickAIERP 已具备供开发人员开始 PostgreSQL Schema 设计的业务关系上层约束。

下一步技术工作应由开发侧基于：

- 阶段 0 Business Baseline；
- Foundation Architecture；
- Domain / Data Ownership；
- 三批 Core Object Business Information；
- 本文业务对象关系与开发交接约束；

输出第一版 PostgreSQL Logical / Physical Schema，再进行业务一致性评审。
