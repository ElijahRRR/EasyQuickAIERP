# 核心对象业务信息需求（第三批）v0.1

> 状态：已确认  
> 所属阶段：阶段 1 — ERP 地基  
> 说明：本文确认 Platform Shipment、Tracking、Return、Refund、Settlement Entry、Settlement Period / Payout 的业务信息需求和历史保留原则。数据库物理结构由开发人员负责。

## 1. Platform Shipment（平台发货记录）

Platform Shipment 表示：

> 销售平台认知中的一次发货记录。

业务上至少需要长期知道：

- 所属 Store；
- 所属 Sales Order；
- 所属 Order Line；
- 本次 Shipment 覆盖的 Quantity；
- Carrier；
- Tracking Number；
- Shipment Time；
- 平台当前识别的 Shipment Status；
- 平台当前识别的 Delivered Status；
- 发货动作来源：ERP、Seller Center 人工、第三方系统等；
- 平台 Shipment ID（如存在）；
- Last Sync Time。

一个 Order Line 底层允许关联多个 Shipment / Tracking。

当前团队虽然通常只向 Walmart 上传一条 Tracking，但 ERP 底层不把这一规则硬编码成全局限制。

### 1.1 Tracking 替换历史必须保留

如果已经上传：

```text
Tracking A
```

后来替换为：

```text
Tracking B
```

当前确认：

> Tracking A 的历史必须永久保留，不能无痕覆盖。

这样才能排查：

- 为什么平台曾经识别错误 Tracking；
- 哪个 Actor / Workflow 做了变更；
- 某个物流绩效问题对应的是哪一个历史 Tracking。

---

## 2. Tracking / Logistics Observation（物流追踪）

Tracking 表示：

> ERP 实际观察到的一条物流追踪关系。

可能包括：

- Amazon Source Tracking；
- 第三方物流 Tracking；
- USPS / UPS / FedEx 等 Carrier Tracking；
- Walmart Platform Tracking。

业务上至少需要知道：

- Tracking Number；
- Carrier；
- Tracking 类型；
- 所属 Order Line / Shipment（如果能够确认）；
- Current Tracking Status；
- ETA；
- Delivered Time；
- Last Tracking Update Time；
- ERP Last Query Time；
- Tracking 是否有效；
- Tracking Data Source；
- 当前是否可继续观察。

如果无法继续查询，可以明确表示：

- Unknown；
- Unobservable；
- Partial Observation。

### 2.1 所有真实 Tracking Event History 都保存

当前确认：

> ERP 实际获取到的每一个物流节点都长期保存。

例如：

```text
Oct 1  In Transit
Oct 2  Arrived at Facility
Oct 3  Out for Delivery
Oct 3  Delivered
```

不能因为当前状态已经变为 Delivered，就覆盖或删除前面的物流历史。

这些历史用于：

- Delay 分析；
- Lost 判断；
- 物流时效分析；
- 异常追踪；
- 平台绩效排查。

---

## 3. Return（退货）

Return 表示：

> 平台的一次正式 Return 流程。

业务上至少需要知道：

- Platform Return ID；
- 所属 Sales Order；
- 所属 Order Line；
- SKU；
- Return Quantity；
- Return Reason；
- Return Method；
- Current Return Status；
- Created Time；
- Last Updated Time；
- Return Tracking（如平台提供）；
- 其他平台返回的重要 Return 信息。

同一个 Return：

```text
Initiated
↓
In Progress
↓
Completed
```

始终是同一个 Return。

### 3.1 Return 保持简单

当前确认：

> Return 当前状态必须保存，最终结果必须保存；当前不专门建设完整复杂的 Return 状态变更历史。

ERP 当前关注：

- Return 当前进展；
- 是否完成；
- 最终结果；
- 对 Refund / Finance 的影响。

不把 Return 扩展成完整 CRM Case System。

---

## 4. Refund（退款）

Refund 与 Return 是两个不同对象。

可能存在：

```text
Return + Refund
```

也可能存在：

```text
Refund Only
```

每次实际 Refund 业务上至少需要知道：

- 所属 Sales Order；
- 所属 Order Line；
- Return Reference（如果有关联）；
- Refund Amount；
- Refund Quantity；
- Full / Partial；
- Refund Reason；
- Refund Time；
- Platform Refund Status；
- Refund Trigger Source；
- Actor（如果由 ERP Human / Digital Actor 发起）；
- Platform Refund ID（如存在）。

### 4.1 一个 Order Line 允许 0..N 个 Refund

当前确认：

> 同一个 Order Line 可以保存多次独立 Refund 记录。

例如：

```text
Refund 1 = $10
Refund 2 = $15
```

Finance 可以计算：

```text
Total Refund = $25
```

但两个真实 Refund 事件都永久保留。

不能只在 Order Line 上维护一个最终累计退款数字而丢失实际发生过程。

---

## 5. Settlement Entry（对账明细）

Settlement Entry 表示：

> Marketplace 账单中真实存在的一条资金记录。

例如：

```text
Sale +$50
Commission -$5
Refund -$50
Adjustment -$7.50
```

每条真实账单记录都需要保存。

业务上至少需要知道：

- Store；
- Settlement Period；
- Entry Type；
- Amount；
- Currency；
- Transaction / Posting Date；
- 平台提供的 Order / SKU / Order Line 关联信息；
- ERP Order Line Reference（如果能够可靠匹配）；
- 平台原始账单数据；
- Matching Status。

### 5.1 无法关联 Order Line 的账务记录也必须保存

当前确认：

> 无法关联具体 Order Line 的 Settlement Entry 也全部保存。

例如：

- Store-level Fee；
- Adjustment；
- Period-level Charge；
- 其他无法合理归属于某条 Order Line 的账务记录。

这些数据：

> 不直接参与某个 Order Line 的利润匹配，但仍然参与 Store / Settlement Period 层面的真实资金核对和 Total Payable。

不能因为无法匹配订单就丢弃。

---

## 6. Settlement Period / Payout（账期与回款）

Settlement Period 表示：

> 某个 Store 在 Marketplace 上的一次正式结算周期。

业务上至少需要长期知道：

- Store；
- Period Start；
- Period End；
- Settlement Date；
- Total Payable；
- Currency；
- Platform Settlement Period ID；
- Sync Status；
- Original Report / Source Reference；
- First Retrieved Time；
- Last Retrieved Time。

### 6.1 正式账期作为历史快照长期保留

当前确认：

> 正式 Settlement Period 需要作为历史账期快照长期保留。

不能只维护一个“当前累计回款数字”。

当前正式定义继续保持：

```text
Cumulative Payout
=
Σ 正式 Settlement Period.Total Payable
```

每日 Pending Payout 不是正式账期，不允许通过每日累计得出 Cumulative Payout。

如果平台以后对历史账期有修正，具体数据库如何版本化或保存修订轨迹由开发人员根据平台真实行为设计，但必须保证：

> 历史账期可追踪，不能无痕丢失原始账期事实。

---

## 7. 第三批对象关系

```text
Sales Order / Order Line
│
├ Platform Shipment
│   └ Tracking
│       └ Tracking Event History
│
├ Return
│   └ Refund（如果有关联）
│
├ Refund Only
│
└ Settlement Entry
      ↓
   Current Settled Amount
```

Store 级：

```text
Store
↓
Settlement Period
↓
Total Payable
↓
Cumulative Payout
```

---

## 8. 当前确认原则

1. Platform Shipment 的旧 Tracking 在替换后仍永久保留。
2. 所有真实获取到的 Tracking Event History 都保存。
3. Return 当前只要求保存当前状态与最终结果，不建设复杂完整状态机历史。
4. Return 与 Refund 是不同对象。
5. 一个 Order Line 可以拥有 0..N 个独立 Refund。
6. Refund 事件不能只压缩成最终累计金额。
7. 无法匹配 Order Line 的 Settlement Entry 也必须保存。
8. 无法匹配订单的账务记录仍可能影响 Store / Period Total Payable。
9. 正式 Settlement Period 作为历史快照长期保留。
10. Cumulative Payout 只能基于正式 Settlement Period，不基于每日 Pending Payout 累计。

---

## 9. 核心业务对象信息需求当前覆盖范围

阶段 1 当前已经完成：

### 第一批

- Store；
- Product；
- Product Source；
- Listing。

### 第二批

- Sales Order；
- Order Line；
- Order Audit；
- Procurement Task；
- Purchase。

### 第三批

- Platform Shipment；
- Tracking；
- Return；
- Refund；
- Settlement Entry；
- Settlement Period / Payout。

这些文档定义业务要求，不代替数据库 Schema。

---

## 10. 下一步

下一步建议进入：

> **业务对象关系与开发交接约束**

重点不是继续追加数据库字段，而是把已经确认的对象整理成一张开发可执行的逻辑关系图，并明确：

- 一对一 / 一对多关系；
- 哪些关系允许为空；
- 哪些对象删除后必须保留历史；
- 哪些业务数据只能追加不能覆盖；
- 哪些变化必须产生 Operation / Audit History；
- 哪些数据由平台同步，哪些由 ERP 决策产生。

完成后即可作为开发人员设计 PostgreSQL Schema 的正式上层约束。
