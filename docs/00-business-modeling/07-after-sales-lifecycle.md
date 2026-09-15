# After-sales Lifecycle（售后生命周期）v0.3

> 状态：阶段 0 已确认业务基线  
> 目标：定义 ERP 在售后域真正需要管理的平台事实、运营动作与财务影响边界。

## 1. 模块定位

售后模块首先是**平台售后数据管理**，不是客服 CRM。

当前团队真正需要的是：

> 从 Walmart 拉取可通过 API 获得的售后数据，将其与原 Sales Order / Order Line 关联，让运营识别哪些售后需要处理，执行 Refund，并持续同步最终 Return / Refund 结果。

当前阶段不建设：

- Walmart 站内信完整沟通；
- Email 完整沟通；
- 待回复 / 已回复 / 等待客户等客服状态；
- 依赖完整 Customer Service Case 的工单体系。

这些当前主要存在于平台或邮件系统中，ERP 不强行建模。

---

## 2. 当前 Walmart 售后主流程

```text
Platform Generates After-sales Data
        ↓
ERP Pulls Return / Refund
        ↓
Link to Sales Order / Order Line
        ↓
Show Pending After-sales
        ↓
Operations Reviews
        ↓
If Refund Needed
→ Operations Processes Refund
        ↓
Procurement Handles Purchase Side if Needed
        ↓
ERP Continues Syncing Return / Refund Status
        ↓
After-sales Completed
```

---

## 3. ERP 管理平台可获得的售后事实

ERP 主要保存平台 API 可以获取的结构化信息，例如：

- Return Order ID；
- Sales Order / Order Line；
- SKU；
- Quantity；
- Return Status；
- Return Reason；
- Refund Status；
- Refund Reason；
- Refund Amount；
- Return Method；
- Tracking；
- Created Time；
- Updated Time；
- 其他平台返回字段。

这些属于 **Platform After-sales Facts**。

退款原因可以通过 API 获取，因此 ERP 应直接保存、查询、筛选和统计，不需要从客户沟通内容中推断。

---

## 4. 客户沟通不属于当前核心模型

真实售后可能通过以下渠道被运营感知：

- Walmart 站内信；
- Email；
- Customer Complaint；
- Platform Case；
- Chargeback。

但当前 ERP 无法完整获得这些沟通数据，因此：

> 不把客户沟通设计成售后模块的必要组成部分。

如果未来平台开放对应 API，或 ERP 接入 Email / Message Connector，再作为独立 Communication 能力扩展。

---

## 5. 同一个 Return 只管理一个持续更新的 Return 对象

对于 Walmart API 拉取到的同一个 Return：

> ERP 持续更新同一个售后记录。

不因为：

- 状态变化；
- 客户再次发消息；
- 运营产生新的人工沟通；

就创建新的 After-sales Case。

当前团队真正关心的是：

> 这个 Return 是否仍需要处理，以及最终是否完成。

具体完成条件后续根据 Walmart API 实际字段定义，阶段 0 不提前写死。

---

## 6. Return 与 Refund 必须区分

虽然当前团队在操作上把它们视作一条售后流程，但数据上必须分开。

### Return

表示商品进入退货流程。

### Refund

表示实际发生退款。

可能存在：

- Return + Refund；
- Refund Only。

因此：

> Refund-only 必须独立显示，即使不存在 Return Order。

---

## 7. 当前退款业务

当前实际业务以：

- Full Refund；
- Refund without Return；

为主。

Partial Refund 也可能存在。

运营关注的核心问题包括：

1. 售后属于哪张订单 / 哪个 Order Line；
2. 当前 Return 状态；
3. Return / Refund 原因；
4. 是否需要处理 Refund；
5. 已退款金额；
6. Refund 是否完成；
7. 是否仍有后续动作。

因此售后模块更接近：

> **平台售后事实 + 待处理视图。**

---

## 8. 待处理售后

ERP 需要帮助运营识别仍需动作的售后，例如：

- Return 已产生但 Refund 尚未处理；
- Refund 需要运营确认；
- Refund 处理中；
- 状态异常；
- 其他团队定义的待处理条件。

具体条件后续依据平台 API 实际字段和 Team Policy 定义。

---

## 9. Cancellation 与售后的关系

订单取消可能来自：

- Customer / Walmart；
- Operations 主动取消；
- Procurement 无法履约后由运营取消。

Cancellation 本身主要属于 Sales Order 事实。

如果取消进一步产生：

- Refund；
- Return；

则进入售后数据。

不需要为了售后模块额外复制一套复杂 Cancellation Case。

---

## 10. 采购侧处理保持独立

售后产生后，采购人员根据实际情况处理 Purchase，例如：

- Cancel Purchase；
- 不取消；
- 等待；
- 重新采购；
- 其他处理。

但：

> Sales After-sales 不强制驱动 Procurement 状态。

售后与采购有关联，但生命周期独立。

---

## 11. Delay / Lost 主要属于 Logistics Domain

Delay / Lost 的检测首先属于 Logistics Lifecycle。

物流异常可能最终导致：

- Customer Refund；
- Return；
- Re-purchase。

一旦平台产生正式 Return / Refund 数据：

> After-sales Module 再负责记录和处理平台售后结果。

因此不需要把所有 Delay / Lost 预先变成 After-sales Case。

---

## 12. 售后对利润的影响

售后发生后，ERP 可以立即根据当前数据更新：

> **Estimated Profit**。

例如 Full Refund 发生后，可以立即降低预计收入和预计利润。

但：

> 这仍然只是 ERP 内部估算值。

最终平台可能在账单中体现：

- Refund；
- Commission Reversal；
- Fee；
- Adjustment；
- Tax；
- Shipping Adjustment；
- 其他 Settlement Transaction。

因此准确财务结果以：

> **Settlement / Recon**

为准。

---

## 13. Estimated 与 Actual 财务结果分开

售后财务影响应理解为：

```text
Return / Refund Data
        ↓
Estimated Financial Impact
        ↓
Wait for Settlement / Recon
        ↓
Actual Financial Impact
        ↓
Recalculate Accurate Profit
```

售后模块告诉财务模块：

> **发生了什么。**

财务模块根据账单回答：

> **平台最终实际记了多少钱。**

---

## 14. 售后窗口

Sales Order 仍然需要存在：

> **After-sales Window**。

用于判断订单是否已经基本脱离正常退货 / 退款风险期。

阶段 0 不定义具体 N 天。

具体窗口后续由：

- Platform Policy；
- Team Policy；

确定。

---

## 15. 当前确认的售后业务原则

1. 售后模块首先是平台售后数据管理，不是客服 CRM。
2. ERP 拉取 Walmart 可以通过 API 提供的售后数据。
3. 售后记录必须关联原 Sales Order / Order Line。
4. 运营主要查看哪些售后仍需要处理。
5. Refund 是运营最重要的处理动作之一。
6. Refund / Return Reason 可以通过平台 API 获取并保存。
7. Return 和 Refund 在数据上必须区分。
8. Refund-only 必须独立显示。
9. 同一个 Walmart Return 持续更新同一条售后记录。
10. 不需要把客户沟通拆成多个 After-sales Case。
11. Walmart Message / Email 当前不进入 ERP 售后工作流。
12. 采购人员根据售后情况独立处理 Purchase。
13. Sales After-sales 不强制驱动 Procurement 状态。
14. Delay / Lost 主要属于 Logistics Domain。
15. 物流问题产生正式 Return / Refund 后，再进入售后数据。
16. 售后发生后可以立即更新 Estimated Profit。
17. ERP 内售后阶段的财务影响只是估算。
18. Settlement / Recon 到达以后才形成准确财务结果。
19. After-sales Window 需要存在，但阶段 0 不定义具体天数。
20. 售后模块保持简单，优先解决“拉取、关联、识别待处理、退款、同步结果”。

---

## 16. 建模结论

当前团队真正需要的售后模块不是：

> 一个完整的客户服务系统。

而是：

> **把 Walmart 的售后订单拉回来，准确关联原销售订单，让运营知道哪些需要退款处理，并持续获取最终 Return / Refund 结果。**

核心关系：

```text
Sales Order
    ↓
Platform After-sales Data
    ↓
Return / Refund
    ↓
Operations Action
    ↓
After-sales Result
    ↓
Estimated Financial Impact
    ↓
Settlement / Recon
    ↓
Accurate Financial Result
```

因此 V1 售后模块应保持简单：

> **数据同步 + 订单关联 + 待处理识别 + Refund 操作 + 状态跟踪。**

复杂客户沟通、CRM、工单协作不属于当前核心范围。
