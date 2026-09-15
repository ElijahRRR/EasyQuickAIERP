# 订单生命周期业务模型 v0.1

> 状态：已确认业务基线，后续继续迭代  
> 适用说明：本文首先描述当前团队真实运营流程，用于定义 ERP 应提供的基础业务能力；这些步骤未来可以作为 Workflow Template，但不代表所有团队必须采用相同流程。

---

## 1. 订单生命周期的核心定义

订单进入 ERP 后，第一步只是：

**同步并保存平台订单事实**

然后系统开始判断：

> 这个订单是否正常、是否具备继续履约的条件。

当前团队的一条典型订单主线是：

```text
新订单
→ 订单审核
→ 采购
→ 发货
→ Delivered
→ 售后结束
→ 拉取到对账单
→ 核算准确利润
```

但真实业务中，财务和售后并不总是严格串行。例如：

```text
新订单
→ 订单审核
→ 采购
→ 发货
→ Delivered
→ 拉取到对账单
→ 核算利润
→ 后续发生售后
→ 新的退款 / 调整进入对账
→ 再次核算利润
→ 售后窗口结束
```

因此：

> **订单生命周期不是一条简单直线，而是履约、售后、财务三条时间轴在同一订单上的汇合。**

---

## 2. ERP 不应强制所有团队采用该流程

当前团队把：

- 订单审核；
- 来源采购；
- 物流同步；
- 售后跟踪；
- 对账与利润核算；

作为完整经营链中的重要步骤。

但这些步骤不能成为 ERP 对所有团队的强制流程。

未来 ERP 应首先提供独立基础能力，例如：

- 同步订单；
- 审核订单；
- 创建采购需求；
- 关联采购单；
- 记录发货；
- 同步物流；
- 管理售后；
- 拉取结算 / 对账；
- 计算利润。

团队再通过 Workflow 决定是否采用以及采用什么顺序。

---

## 3. 订单进入 ERP

订单来源于销售平台。

当前核心场景是：

**Walmart Order → ERP**

ERP 首先需要完整保存平台订单事实，而不是在同步阶段直接改变订单业务结论。

同步后进入订单判断阶段。

因此要区分：

### Platform Order Fact

平台真实返回的订单、订单行、金额、地址、状态等数据。

### ERP Order Business State

团队当前认为这个订单应该处于什么业务阶段，例如：

- 待审核；
- 待采购；
- 已采购；
- 已发货；
- 已送达；
- 售后处理中；
- 已结束。

二者不能混为一个状态字段。

---

## 4. 订单与订单行

业务判断、采购和利润核算经常发生在 Order Line 层，而不是整个订单层。

例如一个 Walmart Order 中可以有多个商品行，不同 Order Line 可以：

- 使用不同来源；
- 产生不同采购成本；
- 使用不同物流；
- 出现不同售后；
- 产生不同利润。

因此 ERP 必须同时管理：

**Order**

和：

**Order Line**

订单级状态不能替代订单行级经营事实。

---

## 5. 订单审核的目的

当前订单审核主要回答：

> **这个订单当前是否适合继续采购和履约。**

审核不是商品审核。

商品审核回答：

> 这个 Product 是否适合上架到某个平台。

订单审核回答：

> 这个已经产生的销售订单是否具备正常履约条件。

---

## 6. 订单审核：商品一致性

第一项判断是：

> 当前准备采购的 Source Product 与实际售卖给客户的商品是否一致。

需要避免出现：

- 型号不同；
- 规格不同；
- 数量不同；
- 变体不同；
- 实际商品不是同一产品；
- 来源商品发生变化以后与 Listing 已不一致。

因此订单审核不能只知道：

`Listing → Source`

还需要基于当前来源商品事实再次确认一致性。

---

## 7. 订单审核：经济性

需要判断当前订单继续履约是否会造成不可接受的亏损。

至少需要考虑：

- Walmart 实际销售金额；
- 当前 Source Price；
- Source Shipping；
- 预计采购成本；
- 平台费用或其他已知费用；
- 其他会影响订单利润的成本。

这时计算的仍然是：

> **履约前预估利润。**

不是最终准确利润。

最终利润必须结合后续真实采购、平台结算和售后调整重新计算。

---

## 8. 订单审核：来源库存

需要判断：

> 当前主来源或可用来源是否有足够库存完成订单。

如果主来源缺货，不代表订单必然失败。

因为当前业务允许：

> 为同一个 Order Line 寻找其他同款 Source Product 完成采购。

因此库存判断需要支持：

- 主来源库存；
- 备用来源；
- 新找到的替代来源。

---

## 9. 订单审核：配送方式与物流时效

需要确认来源商品当前配送能力是否满足销售平台要求。

主要关注：

- 来源配送方式；
- 当前客户地址；
- 预计送达时间；
- Walmart 对该订单要求的物流时效。

核心判断：

> 来源商品是否能够在允许时间内送达客户。

---

## 10. 订单审核：钓鱼订单风险

当前团队还会利用客户邮编判断：

> 该订单是否可能属于钓鱼 / 高风险订单。

这是订单风险判断的一部分。

它与：

- 商品是否侵权；
- 商品是否允许销售；

无关。

属于：

> Order-level Risk Signal。

具体判断规则未来属于团队自己的订单审核策略，不应成为 ERP 全局硬编码。

---

## 11. 订单审核结论

当前已经确认审核至少需要综合：

1. Source Product 与 Listing 商品是否一致；
2. 当前价格是否会造成不可接受亏损；
3. Source 是否有足够库存；
4. 配送方式是否可接受；
5. 配送时长是否满足订单要求；
6. 邮编是否存在高风险信号。

具体审核结果枚举和自动处理策略仍待后续确认。

ERP 基础模型只需要保证：

> 审核结果、原因、证据和当时使用的数据能够被保存和追踪。

---

## 12. 采购不是 Order Line 与一个 Source Order 的一对一关系

当前业务明确：

> 一个 Walmart Order Line 可能对应多个 Amazon Purchase / Source Purchase。

原因包括：

- 原产品源缺货；
- 更换新的同款来源；
- 拆采购；
- 多次采购；
- 第一次采购失败后重新采购。

因此不能建模成：

```text
Order Line → Purchase Order
```

固定一对一。

更准确的是：

```text
Order Line
   ↓
一个或多个 Procurement / Purchase
   ↓
每次 Purchase 关联当时实际使用的 Source Product
```

---

## 13. 来源切换发生在订单履约阶段

商品中心可以存在主来源和备用来源。

但真正产生订单以后，采购仍可能根据实时情况改用：

- 当前主来源；
- 已有备用来源；
- 临时发现的新同款来源。

因此必须区分：

### Product Primary Source

商品层当前默认采购来源。

### Actual Order Procurement Source

这一个订单行最终真实使用的采购来源。

最终利润和履约审计必须使用：

> 实际采购来源和实际采购金额。

不能使用商品主来源当前价格反推历史采购成本。

---

## 14. 采购需要保留真实交易事实

每一次采购至少需要能够追踪：

- 对应哪个 Order Line；
- Source Platform；
- Source Product；
- Source Purchase / Order ID；
- Purchase Quantity；
- 实际采购价格；
- 实际运费；
- 实际支付金额；
- 采购时间；
- 采购状态；
- 是否取消；
- 是否退款；
- 谁执行了采购。

这些数据是后续准确利润计算的重要基础。

---

## 15. 采购前取消是一个重要业务分支

日常售后中存在：

> 客户在采购之前取消 Walmart 订单。

此时最重要的动作不是退款计算，而是：

> **立即阻止采购继续发生。**

流程：

```text
Order Cancellation Detected
→ 检查是否已经采购
```

如果：

### 尚未采购

则：

**Cancel Procurement / Do Not Purchase**

如果：

### 已经采购

则进入另一套采购取消 / 退货 / 损失处理流程。

因此：

> Cancellation 与 Procurement 之间存在明确的竞态关系。

订单系统必须知道订单当前是否已经产生真实采购动作。

---

## 16. 物流履约模式

当前主要业务模式是：

> Amazon / Source → Walmart Customer 直接配送。

商品不以常规模式先进入自有仓库再重新发货。

但在向 Walmart 提交发货信息和后续追踪时，会使用第三方物流能力，并同步相应物流轨迹。

因此 ERP 需要区分：

### Physical Fulfillment Source

真正负责商品发出的来源 / 供应端。

### Customer-facing Tracking / Logistics

向销售平台和客户展示、持续跟踪的物流轨迹。

二者不能被简单假设为完全同一个系统。

---

## 17. 发货阶段

采购完成以后，订单进入履约和物流阶段。

至少需要管理：

- 是否已经发货；
- Tracking Number；
- Carrier / Logistics Provider；
- 发货时间；
- 预计送达时间；
- 当前物流状态；
- 是否成功同步到 Walmart；
- Walmart 当前记录的 Shipment 状态。

---

## 18. 物流同步不是一次性动作

发货后需要持续获取并同步物流轨迹。

因为后续售后中非常重要的两类问题：

- Delay；
- Lost Package；

都依赖真实物流状态。

因此：

> Tracking 是订单生命周期中的持续数据源，而不是发货时保存一个单号就结束。

---

## 19. Delivered 不是订单业务结束

当物流显示：

**Delivered**

只能说明：

> 履约配送阶段基本完成。

但订单仍可能发生：

- Refund；
- Return；
- Chargeback；
- Customer Complaint；
- 其他售后调整。

因此：

> Delivered 不是订单终态。

---

## 20. 当前主要售后类型

业务中可能存在：

- Cancellation；
- Refund；
- Return；
- Chargeback；
- Customer Complaint；
- Lost Package；
- Delay。

当前日常最主要处理：

1. Delay；
2. Lost Package；
3. Cancellation；
4. Refund。

---

## 21. 延迟订单

Delay 的处理通常需要：

- 查询最新 Tracking；
- 判断是否仍在正常运输；
- 判断预计到达时间；
- 判断是否需要主动处理；
- 后续反复查询物流变化。

因此人工成本高的原因不是一次判断，而是：

> **同一个异常订单需要持续回访。**

这类业务天然需要 ERP 提供：

- 异常队列；
- 下次检查时间；
- 最新物流状态；
- 历史检查记录。

是否自动检查属于后续 Workflow 阶段。

---

## 22. 丢件订单

Lost Package 同样需要持续确认：

- 是否真的长时间无轨迹；
- 是否 Carrier 已确认异常；
- 是否最终恢复运输；
- 是否需要退款；
- 是否需要其他补救措施。

因此丢件不能只是一个永久静态标签。

它应该拥有自己的事件和处理进度。

---

## 23. Refund

退款会直接改变订单真实利润。

因此售后退款必须与：

- Order；
- Order Line；
- Walmart Settlement / Recon；

建立关系。

不能只把退款保存成客服备注。

---

## 24. 财务结算与售后可以交叉发生

这是订单生命周期的重要特征。

可能出现：

```text
Delivered
→ Walmart 第一次结算
→ 计算订单利润
→ 后续客户退款
→ Walmart 后续账期出现 Refund / Adjustment
→ 重新计算订单利润
```

所以：

> 对账完成一次，不等于订单财务永远结束。

---

## 25. 利润需要区分“预估”和“准确”

订单至少存在两个利润概念。

### Estimated Profit

在采购前或采购时用于判断是否值得履约。

可能根据：

- Sale Amount；
- 当前 Source Price；
- Shipping；
- 预计平台费用；
- 其他预计成本。

这是订单审核依据。

### Actual / Reconciled Profit

根据真实发生的数据计算：

- 平台最终收入；
- 平台费用；
- 实际采购金额；
- 实际 Source Shipping；
- Refund；
- Return；
- Chargeback；
- Adjustment；
- 其他真实资金变化。

这才是用于经营核算的准确利润。

---

## 26. 准确利润可能被多次重新计算

因为后续可能继续出现：

- Refund；
- Return；
- Adjustment；
- Chargeback；
- 其他平台账务修正。

所以 ERP 不能把：

> “第一次匹配到 Recon 后的 Profit”

当成永远不变的最终数字。

更准确的业务概念是：

```text
Estimated Profit
→ First Reconciled Profit
→ Adjusted Reconciled Profit
→ Final Profit
```

---

## 27. 什么情况下订单才真正结束

当前团队的业务定义：

> **订单必须经过退款 / 退货窗口以后，才可以认为业务结束。**

因此：

**Delivered**

不结束。

**第一次 Settlement / Recon**

也不一定结束。

需要等待售后风险窗口基本关闭。

最终结束时应该能够确认：

- 履约已完成；
- 没有需要继续处理的主要售后；
- 退款 / 退货窗口已结束；
- 已发生的财务调整已经进入核算；
- 订单利润已经形成当前最终值。

---

## 28. 订单业务实际上包含三条子生命周期

### A. Fulfillment Lifecycle

```text
New
→ Audit
→ Procurement
→ Shipment
→ In Transit
→ Delivered
```

### B. After-sales Lifecycle

```text
No Issue
→ Cancellation / Delay / Lost / Refund / Return / Complaint / Chargeback
→ Handling
→ Resolved
→ After-sales Window Closed
```

### C. Financial Lifecycle

```text
Estimated Revenue / Cost
→ Settlement Available
→ Recon Matched
→ Profit Calculated
→ Later Adjustment
→ Profit Recalculated
→ Financially Final
```

一张订单的完整生命周期是三条线共同决定的。

---

## 29. Order Business State 不能简单等于一个线性枚举

例如一张订单可以同时是：

- Fulfillment：Delivered；
- After-sales：Refund Pending；
- Finance：First Recon Completed。

如果系统只保存：

`status = delivered`

就无法表达真实业务。

因此阶段 0 已确认：

> **履约状态、售后状态、财务状态必须在业务概念上分开。**

具体技术实现方式留到阶段 1 设计。

---

## 30. 多店铺带来的售后运营压力

当前人工成本高的重要原因不是单个订单复杂，而是：

> 同时管理大量 Store 后，需要持续关注多个来源的信息。

尤其包括：

- 多个店铺邮件；
- 客户消息；
- 平台通知；
- Delay；
- Lost Package；
- Refund；
- Cancellation。

因此未来 ERP 的售后能力不能仅仅是：

> Order Detail 页面放一个“售后”Tab。

它还需要从运营角度形成：

> **跨店铺统一异常工作台。**

具体界面属于后续阶段，此处只确认业务需求。

---

## 31. 邮件 / 消息是售后事件来源之一

当前多个店铺的售后处理需要大量查看邮件。

因此未来 ERP 需要具备一个概念：

> After-sales Event 可以来自不同外部渠道。

例如：

- Platform API；
- Email；
- Customer Message；
- Logistics；
- Manual Entry。

这些最终应该尽量关联回：

- Team；
- Store；
- Order；
- Order Line。

但具体邮件接入方式不属于阶段 0 技术设计。

---

## 32. 采购订单请款是独立业务问题

当前订单相关人工成本中还包括：

> 采购订单请款。

这说明 Procurement 不仅是：

> “在哪个平台买了东西”。

还涉及内部资金 / 报销 / 请款流程。

目前尚未展开具体业务规则，因此阶段 0 暂时只记录：

> **采购请款是后续需要单独建模的子流程。**

不能在没有确认真实流程之前直接并入普通订单状态。

---

## 33. 当前订单域最消耗人工的工作

目前主要包括：

### 1. 售后信息收集

多 Store、多邮件、多渠道，信息分散。

### 2. 物流轨迹同步

发货后需要持续同步物流状态。

### 3. Delay / Lost Package 持续追踪

需要反复查询同一订单，而不是一次处理完成。

### 4. Cancellation 与采购协调

尤其采购前取消，需要及时阻止采购。

### 5. Refund 处理

直接影响订单状态和利润。

### 6. 采购订单请款

属于采购后的内部资金流程。

---

## 34. 当前团队订单 Workflow Template

根据已经确认的业务，当前团队未来可以形成类似 Workflow Template：

```text
Sync New Orders
→ Audit Order
→ If Approved: Procure
→ Track Purchase
→ Ship / Sync Tracking
→ Monitor Logistics
→ Delivered
→ Monitor After-sales
→ Pull Settlement / Recon
→ Reconcile Profit
→ Recalculate When Adjustments Arrive
→ Close After Refund / Return Window
```

但：

> 这只是当前团队的模板，不是 ERP 对所有 Team 的强制流程。

---

## 35. 当前已经确认的业务原则

1. 订单进入 ERP 后先同步保存，再进行业务判断。
2. Platform Order Fact 与 ERP Business State 必须区分。
3. Order 与 Order Line 都是核心业务对象。
4. 当前团队的订单审核发生在采购前。
5. 订单审核需要确认 Source Product 与售卖商品一致。
6. 订单审核需要判断预估利润是否可接受。
7. 订单审核需要判断来源库存。
8. 订单审核需要判断配送方式和预计物流时效。
9. 邮编风险属于订单审核的一类团队风险规则。
10. 一个 Order Line 可以对应多个 Source Purchase。
11. 订单履约期间可以更换采购来源。
12. Product Primary Source 与 Actual Procurement Source 必须区分。
13. 历史利润必须使用真实采购事实，而不能用商品当前来源价格反推。
14. 采购前 Cancellation 必须能够阻止采购继续发生。
15. 当前主要履约模式为 Source / Amazon 直接配送至最终客户。
16. 面向 Walmart 的物流轨迹通过第三方物流能力处理和同步。
17. Tracking 是持续数据，而不是仅保存一个 Tracking Number。
18. Delivered 不是订单业务结束。
19. 当前主要售后为 Delay、Lost、Cancellation、Refund。
20. Delay / Lost 的核心特点是需要持续回访物流状态。
21. Refund 会影响订单最终利润。
22. 售后与平台财务结算可以交叉发生。
23. 订单至少需要区分 Estimated Profit 与 Actual / Reconciled Profit。
24. Reconciled Profit 可能因后续 Refund / Adjustment 重新计算。
25. 当前团队以退款 / 退货窗口结束作为订单最终闭环的重要条件。
26. Fulfillment、After-sales、Finance 是三条相对独立的订单子生命周期。
27. 一张订单可以同时处于不同履约、售后和财务状态。
28. 多店铺统一售后信息管理是重要业务需求。
29. 邮件、平台消息、物流状态等都可能成为售后事件来源。
30. 采购订单请款需要作为独立子流程继续建模。
31. 当前团队流程未来可以作为 Workflow Template，而不是 ERP 强制流程。

---

## 36. 当前尚待进一步建模的问题

### 36.1 订单审核的正式结论

需要继续确认：

- 审核通过；
- 拒绝；
- 建议拒绝；
- 待人工 / 待处理；

实际应该有哪些正式业务状态，以及不同结论下一步是什么。

### 36.2 Procurement Lifecycle

需要单独确认：

- 谁创建采购；
- 谁执行采购；
- 采购成功 / 失败如何定义；
- 多次采购如何关联；
- Source Order Cancel / Refund / Return 如何处理；
- 采购请款的真实流程。

### 36.3 Logistics Lifecycle

需要继续确认：

- 第三方物流具体承担什么角色；
- Tracking Number 从哪里产生；
- 哪些物流状态需要映射；
- 什么时候算 Delay；
- 什么时候算 Lost；
- 物流同步失败如何处理。

### 36.4 After-sales Lifecycle

需要继续确认：

- Cancellation；
- Refund；
- Return；
- Chargeback；
- Complaint；
- Delay；
- Lost；

分别如何产生、如何处理以及什么时候关闭。

### 36.5 Profit / Reconciliation

需要继续确认：

- 实际利润包含哪些收入和费用；
- 如何匹配 Walmart Recon；
- 一笔 Settlement 如何映射 Order / Order Line；
- Source Purchase Refund 如何进入成本；
- Walmart Refund / Adjustment 如何重新计算利润；
- 什么时间点才允许标记 Financially Final。

---

## 37. 当前订单生命周期结论

订单域真正管理的不是简单的：

```text
New → Shipped → Delivered
```

而是同时管理：

> **销售订单事实 + 履约 + 采购 + 物流 + 售后 + 资金结算 + 利润。**

其中最重要的业务认知是：

```text
Delivered ≠ Order Complete
First Settlement ≠ Financial Final
First Profit Calculation ≠ Final Profit
```

完整订单必须一直能够追踪到：

> 售后风险窗口结束，并且相关财务变化已经进入最终核算。

这才构成当前团队意义上的订单业务闭环。
