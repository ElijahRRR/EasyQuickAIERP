# 核心对象业务信息需求（第二批）v0.3

> 状态：已确认  
> 所属阶段：阶段 1 — ERP 地基  
> 说明：本文只确认 Sales Order、Order Line、Procurement Task、Purchase 在业务上必须保存什么，以及历史数据如何保留。数据库表结构和字段实现由开发人员负责。

## 1. Sales Order（销售订单）

Sales Order 表示：

> 销售平台产生的一张客户订单。

业务上至少需要长期保存：

- 所属 Team；
- 所属 Store；
- Platform；
- 平台订单标识；
- Customer Order ID / Purchase Order ID 等平台标识；
- 下单时间；
- 当前平台订单状态；
- 预计发货时间；
- 预计送达时间；
- Customer Promise Date 等履约时效信息；
- 订单级金额信息；
- 客户姓名；
- 电话；
- 收货地址；
- 邮编；
- Cancellation 等平台事实；
- First Sync Time；
- Last Sync Time；
- 必要的原始平台订单数据，用于以后排查平台返回与解析问题。

### 1.1 客户与配送信息按下单时快照永久保留

当前确认：

> 客户姓名、电话、地址、邮编等按订单发生时快照永久保留。

不能因为平台后续资料变化，改写历史订单当时的数据。

### 1.2 订单历史责任归属

当前确认：

> Sales Order / Order Line 的运营责任归属，以订单下单时的 Store Assignment 为准。

不是以：

- 审核时；
- 采购时；
- Refund 时；
- Settlement 时；

的当前负责人重新计算。

例如：

```text
5 月下单 → 张三负责
7 月 Store 转给李四
10 月发生 Refund

该订单经营责任仍归张三时期
```

---

## 2. Order Line（订单商品行）

Order Line 是订单中一个具体商品行。

采购、退款、利润等业务通常落在 Order Line 粒度。

业务上至少需要长期保存：

- 所属 Sales Order；
- 平台 Order Line 标识；
- 下单时 SKU；
- Product Name；
- Quantity；
- Product Amount；
- Shipping Amount；
- 下单时平台商品相关数据；
- 当前 Platform Line Status；
- Cancellation Reason；
- 预计发货 / 送达时间；
- Listing Reference（如果可可靠匹配）；
- Product Reference（如果可可靠匹配）；
- Order Audit Result；
- Order Audit Reason / Evidence；
- Pending Reason；
- Pending Resolution / Next Check。

### 2.1 Order Line 自身快照必须独立存在

正式原则：

```text
Order Platform Snapshot = 必须保存
Listing / Product Relation = 可以为空
```

即使某历史订单无法匹配当前 ERP Listing / Product：

> Order / Order Line 仍然必须正常入库并永久存在。

---

## 3. Order Audit History（订单审核历史）

Order Audit 正式结果：

- Pass；
- Reject；
- Pending。

Pending 可以来自：

- Machine / LLM 无法可靠判断商品一致性；
- 等待人工复核；
- Source 暂时缺货；
- 等待 X 天后重新检查；
- 其他暂时不能得出可靠结论的情况。

### 3.1 审核必须保存当时使用的判断依据

当前确认：

> Audit 当时使用的 Source、Price、Stock、Delivery 等信息必须作为审核历史保存。

不能以后读取 Product Source 当前最新值来解释历史审核。

例如审核当时：

```text
ASIN A
Price = $20
Stock = In Stock
ETA = Oct 10
```

即使以后：

```text
Price = $30
Stock = OOS
```

历史 Audit Evidence 仍保持当时事实。

Order Audit Evidence 应尽可能绑定到当时实际观察到的 Source Offer Snapshot，例如：

- ASIN；
- 具体 Variant；
- Seller / Seller ID；
- Price；
- Shipping；
- Stock；
- Fulfillment；
- ETA / Promise Date；
- Ship-to Context；
- Observed Time。

每一次 Order Audit 还必须绑定：

> **当时实际使用的 Team Order Audit Policy / Rule Version。**

这样后续 Team 修改采购限价、等待天数、配送时效等规则后，仍能准确解释历史订单为什么在当时得到 Pass / Reject / Pending。

Seller Blacklist 判断针对当次候选 / 选中 Offer 的实际 Seller。

不能因为 Product Source 仍然是同一个 ASIN，就忽略 Seller 已经变化。

### 3.2 多次审核 / 复查历史全部保留

同一个 Order Line 可以出现：

```text
Audit 1 → Pending
Audit 2 → Pending
Audit 3 → Pass
```

三次记录都保留。

不能只覆盖成最终：

`Pass`

这样未来可以分析：

- 等待了多少次；
- 为什么 Pending；
- 哪一次数据变化后通过；
- 是机器、LLM 还是人工做出的判断。

---

## 4. Procurement Task（采购任务）

Procurement Task 表示：

> 为某个 Sales Order Line 产生的一次采购需求 / 工作任务。

它不是实际 Source Order。

业务上至少需要知道：

- 所属 Order Line；
- Required Quantity；
- Purchased / Covered Quantity；
- Remaining Quantity；
- Created By；
- Assigned By；
- Assigned Procurement User / Team；
- Claimed By；
- 当前是否仍需要处理；
- 是否被退回 Operations；
- Return Reason；
- Created Time；
- Assigned Time；
- Claimed Time；
- Completed / Closed Time；
- Procurement Task Reason。

例如：

```text
Order Line Qty = 3
Procurement Task Required Qty = 3

Purchase A = 1
Purchase B = 2
```

任务完成。

---

## 5. 补发 / 再采购创建新的 Procurement Task

当前确认：

> 因 Lost、Replacement、After-sales 等原因需要再次采购时，创建新的 Procurement Task。

不继续修改已经完成的原采购任务。

例如：

```text
Order Line Qty = 1

PT001
Reason = Initial Fulfillment
└ Purchase A

物流丢件

PT002
Reason = Replacement / Lost
└ Purchase B
```

这样可以清楚区分：

- Initial Fulfillment Cost；
- Replacement Cost；
- 不同采购原因；
- 不同采购责任与处理过程。

同时更利于 Finance 准确计算真实利润。

---

## 6. Purchase（实际采购）

Purchase 表示：

> 实际发生的一次来源采购交易。

业务上至少需要保存：

- 所属 Procurement Task；
- 所属 Sales Order Line；
- Source Platform；
- Product Source Reference（如果来自已有 Source）；
- 实际 ASIN / Product Identifier；
- Parent / Child ASIN Relation（如适用）；
- 实际 Variant / 关键规格；
- 实际 Seller；
- Seller ID；
- 实际 Fulfillment；
- Purchase 时承诺 / 预计 Delivery；
- Source Order ID；
- Purchase Quantity；
- 实际商品单价；
- 实际商品金额；
- 实际 Tax；
- 实际 Shipping Cost；
- 最终 Actual Paid Amount；
- Purchase Time；
- Payment Method；
- Procurement Actor；
- Purchase Status；
- Source Estimated Delivery；
- Cancellation Fact；
- Refund Fact；
- Actual Refund Amount；
- External Procurement Evidence（如适用）。

---

## 7. ERP Purchase 与外部 Source Order ID 分开

ERP Purchase 保持 Procurement Task / Sales Order Line 粒度。

同一个 Amazon / Source Order ID 可以被多个 ERP Purchase 引用。

例如一次 Amazon Checkout：

```text
Amazon Order 114-ABC
├ 商品 A → Walmart Order Line A
└ 商品 B → Walmart Order Line B
```

ERP 可以保存为：

```text
Purchase 001
→ Procurement Task A
→ Order Line A
→ Source Order ID = 114-ABC

Purchase 002
→ Procurement Task B
→ Order Line B
→ Source Order ID = 114-ABC
```

因此：

> Source Order ID 不是 ERP Purchase 的唯一业务身份，开发不得把它设计成“一条外部订单只能对应一条 ERP Purchase”的全局限制。

这样 Order Line 级采购数量、成本和利润能够独立核算。

---

## 8. Purchase 保存真实金额，而不是继续使用预计系数

订单采购前可以使用：

```text
Source Price × Qty × 108%
```

作为 Estimated Procurement Cost。

但 Purchase 真正发生以后：

> 必须保存来源平台实际发生的交易金额。

例如：

```text
Product Amount = $40
Tax = $3.20
Shipping = $0
Actual Paid = $43.20
```

实际交易事实与团队 Accounting Procurement Cost 分开。

信用卡 / Virtual Card / 采购方系数：

> 后续由 Finance 根据 Team Cost Policy 计算。

不能覆盖真实 Source Purchase Amount。

---

## 9. Purchase Snapshot 必须永久保存

Purchase 可以关联当前 Product Source，但历史采购不能依赖 Source 当前数据。

例如：

```text
Purchase Time:
ASIN A
Variant = Black / M / 1 Pack
Seller X
Fulfillment = FBA
Price $22.35
Qty 2
Promise Delivery = Oct 10
```

半年以后 Product Source 变化成：

```text
Seller Y
Price $30
```

历史 Purchase 仍然保留：

```text
Seller X
Price $22.35
```

如果采购来自临时 Source：

> Product Source Reference 可以为空，但 Purchase Snapshot 仍必须完整。

---

## 10. Purchase Cancel / Refund 不覆盖原始交易事实

当前确认：

> Purchase 后续发生 Cancel / Refund 时，保留原始下单金额，并单独记录后续结果。

例如：

```text
Purchase
Original Paid = $30
↓
Seller Cancelled
↓
Refund = $30
```

不能最后只把 Purchase 改写成：

`Cost = 0`

因为这样会丢失真实业务过程。

Finance 可以计算：

```text
Net Procurement Cost
=
Original Paid
- Purchase Refund
+ Other Adjustments
```

但 Purchase History 永远保留：

- 原始支付；
- Cancellation；
- Refund；
- 最终净结果。

---

## 11. 第二批对象关系

```text
Sales Order
↓
1..N Order Line
      ├── Platform Snapshot
      ├── Listing / Product Relation（Optional）
      ├── 0..N Order Audit History
      │
      └── 0..N Procurement Task
              └── 0..N Purchase
```

例如补发：

```text
Order Line
├ PT001 Initial Fulfillment
│   └ Purchase A
│
└ PT002 Replacement
    └ Purchase B
```

---

## 12. 当前确认原则

1. Customer / Shipping Data 按下单时快照永久保留。
2. Order 责任归属按下单时 Store Assignment 固定。
3. Order Line 自身 Platform Snapshot 必须独立存在。
4. Listing / Product 关联可以为空，不阻止订单入库。
5. Order Audit 使用的 Source / Offer Snapshot / Price / Stock / Seller / Fulfillment / Delivery 等依据必须历史化。
6. 每次 Order Audit 必须绑定当时实际使用的 Team Order Audit Policy / Rule Version。
7. Seller Blacklist 判断针对当次实际候选 / 选中的 Seller，而不是把 Seller 固化为 ASIN 身份。
8. 同一 Order Line 的所有 Audit / Recheck 历史全部保留。
9. Replacement / Lost 等再采购建立新的 Procurement Task。
10. Procurement Task 与 Purchase 是不同对象。
11. ERP Purchase 保持 Procurement Task / Order Line 粒度；多个 Purchase 可以共享同一个外部 Source Order ID。
12. Purchase 保存真实交易金额：商品金额、Tax、Shipping、Actual Paid。
13. Purchase Snapshot 永久保存，不随 Product Source 当前值变化。
14. 临时采购 Source 可以没有 Product Source Reference。
15. Purchase Cancel / Refund 不覆盖原始交易事实。
16. Finance 可以计算净采购成本，但不能改写 Purchase 历史。

---

## 13. 下一步

下一批建议确认：

- Platform Shipment；
- Tracking / Logistics Observation；
- Return；
- Refund；
- Settlement Entry；
- Settlement Period / Payout。

仍然只确认业务信息需求与历史保留原则，不设计数据库物理字段。
