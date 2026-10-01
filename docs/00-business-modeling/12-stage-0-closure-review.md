# 阶段 0 收口审查 — Business Model Closure Review v1.0

> 状态：已收口（Baseline v1）  
> 日期：2026-10-01  
> 范围：`docs/00-business-modeling`

## 1. 收口目标

阶段 0 的目标是：

> 在技术设计之前，确认 EasyQuickAIERP 当前真实业务中的核心对象、生命周期、边界、Team Policy 与 Platform Fact。

本次收口审查重点检查：

1. 已有业务模型之间是否存在冲突；
2. 是否把平台特有资源误建成 ERP 通用业务对象；
3. 是否遗漏当前真实业务依赖的核心横向模型；
4. 是否存在已经被后续讨论推翻但仍残留在早期文档中的旧结论。

---

## 2. 当前已形成的核心业务模型

阶段 0 已覆盖：

1. Product Lifecycle；
2. Listing Lifecycle；
3. Platform Error Management；
4. Order Lifecycle；
5. Procurement Lifecycle；
6. Logistics Lifecycle；
7. After-sales Lifecycle；
8. Reconciliation & Profit Lifecycle；
9. Store Lifecycle；
10. Team / User / Permission / Organization；
11. Risk Intelligence / Blacklist。

这些模型已经能够覆盖当前核心链路：

```text
Team / Permission
        ↓
Store
        ↓
Product / Source
        ↓
Listing
        ↓
Sales Order / Order Line
        ├── Order Audit
        ├── Procurement
        ├── Platform Shipment / Logistics
        ├── Return / Refund
        └── Settlement / Profit
```

并由 Risk Intelligence 横向服务：

- Product Audit；
- Online Risk Monitoring；
- Source / Procurement Selection；
- Workflow Decision。

---

## 3. 本次收口统一修正的关键冲突

### 3.1 不建立当前没有真实业务需求的 Physical Warehouse / WMS Domain

当前 Walmart Warehouse 属于：

> Platform Resource。

ERP 需要同步并保存 Platform Warehouse Reference，并在 Listing / Inventory Operation 中正确引用。

当前确认：

- Listing 上架可以显式选择 Walmart Warehouse；
- 未选择时使用 Walmart Default Warehouse；
- Shipping Template 可以显式选择；
- 未指定时由 Walmart 根据最终 Warehouse 地区使用默认模板；
- Inventory Update 可以显式指定 Warehouse；
- 未指定时维护 Default Warehouse Inventory；
- Inventory Update 可以同时调整 Shipping Template。

这些不代表 ERP 已经拥有：

- 入库；
- 出库；
- 调拨；
- 库位；
- 盘点；
- Physical Warehouse Inventory；
- WMS。

未来真正出现实体仓储履约需求时，再独立建立 Warehouse / Inventory / Fulfillment Domain。

---

### 3.2 Store Platform State 与 Connection Health 分开

正式统一：

```text
Platform State
≠
ERP Enable State
≠
Connection Health
```

因此：

- Token Failed；
- Credential Invalid；
- Authentication Failed；
- API Permission Missing；

不能自动推断：

- SUSPENDED；
- TERMINATED。

---

### 3.3 Order Audit 正式结果闭合

Order Audit 当前正式结果为：

```text
Pass
Reject
Pending
```

Pending 可以表示：

- Machine / LLM 无法可靠判断商品一致性，需要人工复核；
- 当前暂时无货，需要等待 X 天并定期重新检查；
- 其他暂时缺少足够证据的情况。

因此：

> Pending ≠ Manual Review Only。

---

### 3.4 Order 不再使用单一线性业务状态

不再采用：

```text
New
→ Purchased
→ Shipped
→ Delivered
→ Completed
```

作为 ERP 全局 Order Lifecycle。

正式统一为：

```text
Sales Order / Order Line
│
├── Platform Order Fact
├── Order Audit Decision
├── Procurement
├── Platform Shipment / Logistics
├── Return / Refund
└── Settlement / Profit
```

这些业务对象可以同时存在不同状态。

---

### 3.5 Procurement 与 Platform Shipment 解耦

正式统一：

> Purchase、Source Shipment、Platform Shipment 不存在 ERP 全局固定先后关系。

因此不硬编码：

```text
Purchase Shipped
→ Walmart Confirm Shipment
```

是否需要前置条件由 Team Workflow 决定。

---

### 3.6 After-sales 不做当前业务不存在的 CRM

当前 After-sales 保持：

```text
Pull Return / Refund
→ Link Sales Order / Order Line
→ Show Pending
→ Operations Refund
→ Sync Result
```

当前不把：

- Walmart Message；
- Email；
- 完整 Customer Conversation；
- Waiting Customer / Replied 等客服状态；

作为 V1 核心售后对象。

---

### 3.7 财务不建立复杂 Final Profit 状态机

正式统一核心财务值：

- Estimated Profit；
- Current Reconciled Profit；
- Store Payout。

不维护：

```text
First Reconciled
→ Adjusted Reconciled
→ Final Profit
```

独立状态机。

后续新 Settlement / Refund / Adjustment 到达时：

> 持续更新 Current Reconciled Profit。

---

### 3.8 Organization、Assignment、Permission 分开

正式统一：

```text
Organization
≠
Assignment
≠
Permission
≠
Role Template
```

其中：

- Team 是租户隔离边界；
- Group 只是成员集合，不自动带权限；
- Store Assignment 表达责任归属；
- Permission 决定 Actor 实际能做什么；
- Role Template 只是可复用 Permission Preset。

权限核心结构为：

```text
Actor
→ Platform
→ Store Scope
→ Function
→ Action
```

Human 与 Digital Employee 使用同一授权边界。

---

### 3.9 Team Administrator 作为最高管理身份

当前不区分 Team Owner 与 Team Administrator。

Team 首个创建者：

> 默认成为该 Team 的 Administrator。

一个 User Account 永远只属于一个 Team。

---

### 3.10 Risk Intelligence / Blacklist 独立于 Audit

Risk Intelligence 正式成为横向风险资料层。

当前主要对象：

- Brand；
- Product / ASIN；
- Amazon Seller。

正式优先级：

```text
Team Whitelist
>
Team Private Blacklist
>
System Public Blacklist
```

Audit 可以查询 Blacklist，但：

> Audit Reject 不自动创建 Blacklist Entry。

在线商品命中新风险后，当前团队默认先产生运营建议；自动下架属于 Team Workflow。

---

## 4. 阶段 0 的核心不变量

后续阶段设计不得无意破坏以下业务边界：

### Product ≠ Listing

Product 是团队经营商品资产。

Listing 是 Product 在某 Platform / Store 的销售实例。

### Product Source ≠ Actual Procurement Source

商品主来源只是默认来源。

历史订单成本使用真实 Purchase Fact。

### Sales Order ≠ Purchase

一个 Order Line 可以有 0..N Purchase。

### Purchase ≠ Platform Shipment

二者可以关联，但不存在固定一一对应或固定时间顺序。

### Source Tracking ≠ Platform Tracking

两者可以有关联，也可以独立存在。

### Return ≠ Refund

Refund-only 必须能够独立存在。

### Platform State ≠ ERP State

平台事实不能被 ERP 内部流程状态替代。

### Estimated Profit ≠ Current Reconciled Profit ≠ Payout

三者回答不同财务问题。

### Organization ≠ Assignment ≠ Permission

组织关系、业务责任和系统授权必须分开。

### Platform Resource ≠ ERP-owned Business Entity

Walmart Platform Warehouse / Shipping Template 等属于平台资源引用，不自动升级为 ERP 内部 Warehouse Domain。

---

## 5. 当前明确不进入阶段 0 的业务域

以下内容当前不阻塞 ERP 基础设计，因此不为了追求“大而全”提前建模：

- Physical Warehouse / WMS；
- 完整 CRM / Customer Message Center；
- 总账 / 完整会计系统；
- 人工成本与企业固定成本分摊；
- Dashboard / KPI 作为独立生命周期；
- Proxy Infrastructure 作为独立业务域；
- Workflow Engine 的技术实现；
- AI Agent 的技术实现。

未来业务真实出现时再扩展。

---

## 6. Platform Capability 与通用业务模型分开

阶段 0 只确认通用业务语义。

以下内容应进入后续 Platform Capability Specification：

- Walmart API / Feed 的具体接口；
- Platform Warehouse 的技术标识和默认仓解析；
- Shipping Template 参数；
- 字段是否可修改；
- Retire / Delete 的平台真实技术行为；
- Return / Refund 具体状态枚举；
- Carrier / Tracking 平台要求；
- Platform Error Code Mapping；
- 其他 Walmart / Amazon / eBay 等平台特有能力。

原则：

> 平台技术限制不能污染通用 ERP Business Model，但 Platform Adapter 必须准确表达真实能力。

---

## 7. 阶段 0 收口结论

当前阶段已经达到进入技术设计所需的业务完整度。

因此：

> **阶段 0：业务建模，正式标记为 Closed / Baseline v1。**

Closed 不表示以后永远不能改。

如果后续真实业务产生新事实：

1. 先修改对应业务模型；
2. 保留 Git 历史；
3. 再调整技术实现。

阶段 0 Baseline v1 从现在开始作为：

> **阶段 1 ERP 地基设计的业务事实来源。**

---

## 8. 下一阶段

下一步进入：

> **阶段 1：ERP 地基**

阶段 1 才开始讨论技术结构，例如：

- Tenant / Team 数据隔离；
- Identity 与稳定业务 ID；
- Platform Adapter / Capability Boundary；
- Business Operations；
- Permission Enforcement；
- Actor / Audit Log；
- Credential / Secret Management；
- Task / Job / Async Operation 基础能力；
- 模块边界与数据所有权；
- 为后续 Workflow / AI 保留统一调用接口。

具体技术方案需要重新讨论和确认，不能因为阶段 0 的业务概念而直接推导数据库表结构。
