# 物流生命周期业务模型 v0.2

> 状态：已确认  
> 所属阶段：ERP 阶段 0 — 业务建模

## 1. 物流业务的核心定位

物流模块负责提供与订单运输有关的基础业务能力，但：

> **物流流程不是所有团队的必经 ERP 流程。**

团队可能：

- 完全使用 ERP 内置物流能力；
- 只使用 ERP 保存 Tracking；
- 只从销售平台同步物流状态；
- 使用自己的外部物流系统；
- 使用自己的 Tracking Number；
- 在 ERP 外完成发货；
- 完全不使用 ERP 的发货功能。

因此 ERP 应该提供物流能力，但不能要求所有订单必须经过固定的物流 Workflow。

---

## 2. Purchase、Source Shipment、Platform Shipment 不是强依赖链

不能固定认为：

```text
Purchase
→ Source Shipped
→ Platform Shipment
→ Walmart Delivered
```

实际上它们是三个可以独立存在、按业务需要建立关联的对象。

---

## 3. Procurement 与 Platform Shipment 不存在必然先后关系

当前团队实际业务中：

> Walmart 发货确认不要求 Amazon / Source 已经进入 Shipped。

甚至：

> Walmart Shipment 与采购行为本身也不存在系统层面的必然前后关系。

因此 ERP 不应硬编码：

> `Source Shipped = Platform Shipment 的前置条件`

这只是某些团队可能采用的 Workflow 规则。

---

## 4. 三类核心物流对象

ERP 通用物流模型至少需要理解三个独立概念。

### 4.1 Source Shipment

表示：

> 某个供应来源实际发生的商品配送。

例如：

Amazon Purchase → Customer。

它可以包含：

- Source Order；
- Package；
- Source Carrier；
- Source Tracking；
- Current Status；
- ETA；
- Tracking Events；
- Delivered Time。

但是：

> Source Shipment 不一定能够被 ERP 完整获取。

### 4.2 Platform Shipment

表示：

> 销售平台中的发货业务记录。

例如 Walmart Order Line 对应的 Shipment。

它可以包含：

- Sales Order；
- Order Line；
- Quantity；
- Carrier；
- Tracking Number；
- Ship Time；
- Platform Shipment Status；
- Platform Delivery Status。

Platform Shipment：

> 可以由 ERP 创建，也可以在 ERP 外创建后由 ERP 同步回来。

### 4.3 Tracking Reference

表示：

> 某个系统或物流服务中存在的一条可追踪物流标识。

可能包括：

- Amazon Tracking；
- USPS；
- 4PX；
- 佳成；
- 其他 Carrier Tracking；
- 外部物流系统 Tracking。

Tracking Reference 可以关联：

- Source Shipment；
- Platform Shipment；

也可能暂时：

> 没有可以确认的另一侧关联关系。

---

## 5. 三层物流不存在必须关联的关系

ERP 不应该要求：

**Source Tracking**

必须对应：

**Platform Tracking**

也不要求：

**Purchase**

必须对应：

**Platform Shipment**。

可能存在：

### 情况 A

ERP 知道：

- Purchase；
- Source Tracking；
- Platform Tracking。

三者可以建立完整关系。

### 情况 B

ERP 只知道：

- Platform Shipment；
- Platform Tracking。

没有 Source Tracking。

### 情况 C

团队使用外部物流系统。

ERP 只从 Walmart 同步：

- Tracking；
- Shipment State。

### 情况 D

团队甚至不通过 ERP 执行发货。

ERP 后续只同步平台最终状态。

这些都属于合法的 ERP 使用模式。

---

## 6. 内置物流是 Optional Capability

因此系统应该区分：

### Logistics Capability Available

ERP 提供：

- Tracking 管理；
- Carrier 管理；
- 发货；
- Tracking 查询；
- Delay Detection；
- Lost Detection；
- Logistics Reconciliation。

### Team Workflow Usage

团队决定：

> 实际使用其中哪些能力。

不能因为系统有 Logistics Module，就要求所有 Team：

> 必须通过 ERP 发货。

---

## 7. Platform Shipment 可以独立创建

团队只要拥有一个需要发货的 Sales Order / Order Line，就可以根据自己的流程创建：

**Platform Shipment**

不要求 ERP 能证明：

- 已经采购；
- Source 已 Shipped；
- 已经获得 Source Tracking。

这些是否需要作为发货前置条件：

> 由 Team Workflow 自己决定。

---

## 8. 当前团队的实际 Platform Shipment 流程

当前团队可能采用：

**获取 Platform Tracking**

↓

**Walmart Confirm Shipment**

与此同时，采购端继续自己的生命周期：

**Purchase**

↓

**Source Shipped**

↓

**Source Delivered**

两条流程可以在时间上交叉。

它们可以关联，但：

> 不存在系统级强制依赖。

---

## 9. Sales Order Line 与 Shipment

ERP 通用能力必须允许：

> 一个 Sales Order Line 对应一个或多个 Platform Shipment。

同时也允许：

> 一个 Sales Order Line 对应一个或多个 Source Shipment。

两者数量不要求相同。

---

## 10. 当前团队的 Walmart Tracking 策略

当前团队采用：

> 一个 Sales Order Line 向 Walmart 提交一条 Tracking。

但这是：

**Current Team Policy**

而不是：

**ERP Constraint**。

其他 Team 可以根据：

- Platform；
- Package；
- Quantity；
- 自己的履约方式；

决定使用一个或多个 Shipment / Tracking。

---

## 11. Quantity 必须能够独立追踪

例如：

**Sales Order Line**

Qty = 3

采购可能为：

- Purchase A：Qty 1；
- Purchase B：Qty 2。

Source Shipment 也可能：

- Shipment A：Qty 1；
- Shipment B：Qty 2。

Platform Shipment 又可能：

- 只使用一个 Tracking；
- 或拆成多个 Tracking。

因此：

> Purchase Quantity、Source Shipment Quantity 和 Platform Shipment Quantity 都需要独立表达。

不能假设三者结构完全一致。

---

## 12. Source Tracking 是可选信息，而不是系统必填

如果 ERP 可以获取真实 Source Tracking：

则可以持续追踪：

- Shipped；
- In Transit；
- ETA；
- Delivered；
- Delay；
- Lost。

但如果：

- TBA 无公开查询能力；
- 外部采购团队不提供完整 Source Tracking；
- Team 不使用 ERP 管理采购物流；

则：

> Source Tracking 可以不存在或状态未知。

这种情况不应该自动被判定成：

> Logistics Error。

---

## 13. TBA 的业务含义

Amazon TBA Tracking 在某些情况下：

> 只能通过对应 Amazon Account 的订单页面看到真实状态。

如果采购由自己的账户完成，并且 ERP 有合法的数据访问能力，则可以：

> 获取并保存真实 Source Status。

如果采购由外部采购团队完成，而团队没有相应账户访问：

则可能只知道：

- 已下单；
- 已发货凭证；
- 其他有限信息。

此时 Source Delivery State 可以保持：

**Unknown / Unobservable**

而不是伪造一个确定状态。

---

## 14. Shipment Evidence 与 Tracking 不同

对于外部采购团队：

可能有：

- Order Details Screenshot；
- Track Package Screenshot；
- Shipment Evidence。

这些能够证明某些采购或发货事实。

但：

> Evidence 不等于可持续查询的 Tracking Feed。

因此 ERP 应区分：

**Evidence**

和：

**Tracking Data**。

---

## 15. 外部提供的 Platform Tracking

团队可能使用：

- ERP 内置物流；
- 第三方物流服务；
- 自己已有的物流系统；
- 人工输入的 Carrier / Tracking。

ERP 的职责可以是：

- 保存；
- 验证格式；
- 判断平台是否接受；
- 查询能够查询到的实际状态；
- 同步平台结果。

但是：

> ERP 不应假设自己知道该 Tracking 与某一个 Source Shipment 的真实物理关系。

只有在数据确实能够建立关系时才关联。

---

## 16. Source Shipment 与 Platform Tracking 的关联是 Optional Relation

可以存在：

**Platform Tracking A**

↔

**Source Shipment B**

的已知映射。

也可以：

**Platform Tracking A**

↓

`source_shipment = unknown`

因此：

> 不允许为了让数据模型完整而强制创建虚假的 Source Shipment 关联。

---

## 17. Tracking Synchronization 应改成 Optional Reconciliation

更准确的概念是：

> 对“团队选择纳入 ERP 管理，并且 ERP 可以观察到”的物流数据进行状态比较与 Reconciliation。

所以：

**Logistics Reconciliation Scope**

需要先知道：

> 这条订单到底有哪些物流来源被纳入监控。

---

## 18. Observable Logistics Scope

对每个 Order / Shipment，可以存在不同的可观察范围。

例如：

### Full

ERP 可以看到：

- Source Tracking；
- Platform Tracking；
- Walmart State。

### Partial

ERP 只能看到：

- Platform Tracking；
- Walmart State。

### Platform Only

ERP 只有：

- Walmart State。

因此物流完成判断必须基于：

> 当前可观察范围。

---

## 19. Delivered 需要继续区分不同事实

不能只有：

`Delivered = true`

至少存在：

### Source Delivered

已知 Source Shipment Delivered。

### Carrier / Tracking Delivered

某个被追踪 Carrier 已显示 Delivered。

### Platform Delivered

销售平台已经识别订单 / Shipment 为 Delivered。

三者是不同事实。

---

## 20. ERP 销售订单的 Platform Delivery State

当前 ERP 对 Walmart 销售订单的：

> 平台配送完成状态

以：

**Walmart Platform = Delivered**

为准。

也就是说：

> 如果 Walmart 已将订单识别为 Delivered，则 Sales Order 的 Platform Delivery Status 可以是 Delivered。

---

## 21. Platform Delivered 不等于“所有物流证据均完整”

即使：

**Walmart Delivered**

仍然可能：

- Source Tracking 未知；
- Source Shipment 未确认 Delivered；
- 外部采购团队没有提供最终 Tracking；
- Carrier 数据仍未同步结束。

因此需要区分：

### Platform Delivery Completed

Walmart 已 Delivered。

### Logistics Observation Completed

当前要求监控的物流数据已经达到结束条件。

---

## 22. 当前团队的人工作业标准不是所有订单都能满足

如果团队拥有完整可观察数据：

例如：

- Walmart Delivered；
- Source Delivered；
- Carrier Tracking Delivered；

则人工可以认为：

> 物流证据完整闭环。

但是如果：

- 外部采购团队无法提供 Source Delivered；
- TBA 状态无法继续获取；

则不能要求：

> 三层必须全部 Delivered 才允许订单继续。

因此“三者均 Delivered”只能是：

> 数据完整时的一种强确认状态。

不能成为系统全局完成条件。

---

## 23. Logistics Completion 需要分级

建议业务上区分：

### Platform Delivered

平台确认已送达。

这是订单平台状态。

### Observed Delivery Complete

团队要求监控且可以观察的所有 Shipment 已完成。

### Fully Reconciled

Source、Tracking、Platform 等所有预期数据均可用，并且完成一致性核对。

### Partially Observed

部分物流来源不可获取，但平台已经 Delivered 或主要履约已经完成。

这使系统能够诚实表达：

> 我知道什么，以及我不知道什么。

---

## 24. 完整履约数量与 Reconciliation

如果某个 Team 的 Reconciliation Scope 明确要求追踪全部 Source Shipment，并且这些数据可观察：

则：

> 所有 Required Quantity 对应的 Source Shipment 都完成，才可 Fully Reconciled。

例如：

**Order Line Qty 3**

- Source Shipment A → Qty 1 → Delivered；
- Source Shipment B → Qty 2 → Not Delivered。

即使 Walmart 已显示 Delivered：

> `Fully Reconciled = false`

直到团队要求追踪的全部数量都完成。

但是：

> 这不是所有 Team、所有 Order 的全局硬条件。

如果 Source Shipment 不可观察：

系统应该表达：

**Source Delivery Unknown**

而不是永久卡住整个 Sales Order。

---

## 25. Delay Detection

如果 ERP 能获得所需数据，可以根据：

- Walmart Customer Promise Date；
- Walmart Expected Delivery Date；
- Source ETA；
- Carrier ETA；

进行延迟监控。

但：

> Delay Monitoring 本身也是可选能力。

如果 Team 不向 ERP 提供 Source Logistics：

仍然可以仅基于：

- Platform Tracking；
- Walmart Status；

做有限判断。

---

## 26. Delay Risk

可以在真正超时前识别：

> 当前预计无法按承诺时间送达。

例如：

**Source ETA > Walmart Promise Date**

可以产生：

**Delay Risk**。

具体阈值：

> Team Policy。

---

## 27. Lost Detection

ERP 可以结合可用信息判断：

- N 天无更新；
- Carrier Lost；
- Amazon Refund / Replacement；
- Customer Complaint；
- Platform Tracking 异常。

但是：

> 能使用哪些信号取决于实际数据是否存在。

---

## 28. Suspected Lost 与 Confirmed Lost

建议继续区分：

### Suspected Lost

证据不足，但风险较高。

### Confirmed Lost

拥有明确丢件证据。

这属于 Logistics Exception，而不是 Sales Order 主状态。

---

## 29. Tracking Invalid 与 Physical Delivery Failure 分开

例如：

**Platform Tracking Invalid**

并不意味着：

> 实际商品没有运输。

因此必须分别处理：

### Tracking Data Problem

和：

### Physical Shipment Problem

这对于使用外部物流系统的团队尤其重要。

---

## 30. ERP 不应强制 Team 使用内置 Carrier 服务

基础能力可以支持：

- Built-in Carrier Integration；
- External Carrier；
- Manual Tracking；
- External Logistics System；
- Platform-only Sync。

Team 自己决定实际模式。

---

## 31. ERP 也不应强制通过自身 Confirm Shipment

Team 可以：

### 模式 A

通过 ERP 调平台 API 发货。

### 模式 B

在 Walmart Seller Center 人工发货。

### 模式 C

由第三方系统发货。

ERP 后续只：

> 从 Walmart 同步 Shipment State。

三种模式都应该能被系统管理。

---

## 32. 物流的真正核心不是“控制发货”

ERP Logistics 的核心能力应该是：

> **记录、关联、观察和管理物流事实。**

控制发货只是其中一种可选 Operation。

---

## 33. Logistics 与 Procurement 解耦

虽然当前业务中采购与物流经常有关联，但 ERP 必须允许：

```text
Procurement exists
but no ERP Logistics
```

也允许：

```text
Platform Shipment exists
but no Procurement
```

例如：

- 自有库存；
- 外部仓库；
- 第三方履约；
- 人工履约；
- 其他模式。

这也是未来支持不同团队和其他平台的基础。

---

## 34. Logistics 与 Workflow 的关系

ERP 提供基础能力，例如：

- Create Platform Shipment；
- Record Tracking；
- Sync Platform Shipment；
- Query Carrier Status；
- Query Source Shipment；
- Detect Delay；
- Detect Lost；
- Reconcile Status。

当前团队可以创建自己的 Workflow。

其他团队则完全可以：

> 只使用其中几个 Operation。

---

## 35. 当前团队物流模板应只是 Template

当前团队未来可能采用：

**Prepare Tracking**

↓

**Platform Shipment**

↓

**Monitor Source when available**

↓

**Monitor Tracking**

↓

**Monitor Walmart**

↓

**Detect Delay / Lost**

↓

**Platform Delivered**

↓

**Continue optional reconciliation**

这可以成为：

> EasyQuickAIERP 内置 Workflow Template。

但不是系统底层固定生命周期。

---

## 36. 当前确认的物流业务原则

1. ERP 内置物流是可选能力。
2. Team 不一定通过 ERP 发货。
3. Team 可以使用自己的物流系统和 Tracking。
4. Purchase、Source Shipment、Platform Shipment 不存在系统级必然关系。
5. Platform Shipment 不要求 Purchase 已完成。
6. Platform Shipment 不要求 Source 已 Shipped。
7. Source Shipment 与 Platform Tracking 不要求一一对应。
8. 一个 Sales Order Line 可以关联多个 Shipment / Tracking。
9. 当前团队向 Walmart 通常只上传一条 Tracking，但不是 ERP 限制。
10. Shipment 需要支持 Quantity。
11. Source Tracking 是可选数据。
12. Source Tracking 不存在或不可获取，不自动视为错误。
13. TBA 等数据可以处于 Unknown / Unobservable。
14. 外部采购 Evidence 与真实 Tracking 是不同对象。
15. Platform Tracking 可以来自 ERP 外部。
16. ERP 只在数据真实可关联时建立 Source ↔ Platform 关系。
17. Platform Delivered、Source Delivered 和 Tracking Delivered 是不同事实。
18. Walmart Delivered 是 Walmart Sales Order 的平台 Delivery State。
19. Walmart Delivered 不要求 Source Tracking 已知。
20. Logistics Completion 必须考虑 Observable Scope。
21. 对于要求追踪且可观察的全部履约数量，只有全部完成后才可 Fully Reconciled。
22. 部分物流数据不可获取时允许 Partially Observed。
23. 不得为了数据完整性伪造 Source Shipment 关联或确定状态。
24. Delay / Lost Detection 根据 Team 选择的数据范围运行。
25. Logistics 与 Procurement 必须解耦。
26. Logistics 与 Sales Order 有关联但生命周期独立。
27. ERP 基础能力重点是记录、关联、同步、监控和异常处理。
28. Workflow 决定团队实际采用哪一条物流流程。

---

## 37. 物流生命周期建模结论

EasyQuickAIERP 不应该建立：

```text
Purchase
→ Amazon Shipped
→ Tracking
→ Walmart Shipment
→ Delivered
```

这样的固定链条。

更准确的模型是：

```text
Sales Order
   │
   ├── Procurement（可选）
   │      └── Purchase
   │             └── Source Shipment（可能存在/可能不可观察）
   │
   └── Platform Shipment（可由 ERP 或外部系统创建）
           └── Platform Tracking（0..N）
```

这些对象之间：

> 可以建立关系，但关系是可选的。

ERP 最重要的责任是：

> 在已有数据范围内准确记录事实，并明确哪些事实已知、哪些未知，而不是强行把所有团队的履约过程塑造成同一条链。

因此 Logistics Module 最终应该是一个：

> **可插拔、可部分使用、可以独立于采购存在的业务能力。**
