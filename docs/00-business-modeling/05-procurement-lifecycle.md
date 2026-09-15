# 采购生命周期业务模型 v0.2

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

- Source；
- Purchase Order Number；
- Quantity；
- Actual Price；
- Seller；
- Order Time；
- Payment Method；
- 预计配送时间。

## 18. 已发货

表示来源平台显示该采购商品已经 Shipped。

当前团队将 Shipped 视为采购执行动作已经基本完成。

此时销售订单可以正式进入后续物流跟踪阶段。

但 Purchase 本身仍然不能停止追踪。

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

## 31. 第三方物流的角色

第三方物流并不实际接收 Amazon 商品后再转运给客户，而主要提供 Walmart 可以接受和追踪的物流单号 / 物流轨迹服务。

可能使用：

- 佳成；
- 递四方；
- USPS；
- 其他物流服务。

## 32. 销售平台 Tracking 与 Source Tracking 不是同一个概念

实际商品配送是 Amazon → Customer。

Walmart 侧提交的 Tracking 可能使用第三方物流提供的单号。

因此必须区分：

### Source Tracking

Amazon / Source 侧真实配送信息。

### Sales Platform Tracking

提交给 Walmart 等销售平台的 Carrier、Tracking Number 和 Tracking Events。

二者有关联但不是同一个 Tracking 对象。

## 33. 当前第三方轨迹业务

前期第三方物流轨迹按照既定过程运行。

后续商品进入清关及后续运输阶段后，逐步同步真实物流轨迹，一直到 Delivered。

整个过程当前主要依靠人工手动操作，是明显的高人工成本环节。

## 34. 物流同步不属于 Purchase 本身

Purchase Lifecycle 只负责知道：

- Ordered；
- Shipped；
- Delivered；
- Source Tracking。

而第三方物流单号、Walmart Shipment、Platform Tracking Sync、Tracking Event Mapping 属于 Logistics Lifecycle。

因此采购模型在这里截止。

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
采购执行完成
↓
进入 Logistics Tracking
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
11. 采购来源主要按商品一致性、配送方式、库存、速度、价格、Seller 判断。
12. Purchase Limit 属于团队经营策略。
13. 超过限价通常不采购，但可以结合店铺经营情况进一步决定。
14. 采购人员可以将无法采购的任务打回运营。
15. 无法采购不自动等于取消销售订单。
16. Purchase 正常状态为：待采购 → 已下单 → 已发货 → 已签收。
17. 不设置“采购成功”状态。
18. Source 显示 Shipped 后，采购执行动作可以视为完成。
19. Purchase 仍持续追踪至 Delivered。
20. 销售订单与采购订单关联但分别管理。
21. 销售订单取消不强制同步取消采购。
22. Purchase 异常后允许重新采购。
23. 所有历史 Purchase 必须保留。
24. 最终采购成本来源于实际 Purchase。
25. 内部采购主要使用公司信用卡或虚拟卡。
26. 内部采购不存在逐单请款流程。
27. 外部采购团队采用周期性对账结算。
28. 外部采购可以保存发货凭证。
29. Purchase 状态和 External Settlement 状态必须分开。
30. Source 直接配送给最终客户。
31. 第三方物流主要提供销售平台可接受的 Tracking / 轨迹服务。
32. Source Tracking 与 Sales Platform Tracking 必须分开。
33. 第三方物流轨迹同步属于 Logistics Lifecycle，而不是 Procurement Lifecycle。
34. 当前物流轨迹同步人工成本较高，是后续业务建模的重要问题。

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
