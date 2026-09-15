# 对账与利润生命周期业务模型 v0.1

> 状态：已确认  
> 所属阶段：ERP 阶段 0 — 业务建模

## 1. 财务模块的核心定位

Reconciliation & Profit 模块主要解决三个问题：

1. 当前订单预计赚多少钱；
2. Walmart 实际已经为这条订单记账多少钱；
3. 截至目前，这条订单的准确利润是多少。

因此需要明确区分：

- **Estimated Profit：预计利润**；
- **Current Reconciled Profit：当前准确利润**；
- **Payout：店铺实际回款**。

三者不是同一个概念。

---

## 2. 最小利润核算粒度

利润核算的最小业务粒度为：

> **Sales Order Line**

原因是：

- 同一个 Order 可以包含多个商品；
- 不同 Order Line 的采购成本不同；
- 一个 Order Line 可以对应多个 Purchase；
- Refund / Adjustment 可能作用到具体 Order Line；
- Walmart 对账数据可以关联订单行。

然后向上聚合：

```text
Order Line
↓
Order
↓
Store
↓
Team
```

Date / Week / Month 等只是对任意层级结果进行时间维度聚合。

---

## 3. 订单未采购时的预计利润

当前团队在订单产生、尚未实际采购时采用：

```text
Estimated Profit
=
Sales Amount
- Estimated Procurement Cost
- Estimated Platform Commission
- Logistics Cost
```

其中：

```text
Estimated Procurement Cost
=
Product Source Unit Price × Quantity × 108%
```

108% 当前用于包含预计采购税费。

---

## 4. 平台佣金预计值

当前团队在账单尚未出来以前：

> 按销售金额约 10% 预计 Walmart Commission。

即：

```text
Estimated Platform Commission = Sales Amount × 10%
```

这是当前团队的 Estimated Cost Policy，不是 ERP 全局固定值。

其他 Team / Platform / Category 可以采用不同规则。

---

## 5. 采购发生后的预计利润

实际采购发生以后：

> Estimated Procurement Cost 应被 Actual Procurement Cost 替代。

公式变为：

```text
Estimated Profit
=
Sales Amount
- Actual Procurement Cost
- Estimated Platform Commission
- Logistics Cost
```

实际采购价格变化后，ERP 应立即重新计算预计利润。

---

## 6. 多 Purchase 的实际采购成本

如果一个 Order Line 被拆成多个 Purchase，则同一 Order Line 下所有实际计入履约成本的 Purchase 需要合计。

例如：

```text
Purchase A = $10
Purchase B = $15
Actual Procurement Cost = $25
```

不能只取 Primary Purchase 或最后一次 Purchase。

---

## 7. 采购退款与异常成本

如果 Purchase 后续发生：

- Cancellation；
- Refund；
- Other Purchase Adjustment；

则采购成本应根据实际资金结果继续调整。

因此采购成本表示：

> 截至当前实际发生的净采购成本。

---

## 8. Logistics Cost

第三方物流成本属于订单利润成本。

售后场景需要区分：

### 未发货

如果没有实际产生第三方物流费用，则不计 Logistics Cost。

### 已发货

如果已经实际产生物流费用，即使后续 Refund，该物流成本仍然需要计入订单利润。

因此：

> Refund 不代表所有履约成本同时消失。

---

## 9. 信用卡 / Virtual Card 成本

信用卡、Virtual Card 等支付成本属于真实经营成本。

当前团队并不是对每一笔 Purchase 单独录入 Card Fee，而是根据不同采购方、不同 Store 等因素采用相关系数。

因此 ERP 不强制每个 Purchase 都拥有精确 Card Fee，而应支持：

> **Team Cost Policy / Cost Coefficient**

例如：

```text
Accounting Procurement Cost
=
Actual Purchase Amount × Coefficient
```

具体系数在后续 Team Policy 中定义。

---

## 10. 实际采购金额与核算采购成本可以不同

业务上区分：

### Actual Purchase Amount

实际采购支付金额。

### Accounting Procurement Cost

用于团队利润核算的采购成本。

例如：

```text
Actual Purchase Amount = $100
Coefficient = 1.02
Accounting Procurement Cost = $102
```

这样可以把 Card Cost、Virtual Card Cost、采购方成本等通过团队规则纳入核算。

---

## 11. 账单出来以前都是预计利润

即使已经 Purchase、Shipped 或 Delivered，只要对应 Walmart Settlement / Recon 尚未出现：

> ERP 仍然只能计算 Estimated Profit。

因为 Walmart 最终实际入账金额尚未确认。

---

## 12. 对账以后使用实际入账金额核算利润

一旦 Walmart 对账数据出现：

```text
Current Reconciled Profit
=
Current Settled Amount
- Accounting Procurement Cost
- Logistics Cost
```

此时不再额外减 Estimated 10% Platform Commission。

因为 Walmart 实际的 Commission、Refund、Adjustment、Incentive 等平台费用已经反映在实际入账金额中。

---

## 13. 对账后的核心不是重新估算平台佣金

账单出现以后：

```text
Before Recon:
Sales Amount - Estimated Fees

After Recon:
Actual Settled Amount
```

因此准确利润直接使用平台实际累计入账金额，而不是继续估算平台费用。

---

## 14. 平台费用明细与利润核心值分开

Walmart 对账可能包含：

- Sales；
- Commission；
- Refund；
- Adjustment；
- Incentive；
- Commission Saving；
- Tax；
- 其他 Transaction。

这些明细可以保留用于财务分析、对账解释和异常检查。

但当前团队核心利润计算最关心的是：

> 该 Order Line 实际累计入账了多少钱。

---

## 15. Order Line 可以跨多个账期

同一个 Order Line 可能：

- 第一个账期收到销售款；
- 第二个账期发生 Refund；
- 第三个账期出现 Adjustment。

因此：

> 一个 Order Line 与 Settlement Period 不是一对一关系。

---

## 16. 所有账期原始记录必须保留

每一个 Settlement Period 的原始财务记录都需要保留。

不能因为后续出现 Refund 或 Adjustment 就覆盖之前账期。

正确关系是：

```text
Order Line
   ↓
Settlement Period 1
Settlement Period 2
Settlement Period 3
...
```

---

## 17. 当前累计入账金额

对于一个 Order Line：

```text
Current Settled Amount
=
Σ All Reconciled Settlement Entries
```

即累计截至当前所有已出现账期中属于该 Order Line 的入账结果。

其结果可以：

- 为正；
- 为 0；
- 为负。

---

## 18. 入账金额允许为负数

当前业务必须支持类似：

```text
Current Settled Amount = -$7.50
```

因此不能假设 Sales Order 的 Settled Amount 最低只能为 0。

---

## 19. Current Reconciled Profit

Order Line 的当前准确利润为：

```text
Current Reconciled Profit
=
Current Settled Amount
- Accounting Procurement Cost
- Logistics Cost
```

后续如果出现新的 Settlement：

1. 更新累计 Settled Amount；
2. 重新计算 Current Reconciled Profit。

---

## 20. 不需要复杂的 Final Profit 状态

当前团队真正需要两个核心利润值：

### Estimated Profit

基于当前业务事实计算的预计利润。

### Current Reconciled Profit

基于截至目前已出现的 Walmart Settlement / Recon 数据计算的当前准确利润。

如果后续又有 Refund、Adjustment 或新 Settlement：

> 继续更新 Current Reconciled Profit。

阶段 0 不建立复杂 Final Profit State Machine。

---

## 21. 售后对预计利润的影响

售后产生以后，ERP 应立即根据当前已知事实重新计算 Estimated Profit。

但不能把售后 API 返回的退款金额直接当成最终 Walmart Settlement Amount。

---

## 22. 售后需要区分是否已产生物流成本

发生 Full Refund 时：

### 尚未发货

可能不存在 Logistics Cost。

### 已经发货

Logistics Cost 已经发生，即使 Revenue 被退款，也继续保留物流成本。

---

## 23. 售后后的准确结果等待后续账期

```text
Return / Refund
↓
ERP Recalculates Estimated Profit
↓
等待下一次 Walmart Settlement
↓
Settlement Arrives
↓
Update Current Settled Amount
↓
Recalculate Current Reconciled Profit
```

因为实际 Walmart 最终处理可能导致入账金额为 0，也可能为负数。

---

## 24. Estimated Profit 是动态值

Estimated Profit 并不是订单创建时只计算一次。

例如：

```text
Order Created
↓
使用 Source Price
↓
Estimated Profit V1
```

实际采购以后：

```text
Actual Purchase Cost
↓
Estimated Profit V2
```

发生售后后：

```text
Refund / Logistics Cost
↓
Estimated Profit V3
```

直到新的 Settlement 到达。

---

## 25. Current Reconciled Profit 也是动态值

Current Reconciled Profit 不是一次关账后永远不变。

例如：

```text
Period 1 → +$20
Period 2 Refund → -$25
Period 3 Adjustment → -$32.50
```

ERP 始终显示：

> 截至当前所有已知账单的准确利润。

---

## 26. Estimated 与 Reconciled 可以同时存在

即使某 Order Line 已经出现第一期 Settlement，后续售后可能已经发生，但新的财务 Adjustment 还没进入账单。

因此可以同时存在：

- Current Reconciled Profit；
- Current Estimated Profit。

例如：

```text
当前已对账利润 = +$20
今天发生 Full Refund，但尚未进入 Recon
预计利润 = -$30
```

两个值都具有业务意义。

---

## 27. Reconciled Profit 有明确的数据截止范围

Current Reconciled Profit 表达的是：

> **截至当前已同步 Settlement 数据的准确利润。**

不是永久最终利润。

后续有新账期就继续更新。

---

## 28. Payout 与订单利润完全分开

Payout 回答：

> Walmart 实际给这个 Store 结算了多少钱。

Profit 回答：

> 一张 Order / Order Line 给团队贡献多少经营利润。

二者不能混用。

---

## 29. 当前团队的累计回款定义

继续沿用：

```text
Cumulative Payout
=
Σ Settlement Period Total Payable
```

即：

> Walmart 每一个正式 Settlement Period 的 Total Payable 累计。

---

## 30. 回款粒度

回款主要属于：

```text
Store + Settlement Period
```

而不是 Order Line。

因此不强行把 Store Total Payable 精确拆成每一张订单的“回款”。

Order Line 负责 Settlement / Profit，Store 负责 Period Payout。

---

## 31. Pending Payout 不是累计回款

每日 Pending Payout 是某一时点的平台待付款余额快照。

同一笔资金可能连续多天出现，因此不能按天累加作为累计回款。

---

## 32. 财务层级关系

```text
Order Line
├── Sales Amount
├── Estimated Procurement Cost
├── Actual Purchase Cost
├── Accounting Procurement Cost
├── Logistics Cost
├── Estimated Commission
├── Estimated Profit
│
└── 0..N Settlement Entries
        ↓
   Current Settled Amount
        ↓
   Current Reconciled Profit
```

与此同时：

```text
Store
↓
0..N Settlement Periods
↓
Total Payable
↓
Cumulative Payout
```

---

## 33. 当前需要进入订单利润的团队成本

目前明确需要进入订单利润核算的主要成本包括：

1. 商品采购成本；
2. 第三方物流费用；
3. 信用卡 / Virtual Card 等采购资金成本。

其中信用卡 / Virtual Card 成本当前允许通过不同采购方、不同 Store 对应的 Cost Coefficient 计算。

---

## 34. 其他成本不要提前强行纳入

例如：

- 人工成本；
- 团队固定成本；
- 软件订阅；
- 办公室成本；
- 其他企业管理费用。

目前没有确认需要进入每一个 Order Line 的利润。

阶段 0 不强行分摊这些费用；以后如有需要，可以另外建设 Store / Team P&L。

---

## 35. 外部采购团队费用

如果外部采购团队最终收费已经体现在：

- Actual Procurement Cost；
- Procurement Settlement；
- Cost Coefficient；

则不应该再次重复扣除。

因此未来成本模型遵守：

> 同一笔真实成本只核算一次。

---

## 36. 当前团队利润计算主流程

订单创建：

```text
Sales Amount
-
Source Price × Qty × 108%
-
Estimated Commission 10%
-
Logistics Cost
↓
Estimated Profit
```

采购以后：

```text
Actual Purchase Cost
替换 Estimated Procurement Cost
↓
Recalculate Estimated Profit
```

产生售后：

```text
Refund / Return Fact
+
是否已经产生 Logistics Cost
↓
Recalculate Estimated Profit
```

Walmart 账单出来：

```text
Settlement Entry
↓
累计 Order Line Settled Amount
↓
Current Reconciled Profit
=
Settled Amount
- Procurement Cost
- Logistics Cost
```

未来新账期继续更新该结果。

---

## 37. 当前确认的财务业务原则

1. 利润核算最小粒度是 Order Line；
2. Order Line 可以向上汇总到 Order、Store、Team；
3. 时间只是聚合维度，不是单独利润实体；
4. 未采购时使用预计采购成本；
5. 当前团队预计采购成本为 Source Price × Qty × 108%；
6. 未出账单前平台佣金当前按约 10% 估算；
7. 实际采购后立即用真实采购成本替换预计采购成本；
8. 多个 Purchase 的有效成本合并到同一 Order Line；
9. 第三方物流费用进入订单利润；
10. 售后时物流费用是否保留取决于是否实际已经产生；
11. 信用卡 / Virtual Card 成本允许通过 Team Cost Coefficient 核算；
12. Actual Purchase Amount 与 Accounting Procurement Cost 可以不同；
13. Walmart 对账出现后，不再使用预计佣金计算准确利润；
14. 准确利润使用平台实际累计入账金额；
15. 一个 Order Line 可以跨多个 Settlement Period；
16. 各账期原始记录全部保留；
17. Order Line Settled Amount 是所有相关账期金额的累计；
18. Settled Amount 可以为 0，也可以为负数；
19. Current Reconciled Profit 会随着新账期持续变化；
20. 不需要复杂的 Final Profit 状态；
21. Estimated Profit 也会随着 Purchase / Refund 等业务事实持续变化；
22. Estimated Profit 与 Current Reconciled Profit 可以同时存在；
23. 售后产生后先更新 Estimated Profit；
24. 售后最终准确财务影响以后续 Settlement 为准；
25. Payout 与 Order Profit 完全分开；
26. 累计回款按 Settlement Period Total Payable 累计；
27. Pending Payout 不能按日累加；
28. Store Payout 不强制拆分成 Order Line Payout；
29. 同一真实成本不能重复核算。

---

## 38. 对账与利润生命周期建模结论

当前财务模型不需要做成复杂会计系统。

它首先准确回答两组问题。

### 订单经营

```text
这张订单现在预计赚多少钱？
这张订单截至目前实际对账后赚多少钱？
```

对应：

- Estimated Profit；
- Current Reconciled Profit。

### 店铺资金

```text
Walmart 到目前真正结算给这个店铺多少钱？
```

对应：

```text
Settlement Period Total Payable
↓
Cumulative Payout
```

因此核心模型为：

```text
Business Facts
↓
Estimated Profit
↓
Settlement Facts
↓
Current Reconciled Profit
```

以及独立的：

```text
Store
↓
Settlement Period
↓
Total Payable
↓
Cumulative Payout
```

> **利润衡量订单是否赚钱；回款衡量 Walmart 实际向店铺结算了多少钱。两套指标相关，但不能混为一个概念。**
