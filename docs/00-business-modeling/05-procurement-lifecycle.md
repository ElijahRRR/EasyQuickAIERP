# 采购生命周期业务模型 v0.4

> 状态：已确认  
> 所属阶段：ERP 阶段 0 — 业务建模

## 1. 采购业务的定位

采购流程发生在销售订单进入 ERP、完成订单审核并确认需要正常履约之后。

采购的核心目标是：

> 为一个销售订单行找到实际可履约的商品来源，完成购买，并持续追踪该采购商品直到签收。

采购生命周期与销售订单生命周期必须分开管理。

原因是：

- 一条销售订单行可能不采购；
- 一条销售订单行可能产生一次采购；
- 也可能产生多次采购；
- 可以更换来源；
- 可以拆分数量采购；
- 某次采购失败后可以重新采购；
- 销售订单取消后，采购订单也不一定同步取消。

因此：

> **Sales Order Line 与 Purchase 不是一对一关系。**

## 2. 采购流程入口

当前团队的业务流程：

**Sales Order → Order Audit → 审核通过 → 生成待采购任务 → 立即允许采购**

当前团队不存在统一的延迟采购窗口、单笔采购审批、采购前请款或金额超过阈值必须审批。

其他团队未来可以通过 Workflow 自行增加这些步骤。

因此：

> “订单审核通过后立即采购”属于当前团队 Workflow，不是 ERP 全局强制规则。

## 3. 采购任务的参与角色

### System

负责：

- 根据订单状态生成待采购需求；
- 关联销售订单行；
- 提供商品来源和采购参考数据。

### Operations

运营人员负责：

- 分配采购任务；
- 处理采购人员打回的异常任务；
- 在成本、时效和店铺经营之间作最终经营决策；
- 决定是否取消销售订单。

### Procurement

采购团队 / 采购人员负责：

- 领取采购任务；
- 检查主来源；
- 寻找替代来源；
- 实际下单；
- 记录采购信息；
- 追踪采购订单状态。

## 4. 采购任务分配流程

**Order Audit Pass → 生成待采购 → 运营分配 → 采购团队 / 采购人员领取 → 执行采购**

采购人员可以同时服务多个 Store、Sales Order 和 Team 内业务单元。

因此采购任务归属和销售店铺归属不是同一个概念。

## 5. Sales Order Line、Procurement Task 与 Purchase 必须分开

### Sales Order Line

代表：

> 客户购买了什么。

例如：`Walmart Order 123 / SKU A / Qty 3`

### Procurement Task

代表：

> 为完成这条销售订单行，需要完成采购。

例如：`Procurement Task PT001`，目标购买 Qty 3。

### Purchase

代表：

> 实际发生的一次来源采购。

例如：

- Amazon Purchase A → Qty 1
- Amazon Purchase B → Qty 2

因此关系为：

```text
Sales Order Line
      ↓
Procurement Task
      ↓
1..N Purchases
```

异常情况下也可能存在 0 个成功 Purchase，任务被退回运营。

## 6. Purchase 与 Quantity 的关系

一个 Sales Order Line 可以被拆分成多个 Purchase。

例如 Sales Order Line Qty = 3，可以实际采购：

- Amazon Purchase A：Qty = 1
- Amazon Purchase B：Qty = 2

因此每一个 Purchase 必须明确承担了 Sales Order Line 中多少数量。

需要表达 **Quantity Allocation**。

## 7. 多 Purchase 的原因

同一个 Sales Order Line 产生多个 Purchase 可能因为：

- 一个来源库存不足；
- 需要拆分采购；
- 第一次采购失败；
- Seller 取消；
- 需要更换来源；
- 原来源价格变化；
- 配送时间不再满足要求；
- 其他采购异常。

因此 Purchase 历史不能被覆盖。

## 8. 所有 Purchase 历史必须保留

例如：

第一次 Purchase A：Source A，Cost $20，Result = Seller Cancelled。

第二次 Purchase B：Source B，Cost $24，Result = Shipped。

最终商品由 Purchase B 履约，但 Purchase A 仍需保留，因为它可能已经发生付款、后续退款、解释延迟、影响采购成本或采购人员行为记录。

## 9. 主来源优先

正常采购开始时优先查看 ERP 当前配置的 **Primary Source**。

如果主来源满足条件，则正常采购。

如果主来源不可用，则允许寻找新的替代来源。

因此 Primary Source 是默认采购依据，不是强制唯一采购来源。

## 10. 替代来源的判断顺序

当前采购人员寻找新来源时，主要考虑：

1. 商品一致性；
2. 配送方式；
3. 库存；
4. 配送速度；
5. 价格；
6. Seller。

即：

```text
Product Match
↓
Fulfillment
↓
Stock
↓
Delivery Speed
↓
Price
↓
Seller
```

## 11. 临时采购来源与长期 Product Source 分开

采购人员为了完成当前订单，可以临时找到一个新的来源。

该来源可以用于本次 Purchase，但不一定自动成为 Product 的长期 Backup Source。

需要区分：

- Use as Purchase Source；
- Save as Product Source。

二者不能默认绑定。

### 11.1 Product Source、Offer Snapshot、Purchase 分开

正式区分：

```text
Product Source
= ASIN 级来源商品身份

Source Offer Snapshot
= 某个时间点的 Seller、Price、Stock、Fulfillment、Delivery 条件

Purchase
= 最终真实发生的采购交易
```

Seller / Price / Stock / ETA 的变化不创建新的 ASIN Source。

但采购选择 Seller 和 Seller Blacklist 判断必须针对当次实际候选 / 选中的 Offer。

Purchase 自身必须保存实际 ASIN / Variant、Seller、Fulfillment、Price、Quantity、Tax、Shipping、Actual Paid、Promise / Delivery 等历史快照。

## 12. 采购限价

当前团队会根据订单情况设置 **Purchase Price Limit**。

如果 Actual Price ≤ Purchase Limit，通常正常采购。

如果 Actual Price > Purchase Limit，通常取消当前采购。

## 13. 采购限价不是绝对硬规则

超过限价后，运营还可能综合考虑：

- 当前 Store 出单情况；
- 当前 Store 销售额；
- Store Performance；
- 订单价值；
- 实际亏损程度；
- 当前经营需要。

因此 Purchase Limit 是 Team Procurement Policy，而不是 ERP 全局硬规则。

## 14. 采购人员可以打回任务

如果采购人员发现商品无法确认一致、没有合适配送方式、库存不足、配送时效无法满足、价格不可接受、Seller 不合适或没有可靠来源，可以将 Procurement Task 退回运营。

运营再决定：

- Cancel Sales Order；
- Wait；
- 其他处理方案。

因此：

> Procurement Cannot Fulfill ≠ Sales Order Automatically Cancelled。

## 15. Purchase 正常状态

正常采购过程只需要四个主要状态：

```text
待采购
↓
已下单
↓
已发货
↓
已签收
```

不设置单独的“采购成功”状态。

## 16. 待采购

表示销售订单已经产生采购需求，但尚未完成实际来源下单。

等待分配、已分配、已领取、正在寻找来源等属于 Task Assignment / Ownership，而不是 Purchase 生命周期主状态。

## 17. 已下单

表示已经在实际来源平台完成下单。

此时需要开始记录真实采购事实，例如：

- Product Source Reference（如有）；
- 实际 ASIN / Child ASIN / Variant；
- 实际 Seller / Seller ID；
- 实际 Fulfillment；
- Source Order Number；
- Quantity；
- Product Amount；
- Tax；
- Shipping；
- Actual Paid；
- Order Time；
- Payment Method；
- Promise / Estimated Delivery。

### 17.1 外部 Source Order ID 不是 ERP Purchase 唯一身份

ERP Purchase 保持 Procurement Task / Sales Order Line 粒度。

一次 Amazon Checkout 可以同时采购多个商品，并分别履约多个 Walmart Order Line。

因此：

> 多个 ERP Purchase 可以共享同一个 Amazon / Source Order ID。

开发不得把 Source Order ID 设计成“一条外部订单只能对应一个 ERP Purchase”的全局业务约束。

## 18. 已发货

表示来源平台显示该采购商品已经 Shipped。

当前团队将 Shipped 视为：

> Purchase 自身的重要采购执行节点。

如果 Team 选择在 ERP 中继续观察该 Source Shipment，则可以继续追踪其运输状态。

但必须明确：

> Source Shipped 不自动触发或要求 Platform Shipment，也不代表 Sales Order 必须进入某个固定物流阶段。

Purchase 本身仍然可以继续追踪到 Delivered / 已签收。

## 19. 已签收

Purchase 仍然需要继续追踪 Shipped → Delivered，最终进入已签收。

Shipped 是采购执行完成的重要节点，Delivered 是采购履约完成节点。

## 20. 采购异常不需要塞进正常主状态

采购过程中可能发生：

- 下单失败；
- Seller Cancelled；
- Source Cancelled；
- Refund；
- Delivery Failure；
- Lost；
- Delay；
- 其他异常。

这些更适合记录为 Purchase Exception / Resolution，而不是无限扩展主状态。

正常主路径保持：

```text
待采购 → 已下单 → 已发货 → 已签收
```

## 21. 销售订单和采购订单独立管理

Sales Order 关注客户侧：取消、发货、Delivered、退款、退货、售后。

Purchase 关注来源侧：是否购买、使用哪个 Source、数量、实际成本、是否发货、取消、退款、签收。

两者有关联，但拥有独立生命周期。

## 22. 销售订单取消不会强制同步取消 Purchase

如果 Walmart Customer 取消订单，采购端可能取消 Purchase、不取消、商品继续配送或后续再处理。

当前团队没有统一规则要求 Sales Order Cancel → Purchase Cancel。

这种联动只能由 Team Workflow 决定。

## 23. Purchase 异常后的处理

Purchase 出现异常后，由 Operations + Procurement 共同决定下一步。

可能：

- 剩余物流时间足够时重新采购；
- 时间不够、无来源或成本不可接受时取消销售订单；
- 暂时等待后续情况或客户主动退款。

因此采购异常本身不决定销售订单最终状态。

## 24. 真实采购成本

订单审核阶段可以存在 Estimated Purchase Cost，但最终利润不能一直依赖预计成本。

真实采购成本来自 Actual Purchases。

如果发生多个 Purchase、Purchase Refund、Purchase Cancellation、未取消的多余采购或其他资金调整，都可能影响 Actual Procurement Cost。

因此必须区分：

- Estimated Purchase Cost；
- Actual Purchase Cost。

## 25. 内部采购支付方式

当前内部采购主要使用：

- Company Credit Card；
- Virtual Card。

不存在每个采购订单都先请款再购买。

因此“采购请款”不是当前内部采购标准流程。

## 26. 外部采购团队模式

如果采购任务交给外部专门采购团队，流程为：

**Procurement Task → External Procurement Team → 完成采购 → 提供发货凭证 → 团队周期性对账 → 结算付款**

## 27. 外部采购团队采用周期性结算

不是每一单采购完成就马上打款，而是定期汇总采购记录进行对账和结算。

未来需要支持 External Procurement Statement / Settlement，用于核对：

- Purchase；
- Quantity；
- Amount；
- Shipment Evidence；
- Settlement Status。

## 28. 外部采购发货凭证

当前通常包含两张截图：

1. Order Details：显示 Amazon 订单详细信息；
2. Track Package：显示商品已发货、Package 与物流状态。

因此外部采购 Purchase 可以关联 Shipment Evidence，并支持多个附件或证据文件。

## 29. 外部采购结算与采购订单是两个业务对象

Purchase 代表买了什么。

External Procurement Settlement 代表与外部采购服务商之间的钱有没有结清。

采购履约状态与外部结算状态不能混用。

## 30. Source 实际配送模式

当前实际模式是 Amazon / Source 直接将商品配送给 Walmart Customer。

商品不会先进入团队自有中转仓。

```text
Amazon / Source
      ↓
Walmart Customer
```

## 31. Procurement 与 Logistics 的边界

Procurement 负责：

- Procurement Task；
- Actual Purchase；
- Purchase Quantity；
- Actual Cost；
- Purchase Status；
- Source Purchase Evidence；
- Source Shipment Fact（如果可获得）。

Procurement 不负责定义：

- Walmart Platform Shipment；
- Platform Tracking；
- Carrier Tracking Mapping；
- Tracking Synchronization；
- Delay / Lost Detection。

这些属于 Logistics Domain。

---

## 32. Source Tracking 是 Purchase 的可选关联事实

Purchase 可能能够获得 Source Tracking，也可能：

- TBA 等数据只能有限观察；
- 外部采购团队无法提供完整 Tracking；
- Team 根本不使用 ERP 管理 Source Logistics。

因此：

> Source Tracking 可以存在，也可以 Unknown / Unobservable。

缺少 Source Tracking 不自动代表 Purchase Error。

---

## 33. Platform Shipment 与 Purchase 不存在固定顺序

Sales Order 的 Platform Shipment：

> 可以与 Purchase 建立业务关联，但不存在系统级必然先后关系。

不能硬编码：

```text
Purchase Shipped
→ Platform Shipment
```

也不能要求：

> Platform Shipment 必须能够证明 Purchase 已经 Shipped。

具体前置条件由 Team Workflow 决定。

---

## 34. Procurement 模型在 Purchase 事实处截止

Procurement 只需要准确回答：

- 为什么需要采购；
- 谁负责采购；
- 实际从哪里采购；
- 买了多少；
- 花了多少钱；
- Purchase 当前是什么状态；
- 是否存在取消 / 退款 / 异常；
- 是否能够观察 Source Shipment。

Platform Shipment、Platform Tracking 和物流异常处理以：

> [物流生命周期](./06-logistics-lifecycle.md)

为准。

---

## 35. 当前完整采购主流程

```text
Sales Order Line
↓
Order Audit Pass
↓
Create Procurement Task
↓
Operations Assign
↓
Procurement Claims Task
↓
Check Primary Source
↓
Source Available?
↓ Yes
Product Match
→ Fulfillment
→ Stock
→ Delivery
→ Price
→ Seller
↓
Place Purchase
↓
已下单
↓
已发货
↓
Purchase 执行节点完成
↓
Purchase 自身继续追踪（如果可观察）
↓
已签收
```

## 36. 替代来源流程

如果 Primary Source 不可采购：

```text
Primary Source Failed
↓
Search Alternative Source
↓
Product Match
↓
Fulfillment
↓
Stock
↓
Delivery
↓
Price
↓
Seller
```

找到可采购来源：Create Purchase。

找不到：Return Procurement Task to Operations，由运营决定 Cancel Sales Order、Wait 或其他处理。

## 37. 拆单采购流程

例如 Sales Order Line Qty 3，可以形成：

- Purchase A：Qty 1
- Purchase B：Qty 2

ERP 需要持续判断 Procurement Task 的目标数量是否已经全部覆盖。

## 38. 采购业务的关键数量概念

至少需要区分：

- Sales Ordered Quantity；
- Procurement Required Quantity；
- Purchase Quantity；
- Cancelled Purchase Quantity；
- Successfully Purchased Quantity；
- Shipped Quantity；
- Delivered Quantity。

## 39. 当前采购业务原则

1. Order Audit Pass 后，当前团队立即进入采购。
2. ERP 生成待采购任务。
3. 运营分配采购任务。
4. 采购团队 / 人员领取任务。
5. Sales Order Line、Procurement Task、Purchase 是不同对象。
6. 一个 Sales Order Line 可以关联多个 Purchase。
7. 一个 Sales Order Line 的数量可以拆到多个 Purchase。
8. Primary Source 优先，但不是唯一 Source。
9. 主来源不可用时允许寻找临时替代来源。
10. 临时采购 Source 不自动成为长期 Product Source。
11. Product Source、Source Offer Snapshot、Purchase 是三个不同层次。
12. Seller Blacklist 与采购判断针对当次实际 Offer 的 Seller。
13. Purchase 必须保存实际 ASIN / Variant、Seller、Fulfillment、金额和配送承诺等交易快照。
14. 多个 ERP Purchase 可以共享同一个外部 Source Order ID。
15. 采购来源主要按商品一致性、配送方式、库存、速度、价格、Seller 判断。
16. Purchase Limit 属于团队经营策略。
17. 超过限价通常不采购，但可以结合店铺经营情况进一步决定。
18. 采购人员可以将无法采购的任务打回运营。
19. 无法采购不自动等于取消销售订单。
20. Purchase 正常状态为：待采购 → 已下单 → 已发货 → 已签收。
21. 不设置“采购成功”状态。
22. Source 显示 Shipped 后，Purchase 自身的采购执行动作可以视为基本完成。
23. Source Shipped 不自动触发 Platform Shipment，也不形成固定 Sales Order 状态跳转。
24. Purchase 可以继续追踪至 Delivered；Source Shipment 不可观察时允许保持 Unknown。
25. 销售订单与采购订单关联但分别管理。
26. 销售订单取消不强制同步取消采购。
27. Purchase 异常后允许重新采购。
28. 所有历史 Purchase 必须保留。
29. 最终采购成本来源于实际 Purchase。
30. 内部采购主要使用公司信用卡或虚拟卡。
31. 内部采购不存在逐单请款流程。
32. 外部采购团队采用周期性对账结算。
33. 外部采购可以保存发货凭证。
34. Purchase 状态和 External Settlement 状态必须分开。
35. 当前团队的 Source 通常直接配送给最终客户。
36. Procurement 不定义第三方 Platform Tracking 的生命周期。
37. Source Tracking 与 Sales Platform Tracking 必须分开。
38. Platform Shipment / Tracking / Delay / Lost 属于 Logistics Lifecycle，而不是 Procurement Lifecycle。
39. Purchase 与 Platform Shipment 可以关联，但不存在 ERP 全局固定先后关系。

## 40. 采购生命周期建模结论

采购流程真正管理的不是“销售订单对应一个 Amazon Order Number”，而是：

> **一个销售订单履约需求，如何通过一个或多个实际采购行为获得足够商品，并形成真实成本和可追踪的供应履约记录。**

核心关系为：

```text
Sales Order Line
        ↓
Procurement Task
        ↓
Quantity Requirement
        ↓
1..N Purchases
        ↓
Source + Actual Cost + Quantity
        ↓
Shipped
        ↓
Delivered
```

同时：

```text
Purchase
   ≠
Sales Shipment
   ≠
Sales Platform Tracking
   ≠
External Procurement Settlement
```

这些业务对象相互关联，但必须拥有各自独立的生命周期。
