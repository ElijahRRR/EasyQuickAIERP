# 订单生命周期业务模型 v0.3

> 状态：阶段 0 已确认业务基线  
> 适用说明：本文是 Order Domain 的总模型。Procurement、Logistics、After-sales、Reconciliation & Profit 的细节分别以 05–08 文档为准。

## 1. 核心定义

销售订单进入 ERP 后，第一步只是：

> **同步并保存 Platform Order Fact。**

随后 ERP 才根据 Team Policy / Workflow 对订单进行审核、采购、发货、物流观察、售后和财务处理。

因此：

> Order Lifecycle 不是一条强制线性状态机。

一张订单可以同时存在多个独立业务事实。

---

## 2. Platform Order Fact 与 ERP Decision 分开

### Platform Order Fact

保存平台真实返回的：

- Order ID；
- Order Line；
- SKU / Product；
- Quantity；
- Amount；
- Customer / Shipping Information；
- Platform Status；
- Estimated Ship / Delivery；
- Cancellation / Refund 等平台事实。

### ERP Business Decision

表示团队基于当前业务事实做出的判断，例如：

- Order Audit Result；
- 是否继续采购；
- 是否取消；
- 是否需要人工处理。

平台事实与 ERP 决策不能混成一个 `status`。

---

## 3. Order 与 Order Line 都是核心对象

一个 Sales Order 可以包含多个 Order Line。

不同 Order Line 可以：

- 使用不同 Product / Source；
- 产生不同 Purchase；
- 拆分采购数量；
- 产生不同 Shipment；
- 出现不同 Return / Refund；
- 产生不同 Settlement / Profit。

因此：

> 采购、售后和利润核算通常需要落到 Order Line 粒度。

---

## 4. Order Audit 的目的

当前团队的 Order Audit 主要回答：

> **这个订单当前是否具备继续履约的条件。**

Order Audit 与 Product Audit 不同。

Product Audit 判断商品是否适合上架。

Order Audit 判断一个已经产生的销售订单是否适合继续履约。

---

## 5. Order Audit 的主要判断因素

当前已经确认至少包括：

1. Source Product 与已发布商品是否一致；
2. 当前价格 / 预计利润是否可接受；
3. 当前 Source / Alternative Source 是否有库存；
4. 配送方式是否满足要求；
5. 配送速度是否满足订单时效；
6. 邮编等 Order-level Risk Signal。

具体规则属于：

> Team Order Audit Policy。

### 5.1 Order Audit 必须绑定当时证据与规则版本

每一次 Order Audit 都是历史业务判断，不能只保存最终结果。

审核记录至少需要能够还原：

- 当时选择 / 观察到的 Product Source；
- Source Offer Snapshot；
- 实际 Seller；
- Variant；
- Price；
- Stock；
- Fulfillment；
- ETA / Promise Date；
- Ship-to Context；
- Team Order Audit Policy / Rule Version；
- Audit Time；
- Actor / Trigger。

因此：

> 以后 Source 最新价格、Seller 或 Team Policy 发生变化，不能用新数据重解释过去为什么 Pass / Reject / Pending。

同一个 Order Line 的多次 Pending / Recheck / Pass / Reject 全部作为独立历史记录保留。

---

## 6. Order Audit 正式结果

当前正式审核结果为：

### Pass / 通过

当前证据足够，允许订单继续进入团队后续履约流程。

当前团队 Workflow 中：

> Pass 后立即允许生成待采购任务并进入采购。

但其他 Team 可以采用不同 Workflow。

### Reject / 不通过

已经确认存在当前 Team 不接受的履约条件。

后续通常由 Operations：

- Cancel Sales Order；
- 或按 Team Policy 执行其他结束处理。

### Pending / 待处理

当前还不能可靠得出 Pass / Reject。

Pending 不是单纯的“等待人工”。

---

## 7. Pending 可以由不同原因产生

例如：

### Machine / LLM Uncertain

机器审核或 LLM 判断：

> Source Product 与已发布商品可能不一致，但证据不足。

流程可以是：

```text
Pending
→ Human Review / Additional Evidence
→ Pass / Reject
```

### Temporary Stock Unavailable

当前 Source 无货，但 Team 允许等待：

```text
Pending
→ Wait X Days
→ Recheck Stock
├─ Stock Available → Pass
└─ Still OOS Beyond Policy → Reject
```

具体 X 天属于 Team Policy。

因此 Pending 需要能够表达：

- Pending Reason；
- 下一步处理方式；
- 是否需要人工；
- 是否需要未来重新检查。

---

## 8. Audit Result 与后续业务对象分开

Order Audit = Pass：

> 不等于已经 Purchase。

Order Audit = Reject：

> 不等于平台已经 Cancelled。

Order Audit = Pending：

> 不等于订单平台状态异常。

因此 Audit Result 是独立的 ERP Decision Fact。

---

## 9. Order Domain 包含多条独立时间轴

Order 相关业务至少包含：

### Order Audit

```text
Pending / Pass / Reject
```

### Procurement

```text
Procurement Task
→ 0..N Purchase
```

### Platform Shipment / Logistics

```text
Platform Shipment
+
Optional Source / Carrier Observation
```

### After-sales

```text
Return / Refund
```

### Finance

```text
Estimated Profit
+
Settlement / Current Reconciled Profit
```

这些业务相关，但不能压缩成一个总 Order Status。

---

## 10. Procurement 与 Platform Shipment 没有系统级强依赖

当前团队常见业务会同时涉及：

- Walmart Sales Order；
- Amazon / Source Purchase；
- Walmart Platform Shipment。

但阶段 0 已确认：

> Purchase、Source Shipment、Platform Shipment 不存在 ERP 全局固定先后关系。

因此不能硬编码：

```text
Purchase
→ Source Shipped
→ Walmart Shipment
```

Platform Shipment 是否需要等待 Purchase / Source Shipped：

> 由 Team Workflow 决定。

---

## 11. 一个 Order Line 可以对应多个 Purchase

一个 Order Line 可能因为：

- 拆分数量；
- Source 缺货；
- Seller Cancelled；
- 重新采购；
- 更换来源；

产生多个 Purchase。

具体关系与 Quantity Allocation 以：

> [采购生命周期](./05-procurement-lifecycle.md)

为准。

---

## 12. Product Primary Source 与 Actual Procurement Source 分开

商品中心可以维护：

- Primary Source；
- Backup Source。

但订单实际采购时仍然可能使用：

- Primary Source；
- Backup Source；
- 临时找到的新 Source。

最终成本必须来自：

> Actual Purchase Fact。

不能使用 Product 当前 Source Price 反推历史采购成本。

---

## 13. Sales Order Cancellation 与 Purchase Cancellation 分开

Sales Order 可能因为：

- Customer / Walmart Cancel；
- Order Audit Reject；
- Procurement 无法履约后 Operations Cancel；

而被取消。

但：

> Sales Order Cancellation 不强制推导 Purchase Cancellation。

采购端是否取消、继续、退款或其他处理，由采购状态与 Team Policy 决定。

---

## 14. Platform Shipment 与 Logistics 是独立能力

Team 可以：

- 通过 ERP Confirm Shipment；
- 在平台后台人工发货；
- 使用第三方系统发货；
- 只让 ERP 同步 Platform Shipment State；
- 完全不使用 ERP 内置物流能力。

因此：

> ERP 不要求每一张订单都经过固定 Logistics Workflow。

具体以：

> [物流生命周期](./06-logistics-lifecycle.md)

为准。

---

## 15. Delivered 需要区分平台事实与物流观察

Walmart Platform Delivered 表示：

> Walmart 当前认为订单 / Shipment 已经 Delivered。

但 ERP 如果同时观察：

- Source Tracking；
- Carrier Tracking；

这些数据可能仍需要继续追踪。

因此：

> Platform Delivered 不等于所有物流证据都已经闭环。

---

## 16. Delay / Lost 首先属于 Logistics Domain

Delay / Lost 是物流异常判断。

它们可以触发：

- Re-purchase；
- Wait；
- Refund；
- Return；
- 其他运营动作。

但：

> Delay / Lost 本身不需要自动创建 After-sales Case。

正式 Return / Refund 出现后，再进入 After-sales 数据。

---

## 17. After-sales 保持简单

当前 After-sales 核心是：

```text
Pull Platform Return / Refund
→ Link Order / Order Line
→ Show Pending After-sales
→ Operations Handles Refund if Needed
→ Sync Result
```

当前不把：

- Walmart Message；
- Email；
- 客户完整沟通线程；

建成 ERP 核心售后对象。

具体以：

> [售后生命周期](./07-after-sales-lifecycle.md)

为准。

---

## 18. Estimated Profit 与 Current Reconciled Profit 分开

订单未出账单前：

> ERP 根据当前业务事实计算 Estimated Profit。

采购发生后：

> 使用实际采购成本替换预计采购成本。

售后发生后：

> 根据当前已知事实重新估算。

Settlement / Recon 出现后：

> 使用平台实际累计入账金额计算 Current Reconciled Profit。

具体以：

> [对账与利润生命周期](./08-reconciliation-profit-lifecycle.md)

为准。

---

## 19. 不建立复杂 Final Profit 状态机

当前团队只需要长期维护：

- Estimated Profit；
- Current Reconciled Profit。

后续如果新的：

- Refund；
- Adjustment；
- Settlement；

出现：

> 继续重算 Current Reconciled Profit。

因此不再采用：

```text
First Reconciled Profit
→ Adjusted Reconciled Profit
→ Final Profit
```

这样的独立利润状态机。

---

## 20. After-sales Window 仍然是订单闭环的重要概念

订单 Delivered 后仍可能发生：

- Return；
- Refund；
- 其他后续财务调整。

因此：

> Delivered 不是 Order Business Closed。

当前业务仍需要 After-sales Window，用于表达订单已经基本脱离正常退款 / 退货风险期。

阶段 0 不写死具体 N 天。

---

## 21. After-sales Window Closed 不冻结财务历史

即使售后窗口已经结束：

> 如果后续平台仍出现新的 Settlement / Adjustment，ERP 仍然保存并重新计算 Current Reconciled Profit。

因此 Order Operational Closure 与财务数据不可变不是同一件事。

---

## 22. 当前团队 Order Workflow Template

当前团队可以采用类似：

```text
Sync Order
↓
Order Audit
├─ Pending → Human Review / Timed Recheck
├─ Reject  → Operations Resolution / Cancel
└─ Pass    → Procurement Allowed

并行 / 独立能力：
- Platform Shipment
- Logistics Observation
- After-sales Sync
- Settlement / Profit
```

这只是当前 Team Template。

ERP 不把它硬编码为所有 Team 的固定流程。

---

## 23. Order Domain 的业务边界

Order 总模型负责建立和关联：

- Platform Order；
- Order Line；
- Audit Decision；
- Procurement Relation；
- Platform Shipment Relation；
- After-sales Relation；
- Settlement / Profit Relation。

更细业务规则由对应子域负责。

---

## 24. 当前确认原则

1. Order 进入 ERP 后先保存 Platform Fact。
2. Platform Order Fact 与 ERP Decision 分开。
3. Order 与 Order Line 都是核心对象。
4. Order Audit 正式结果为 Pass / Reject / Pending。
5. 每次 Order Audit 必须绑定当时 Evidence 与 Team Order Audit Policy / Rule Version。
6. 同一 Order Line 的多次 Audit / Recheck 历史全部保留。
23. Pending 可以是人工复核，也可以是等待未来数据变化后重新检查。
22. 暂时缺货可以保持 Pending，并按 Team Policy 等待 X 天后重新判断。
23. Order Audit Pass 不等于 Purchase 已发生。
22. Order Audit Reject 不等于平台订单已经 Cancelled。
23. Procurement、Platform Shipment、Logistics、After-sales、Finance 是相对独立业务能力。
22. Purchase 与 Platform Shipment 没有 ERP 全局固定先后关系。
23. 一个 Order Line 可以关联多个 Purchase。
22. Actual Procurement Source 与 Product Primary Source 分开。
23. Sales Order Cancellation 不强制同步 Purchase Cancellation。
22. Team 可以不使用 ERP 内置发货 / 物流能力。
23. Platform Delivered 不等于所有 Logistics Observation 已闭环。
22. Delay / Lost 首先属于 Logistics Domain。
23. 当前售后核心是 Return / Refund 平台数据，不做 CRM。
22. Estimated Profit 与 Current Reconciled Profit 分开。
23. 不建立复杂 Final Profit 状态机。
22. After-sales Window Closed 是订单业务闭环的重要参考，但不冻结后续 Settlement 数据。
23. 当前团队 Order 流程可以作为 Workflow Template，而不是 ERP 全局流程。

---

## 25. 建模结论

Order Domain 不应该被理解成：

```text
New
→ Purchased
→ Shipped
→ Delivered
→ Completed
```

更准确的是：

```text
Sales Order / Order Line
│
├── Platform Order Fact
├── Order Audit Decision
├── Procurement（可选）
├── Platform Shipment / Logistics（可选）
├── Return / Refund
└── Settlement / Profit
```

> **ERP 保存事实，Team Policy 做判断，Workflow 决定这些能力以什么顺序组合。**
