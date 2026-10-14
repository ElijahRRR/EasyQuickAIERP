# Walmart Platform Capability Specification v0.1

> 状态：进行中；本文仅包含当前已确认部分  
> 所属阶段：阶段 1 — ERP 地基  
> 平台：Walmart Marketplace  
> 目标：定义 Walmart 平台真实能力、旧系统已验证能力与 EasyQuickAIERP 产品目标之间的边界。

## 1. 证据与设计原则

Walmart Capability Specification 不以旧脚本能力作为平台能力上限。

判断顺序：

~~~text
Walmart 当前官方文档
        ↓
真实 Production / Sandbox 实测
        ↓
WalmartAPI-Contral 已实现能力与踩坑经验
        ↓
EasyQuickAIERP 产品目标
~~~

因此：

> 旧项目未实现某项 API，不代表 Walmart 不支持，也不代表新 ERP 不应支持。

旧仓库主要用于 Production Lessons、已验证接口行为、限流 / Feed / Retry / Error 经验，以及当前团队真实使用方式。

新 ERP 不复制旧仓库业务结构。

---

## 2. Walmart Store Connection

### 2.1 Store Identity 与 Credential 分开

必须区分：

~~~text
ERP Store Identity
Walmart Seller / Partner Identity
Credential
Connection Health
Platform State
~~~

Credential 轮换不改变 Store Identity。

Credential Invalid / Token Failed 只说明 Connection Health 异常，不自动推导 Store Suspended / Terminated。

### 2.2 Credential 安全存储

Store 只保存 Credential Reference。

Client Secret、Token 等敏感值由 Secret Management 安全保存。

### 2.3 Proxy / Fixed IP

Proxy / Fixed IP 属于 Store Connection Policy，不是 Walmart Store 的永久身份。

当前 Team 可以要求：

~~~text
1 Store
→ 1 Fixed Outbound Proxy / IP
~~~

但系统不能把这一 Team Policy 写死成所有 Walmart Team 的全局要求。

---

## 3. Walmart Platform Resources

Walmart Platform Resources 至少包括：

- Fulfillment Center / Ship Node；
- Shipping Template；
- Return Center；
- 其他 Settings / Fulfillment / Shipping Resource。

这些属于 Walmart 平台资源，不是 ERP 自有 Physical Warehouse / WMS。

### 3.1 Platform Resource Lifecycle

EasyQuickAIERP 的目标不是只读。

只要 Walmart 官方 API 支持，ERP 应能够按权限提供：

- Read；
- Create；
- Update；
- Delete / Deactivate（如平台支持）；
- Sync；
- Reference by Listing / Inventory / Shipping Operation。

旧脚本只实现 Read 不能成为 ERP 能力上限。

### 3.2 Fulfillment Center / Ship Node

ERP 应完整支持 Walmart Store 的多节点模型。

业务上需要能够：

- 同步全部节点；
- 创建节点；
- 更新节点；
- 查看节点状态；
- 在 Listing / Inventory Operation 中引用节点；
- 查看 SKU 在多个节点上的库存。

不能因为旧表格为了运营方便只显示一个节点库存，就把 ERP 数据模型简化成单仓。

例如：

~~~text
SKU X
Total Inventory = 25

Ship Node A = 10
Ship Node B = 8
Ship Node C = 7
~~~

ERP 可以同时展示总库存和节点级库存。

### 3.3 Shipping Template

Shipping Template 是 Walmart Platform Resource。

EasyQuickAIERP 应按照 Walmart 实际 API Capability 支持其完整可用生命周期，而不是只保存一个字符串引用。

如果某项能力平台只支持部分读取 / 引用 / 默认行为：

> ERP 如实表达 Partial Capability，不虚构不存在的能力。

### 3.4 Return Center

Return Center 同样属于 Walmart Platform Resource。

ERP 产品目标包括在 Walmart API 实际允许范围内查询、创建、更新 / 配置，并与适用业务对象建立平台引用。

具体 Endpoint、Account Eligibility 和限制在对应技术实现前再次核验。

### 3.5 Current Resource 与历史操作引用分开

当前 Platform Resource 可以变化。

但每次重要 Listing / Inventory / Shipping Operation 必须能够知道当时实际使用了哪个 Warehouse / Ship Node / Shipping Template 等资源。

~~~text
Current Platform Resource
≠
Historical Operation Resource Reference
~~~

---

## 4. Walmart Listing 上架的两种业务情况

Walmart Listing Submission 当前明确分成两种业务情况：

### 4.1 创建全新 Listing

表示以完整商品资料创建新的 Walmart Listing / Item Offer。

### 4.2 GTIN / UPC 匹配已有 Walmart Listing

即旧系统中的“跟卖”。

ERP 使用 GTIN / UPC 等标识匹配 Walmart 已有 Catalog Listing，并创建当前 Store 的销售关系 / Offer。

### 4.3 不额外建立第三种业务模式

系统可以自动帮助搜索 Walmart Catalog、推荐匹配、判断是否存在可跟卖对象。

但最终一次正式 Submission 必须明确属于：

~~~text
Create New Listing
或
Match Existing Listing by GTIN / UPC
~~~

两者统一属于 ERP Business Operation：Submit Listing。

Walmart Adapter 再执行对应平台能力。

---

## 5. Walmart Spec Versioning

Walmart Spec 不允许永久硬编码在 ERP 代码里。

需要作为版本化 Platform Capability Data 管理。

至少需要能够表达：

- Feed / Operation Type；
- Spec Version；
- Product Type；
- Attribute Requirement；
- Required / Optional；
- Allowed Value；
- Rule / Constraint；
- Observed / Effective Information。

### 5.1 Listing Validation / Submission 绑定 Spec Version

每一次 Listing Validation 与 Submission 必须记录当时实际使用的 Walmart Spec Version。

以后 Walmart 更新 Spec 时，不能用新 Spec 重解释历史 Submission。

### 5.2 Spec 更新采用人工维护

当前正式策略：

~~~text
Monitor New Walmart Spec
↓
Generate Alert / Developer Reminder
↓
Human Review
↓
Test
↓
Human Confirm / Release
↓
ERP Current Spec Version Update
~~~

禁止：

~~~text
发现新 Spec
→ 自动替换生产规则
~~~

ERP 可以监控并提醒，但 Spec 的正式升级由人工维护。

---

## 6. Final Listing Data Version

Listing 当前 Final Listing Data 必须支持版本语义。

例如：

~~~text
Listing L001

Content Version 1
→ Validation 1
→ Submission 1

Content Version 2
→ Validation 2
→ Submission 2
~~~

每次正式 Submission 必须能够还原当时实际提交的是哪一版 Final Listing Data。

产生新 Content Version 的典型变化包括：

- Title；
- Brand；
- Description；
- Images；
- Product Type；
- Attributes；
- GTIN / UPC；
- 其他实际改变 Walmart Listing Content Payload 的重要字段。

Price / Inventory 继续通过独立 Operation History 管理，不强制纳入 Content Version。

---

## 7. Walmart Feed

### 7.1 Feed 不是 Listing State

Feed 表示 Walmart 平台的一次异步提交执行记录。

~~~text
Feed Status
≠
Listing Platform State
~~~

Feed 完成也不等于 Feed 中所有 Listing 都成功。

### 7.2 一个 Business Task 可以拆成多个 Feed

例如：

~~~text
ERP Batch Listing Task #100
├ Walmart Feed A
├ Walmart Feed B
└ Walmart Feed C
~~~

当单个 Feed 无法完整承载一个业务任务时，Walmart Adapter 可以按照平台条数、大小、Feed Type、Rate Limit 等限制自动拆分。

拆分以后必须保留完整映射：

~~~text
ERP Business Task
→ Feed
→ Feed Item
→ Listing / Operation Target
~~~

不能因为拆 Feed 而失去明细对应关系。

### 7.3 一个 Feed 可以包含多个 Listing / Item

例如：

~~~text
Feed F001
├ Listing A → Success
├ Listing B → Failed
├ Listing C → In Progress
└ Listing D → Success
~~~

ERP 必须保留 Item-level Result。

不能只用 Feed-level 状态推断每条 Listing 的结果。

### 7.4 Feed History

需要保留：

- ERP Business Task Reference；
- Feed ID；
- Feed Type；
- Spec Version；
- Submitted Time；
- Submitted By / Actor；
- Feed Status；
- Received / Success / Failed / Processing Summary；
- Feed Item Result；
- Platform Error；
- Retry / Resubmission Relation；
- Correlation / Trace Information。

Feed History 受 Data Retention Policy 管理，不要求所有 Raw Payload 永久留在 PostgreSQL。

### 7.5 ERP Job 与 Walmart Feed 分开

~~~text
ERP Job
≠
Walmart Feed
~~~

一个 ERP Job 可以产生 0..N Walmart Feed。

Job 负责后台执行过程；Feed 负责 Walmart 平台提交事实。

---

## 8. Retire / Delete / Inventory Zero

三种能力必须完全分开。

### 8.1 Inventory Zero

中文：库存设为 0。

表示实际库存写操作。

它不等于 Listing 停用、Listing 删除或 Product Exit。

### 8.2 Retire

中文统一：**停用**。

停用后 Item / Listing 仍然留在 Walmart 后台及历史中，并存在恢复方向。

ERP 内部必须继续保留 Listing Identity 和历史。

### 8.3 Delete

中文统一：**删除**。

表示向 Walmart 发起平台删除请求。

Delete 与停用不同，属于高风险 Business Operation。

但是：

~~~text
Walmart Delete
≠
ERP Historical Record Delete
~~~

ERP 内 Listing、Order、Profit、Feed、Audit 等历史按 Data Retention Policy 管理。

### 8.4 HTTP Verb 不决定 ERP 业务语义

Walmart 某些 Retire Capability 在 HTTP 层可能使用 DELETE Method。

ERP 业务层仍必须叫 Retire / 停用。

不能因为 HTTP Method 是 DELETE 就把它映射成 ERP Delete Listing。

---

## 9. Price Model

Walmart Listing 的价格必须至少区分三层：

~~~text
Source Price
= 来源端当前价格，例如 Amazon 当前价格

ERP Target Price
= ERP / Team Policy 当前希望 Walmart 使用的目标售价

Walmart Observed Price
= ERP 最近一次真实查询 Walmart 后观察到的平台实际售价
~~~

因此：

> **Source Price ≠ ERP Target Price ≠ Walmart Observed Price**

例如：

~~~text
Source Price = 20.00
ERP Target Price = 39.99
Walmart Observed Price = 37.99
~~~

此时即使改价请求已经提交，ERP 也不能把 Walmart Platform Fact 直接改写成 39.99。

### 9.1 单个与批量 Price Operation

ERP 统一提供：

> Update Price

单个或批量操作具有同一业务语义。

Walmart Adapter 根据平台能力和批量规模，在执行前明确选择：

- Single-item Price API；
- Bulk Price Feed；
- 必要时把大批量任务自动拆为多个 Feed。

禁止采用：

> 单品接口失败后无记录地自动换 Feed 再写一次

这类隐式降级。

### 9.2 Price Operation History

价格操作至少需要能够还原：

- Listing / SKU；
- Before Observed Price；
- Requested / Target Price；
- Actor / Trigger；
- Submission Method；
- Feed / API Reference；
- Execution Result；
- Verification Status；
- Verified Observed Price；
- Verification Time；
- Error / Mismatch Reason。

Price Operation History 按 Data Retention Policy 管理。

### 9.3 Promotion

Walmart Promotion 属于 Pricing Capability，但本轮只正式确认普通售价的三层价格模型。

Promotion 的业务对象、活动周期、普通售价与活动售价关系在后续 Pricing Capability 细化时再单独确认，不与当前 Selling Price 静默混为一个字段。

---

## 10. Inventory Model

库存至少区分四类业务数据：

~~~text
Source Inventory
= 来源端真实 / 当前可售库存

Node Maximum Inventory Policy
= Team 对某 Store + Ship Node 可选配置的库存上限

ERP Target Node Inventory
= ERP 当前决定准备写给 Walmart 某节点的目标库存

Walmart Observed Node Inventory
= Walmart 实际查询得到的节点库存 Platform Fact
~~~

因此：

> **Source Inventory ≠ Maximum Inventory Policy ≠ ERP Target Inventory ≠ Walmart Observed Inventory**

### 10.1 Maximum Inventory 按 Ship Node 配置

原“Store Maximum Inventory”修订为：

> **Node Maximum Inventory Policy**

作用范围：

~~~text
Team
→ Store
→ Walmart Ship Node
→ Optional Maximum Inventory
~~~

该值：

- 不是必填；
- 由各 Team 自行配置；
- 不存在时，不由这条规则强制限制该节点库存；
- 可以不同 Node 使用不同上限；
- 属于 Store / Fulfillment Policy；
- 配置变化进入 Store Configuration History。

例如：

~~~text
Store H006

Node A Maximum = 10
Node B Maximum = 5
Node C Maximum = 未配置
~~~

具体多个节点如何共享 / 分配同一个 Source Inventory，属于后续 Node Allocation / Inventory Policy，不在本节写死。

### 10.2 多节点库存是原始 Platform Fact

ERP 必须保留 Walmart 每个 Ship Node 的库存明细。

例如：

~~~text
SKU ABC

Node A = 6
Node B = 4
Node C = 0

Displayed Total = 10
~~~

Total 可以是 ERP 为展示而汇总的值。

不能只保存 Total 而丢失 Node-level Platform Fact。

### 10.3 Inventory Operation

ERP 统一提供：

> Update Inventory

业务上可以：

- 更新 Default Ship Node；
- 更新指定单个 Ship Node；
- 一次更新一个 SKU 的多个 Ship Node；
- 批量更新多个 SKU × 多个 Ship Node。

Walmart Adapter 根据平台能力决定 Single API / Multi-node API / Feed。

### 10.4 Node-level Result

一次多节点更新可能产生不同 Node 的独立执行结果。

例如：

~~~text
Inventory Operation #100

Node A → Success
Node B → Success
Node C → Failed
~~~

因此总体 Operation 可以是 Partial Success。

开发不能只保存一个总的 Success / Failed 而丢失节点级结果。

---

## 11. Platform Effect Verification

对于 Price / Inventory 等会修改 Walmart Platform Fact 的操作：

> **API / Feed 请求成功，不等于平台最终值已经生效。**

必须区分：

~~~text
Requested / Target Value
↓
Submission / Execution Result
↓
Verification Pending
↓
Real Platform Read-back
↓
Confirmed / Still Pending / Mismatch / Verification Failed
~~~

### 11.1 Price 必须真实回读确认

Price Feed Item 或同步 API 显示成功以后：

> ERP 必须联动执行至少一次真实 Walmart 价格查询，读取平台当前值。

只有真实查询得到的 Observed Price 与 Target Price 符合预期时：

> Price Operation 才可以标记为 Effect Confirmed。

Feed Success 本身不能代替确认。

### 11.2 Inventory 同样以真实节点库存为最终平台事实

Inventory 写操作完成后，需要通过 Walmart Inventory Read Capability 获取节点实际库存。

对于多节点：

> Verification 必须能够落到对应 Ship Node。

### 11.3 回读不一致时的业务语义

如果真实 Read-back 与 Requested / Target Value 不一致：

> ERP 不能把该操作标记为 Effect Confirmed。

业务上应进入：

- Verification Pending / Not Yet Confirmed；
- 或在确认窗口结束后进入 Mismatch / Needs Attention。

具体：

- 多久再次查询；
- 是否自动重试；
- 重试多少次；
- Backoff / Queue / Idempotency；

属于开发技术可靠性设计，不由业务模型写死。

### 11.4 Requested Value 与 Observed Value 永远分开

例如：

~~~text
Before Observed = 5
Requested = 10
API Result = Success
First Read-back = 5
Later Read-back = 10
~~~

最终：

~~~text
Execution Result = Success
Effect Verification = Confirmed
Confirmed Observed = 10
~~~

历史必须保留整个闭环，而不是只留下最终 10。

---

## 12. Shipping Template / Mapping / Override

必须区分三个不同概念：

~~~text
Shipping Template
≠
SKU / Fulfillment Center Mapping
≠
Shipping Override
~~~

### 12.1 Shipping Template

Shipping Template 是 Store-level Walmart Platform Resource。

它表示配送规则本身。

ERP 在 Walmart API 实际支持范围内管理其：

- Read；
- Create；
- Update；
- Delete / Status；
- 其他平台提供的配置能力。

### 12.2 SKU / Fulfillment Center Mapping

Mapping 表示：

> 某个 Listing / SKU 当前与哪些 Walmart Shipping / Fulfillment Resource 建立关联。

它不是 Shipping Template 本身。

ERP 必须保存当前平台关联事实，并对重要 Mapping 变化保留 Operation History。

具体一个 SKU 可关联多少 Template / Fulfillment Center，由 Walmart 当前 API Capability 决定，通用 ERP 不自行发明基数限制。

### 12.3 Shipping Override

Shipping Override 表示：

> 某个 SKU 相对于 Shipping Template 的特殊配送例外。

因此：

~~~text
Template
= 通用配送规则

Mapping
= SKU 使用什么平台资源

Override
= SKU 的特殊例外
~~~

ERP 不能把 Override 塞进 Template 当前值中覆盖原模板。

### 12.4 Current Association 与历史分开

例如：

~~~text
Jan:
SKU ABC → Node A / Template 1

Mar:
SKU ABC → Node B / Template 2
~~~

当前平台关系可以是 Node B / Template 2。

但历史需要能够知道之前使用过 Node A / Template 1。

这些历史按照 Data Retention Policy 管理。

---

## 13. Ship Node / Shipping Template / Inventory 职责分开

三个概念回答不同问题：

~~~text
Ship Node
= 从哪里履约

Shipping Template
= 怎么配送

Inventory
= 这个节点当前有多少可售数量
~~~

因此开发不能把 Shipping Template 设计成 Inventory Record 的附属字段。

通用关系应支持：

~~~text
Listing
├ Platform Resource Associations
├ Shipping Template Association / Mapping
├ Shipping Overrides
└ Node Inventories
~~~

Walmart 的具体组合规则由 Walmart Adapter / Capability 层表达。

---

## 14. 单个与批量 Business Operation

EasyQuickAIERP V1 同时支持 Single Listing Operation 和 Batch Listing Operation。

业务语义保持一致，例如：

- Submit Listing(s)；
- Retire Listing(s)；
- Delete Listing(s)；
- Update Price；
- Update Inventory。

Walmart Adapter 根据平台能力选择 Single-item API、Bulk Feed、多节点 API 或自动切片成多个 Feed。

因此：

> Business Operation 不与 Walmart Endpoint 一一绑定。

---

## 15. Risk / Permission

Delete Listing 属于高风险操作。

至少要求：

- 独立 Action Permission；
- 明确 Actor；
- Business Validation；
- Audit；
- Walmart Result Tracking。

Workflow / AI / Digital Employee 只有在 Team 明确授予 Delete Permission 时才能执行。

Listing.Edit 等普通权限不能隐式包含 Delete。

Retire / 停用也应作为独立明确 Business Operation，不与 Inventory Zero 混用。

Price / Inventory 自动化同样必须使用正式 Business Operation 和 Actor / Permission Boundary。

---

## 16. Data Retention

Walmart Listing / Feed / Price / Inventory 历史遵守：

> [Data Retention Policy](./07-data-retention-policy.md)

当前默认：

- Listing 完整 Content Version：24 个月；
- Listing Submission / Feed Item 完整明细：24 个月；
- Walmart API Raw Payload / Response：12 个月；
- Price / Inventory Operation History：24 个月；
- Job 技术运行日志：6 个月。

这些为默认初始值，可由 Platform Admin 后续调整。

---

## 17. 当前已确认原则

1. 旧 WalmartAPI-Contral 只是已实现能力与 Production Lessons，不是 Walmart API 能力上限。
2. Credential、Store Identity、Connection Health、Platform State 分开。
3. Proxy / Fixed IP 属于 Store Connection Policy。
4. Walmart Platform Resource 可以在平台能力允许范围内 Read / Create / Update / Delete / Configure。
5. ERP 支持多 Ship Node / Warehouse 库存，不采用旧表格单仓展示作为数据模型。
6. Shipping Template / Return Center 属于 Walmart Platform Resource。
7. Listing 上架业务只有“创建全新 Listing”和“GTIN/UPC 匹配已有 Listing”两类。
8. Walmart Spec 版本化管理。
9. Spec 版本采用系统监控提醒 + 人工维护发布，不自动升级生产版本。
10. Listing Validation / Submission 绑定当时 Spec Version。
11. Final Listing Data 具有 Content Version。
12. Feed 不是 Listing State。
13. 一个 Business Task 可以自动拆成多个 Walmart Feed。
14. Feed Item 必须能对应具体 Listing / Operation Target。
15. Feed 历史与 Item-level Result 均需要保留。
16. ERP Job 与 Walmart Feed 分开。
17. Inventory Zero ≠ Retire / 停用 ≠ Delete / 删除。
18. Retire 中文统一为“停用”。
19. Delete 中文统一为“删除”，属于高风险业务动作。
20. Source Price、ERP Target Price、Walmart Observed Price 三层分开。
21. Maximum Inventory 按 Store + Ship Node 可选配置，由 Team 自行决定。
22. Source Inventory、Node Maximum、ERP Target Node Inventory、Walmart Observed Node Inventory 分开。
23. Walmart 多节点库存必须保存节点明细，总库存可以作为汇总展示值。
24. 多节点 Inventory Operation 必须保留 Node-level Result，并支持 Partial Success。
25. Shipping Template、SKU / Fulfillment Center Mapping、Shipping Override 三者分开。
26. API / Feed Success 不等于 Price / Inventory 已经实际生效。
27. Price Feed / API 成功后必须通过真实 Walmart Read-back 确认 Observed Price。
28. Inventory 最终平台事实同样来自真实 Node Inventory Read-back。
29. 第一次回读仍是旧值时不立即重复写，而是进入 Verification Pending 后按策略重查。
30. V1 同时支持单个和批量 Business Operation。
31. 历史明细按照 Data Retention Policy 管理，不要求主库无限期保存。

---

## 18. 当前待继续确认

### 18.1 Multi-node Inventory Allocation Policy

已经确认每个 Ship Node 可以拥有独立、可选 Maximum Inventory。

仍待确认：

> 当多个 Ship Node 共享同一个 Product Source Inventory 时，ERP 如何分配库存。

可能的 Team Policy 例如：

- Shared Pool：所有 Node 的 Target Inventory 合计不能超过 Source Inventory；
- Independent Mirror：每个 Node 独立参考 Source Inventory 和自己的 Maximum；
- Priority Allocation：按 Node 优先级依次分配；
- Custom Workflow：由 Team 自定义。

当前不写死。

### 18.2 Promotion

普通价格三层模型已经确认。

Promotion 的独立业务模型后续再确认。

---

## 19. Orders / Shipment / Cancellation / Returns / Refund

## 19.1 Order Sync 与 Acknowledge 分开

必须区分：

~~~text
Order Sync
= 从 Walmart 读取 Platform Order Fact

Acknowledge Order
= 向 Walmart 执行正式平台动作，表示 Seller 已收到订单并准备履约
~~~

因此：

> **收到订单 / 同步订单 ≠ 已经 Acknowledge。**

Walmart 平台决定 Acknowledge 的平台语义、允许条件和技术要求；ERP 只把它作为一个独立可调用的 Walmart Business Operation 提供。

### Acknowledge 不固定属于某个 ERP 流程步骤

正式确认：

> **EasyQuickAIERP 不把 Acknowledge 强制绑定在 Order Sync、Order Audit、Procurement 或 Shipment 的固定前后关系中。**

当前 Team 的真实操作方式是：

- 订单由人工处理；
- 人工在处理订单时执行 Acknowledge；
- 通常 Acknowledge 后会很快继续执行发货；
- 旧 WalmartAPI-Contral 当前并没有自动执行 Acknowledge。

这只是当前 Team 的人工操作习惯，不升级为 ERP 全局 Workflow。

如果未来某个 Team 希望自动化，则由该 Team 自己配置：

- 什么条件下可以 Acknowledge；
- 由 Human / Digital Employee / Workflow 中哪个 Actor 执行；
- Acknowledge 位于本 Team Workflow 的哪个步骤；
- 与 Order Audit、Procurement、Shipment 的先后关系；
- 异常时是否转人工处理。

因此：

~~~text
Acknowledge
≠ Order Audit Pass
≠ Purchase
≠ Shipment
~~~

并且：

~~~text
Acknowledge Timing
= Team Workflow / Human Operation Decision
~~~

### Actor 边界

Order Sync 可以由受限 System Sync Service 执行。

但 Acknowledge 是平台写操作，不由只读 System Sync Service 因“负责同步订单”而自动获得执行权限。

人工操作时由有权限的 Human Actor 执行；自动化时由被授权的 Digital Employee / Workflow 执行，并经过正式 Business Operation、Authorization 与 Audit。

---

## 19.2 Platform Order Fact

Walmart Order / Order Line 的平台事实由 Order Sync / Order Detail Read 获取。

至少要保持：

- Walmart Order Identity；
- Order Line Identity；
- SKU / Product Snapshot；
- Quantity；
- Amount；
- Customer / Shipping Snapshot；
- Platform Status；
- Promise / Delivery Information；
- Cancellation / Shipment / Refund 等平台事实；
- First Seen / Last Synced；
- 必要 Raw Platform Evidence。

这些 Platform Fact 与 ERP Order Audit / Procurement Decision 分开。

---

## 19.3 Confirm Shipment

Confirm Shipment 是独立 Walmart Business Operation。

它不依赖 Source Purchase / Source Shipment 的系统级固定顺序。

一个 Order Line 底层允许：

~~~text
0..N Platform Shipment
~~~

用于支持：

- Partial Shipment；
- Multiple Package；
- Tracking Replacement；
- 其他平台允许的多 Shipment 情况。

### Shipment Effect Verification

当前正式确认：

> **Shipment API / Feed 成功不等于 Walmart Order Line 已经实际进入 Shipped Platform State。**

闭环：

~~~text
Shipment Requested
↓
Walmart API / Feed Result
↓
Verification Pending
↓
真实读取 Walmart Order / Order Line
↓
Platform Shipment / Line State Confirmed
~~~

只有真实回读确认平台状态符合预期，才标记：

> Effect Confirmed

如果真实回读尚未显示预期 Shipped 状态，则业务上保持 Verification Pending / Not Confirmed；后续重查与技术重试策略由开发设计。

---

## 19.4 Cancellation

Cancellation 以 Order Line 为主要平台操作粒度。

不能只在 Sales Order 顶层维护一个简单 Cancelled 状态。

必须区分：

~~~text
ERP Cancellation Decision
≠
Cancellation Requested to Walmart
≠
Walmart Observed Cancelled Fact
~~~

一个 Sales Order 可以出现：

~~~text
Line A = Shipped
Line B = Cancelled
Line C = Active
~~~

### Cancellation Effect Verification

正式闭环：

~~~text
Cancel Order Line Requested
↓
Walmart Execution Result
↓
Verification Pending
↓
真实重新读取 Order / Order Line
↓
Platform Cancellation Confirmed / Mismatch
~~~

API 请求成功本身不能作为最终平台取消事实；如果真实回读尚未显示 Cancelled，则业务上保持 Not Confirmed / Mismatch，具体技术重查与重试由开发设计。

---

## 19.5 Refund

ERP 使用统一 Refund 业务对象。

~~~text
Refund
├ Order Line = required
└ Return Reference = optional
~~~

原因是 Walmart 存在两类退款路径：

### Order Refund

~~~text
Order / Order Line
↓
Refund
~~~

不要求存在 Return。

### Return Refund

~~~text
Return Order
↓
Refund
~~~

可与 Return 建立关联。

因此：

> **业务上只有统一 Refund；Walmart Adapter 记录实际 Execution Route。**

概念上可表达：

~~~text
Refund Execution Route
= Order Refund
或
= Return Refund
~~~

不能把两个 Walmart Endpoint 设计成两个互不兼容的 ERP Refund 业务对象。

Return 与 Refund 仍然保持：

~~~text
Return ≠ Refund
~~~

---

## 19.6 Returns Capability

EasyQuickAIERP 的 Walmart Returns 能力不以旧脚本当前只读实现为上限。

### Seller-Fulfilled Return

在 Walmart 当前实际支持范围内，ERP 目标能力包括：

- View / Sync；
- Approve；
- Reject；
- Refund。

其中 Approve / Reject / Refund 都属于平台写操作，需要：

- Human / Digital Actor；
- Permission；
- Business Validation；
- Audit；
- Platform Result Tracking。

### WFS Return

WFS Return 当前按照：

> **Read / Sync Only**

处理。

ERP 不向 WFS Return 暴露 Seller 不具备的平台写能力。

Capability 层必须能够根据 Fulfillment Type / Return Type 表达：

~~~text
Readable
Writable
Approvaable
Rejectable
Refundable
~~~

而不是对所有 Return 显示同一组按钮。

---

## 19.7 Return / Refund Platform Fact 与 ERP Operation 分开

Returns Sync 获取的是：

> Walmart 当前 Return / Refund Platform Fact。

ERP 主动执行的：

- Approve Return；
- Reject Return；
- Refund；

属于 Business Operation。

因此：

~~~text
Platform Return Fact
≠
ERP Return Decision / Action

Platform Refund Fact
≠
ERP Refund Request
~~~

主动操作完成后仍需要后续平台同步 / 回读，把最终 Walmart Platform Fact 写回。

---

## 19.8 Orders Capability 的 Actor 原则

推荐执行身份：

~~~text
System Sync Service
→ Order / Return / Refund Platform Fact Read / Sync

Human / Digital Employee
→ Acknowledge
→ Confirm Shipment
→ Cancel Order Line
→ Approve Return
→ Reject Return
→ Refund
~~~

System Sync Service 不能因为负责同步订单，就获得所有订单写权限。

如果某个 Team 配置自动 Acknowledge，可由被授权的 Digital Employee / Workflow 执行；当前 Team 仍以人工操作为准。

---

## 20. 当前新增确认原则

32. Order Sync 与 Acknowledge 是两个不同动作。
33. Acknowledge 是独立 Walmart Business Operation，不固定绑定在 Order Sync、Order Audit、Procurement 或 Shipment 的某一步。
34. 当前 Team 由人工在处理订单时执行 Acknowledge，通常随后很快发货；旧 WalmartAPI-Contral 当前不自动执行 Acknowledge。
35. 如果 Team 希望自动 Acknowledge，由该 Team 自行定义触发条件、Actor 和 Workflow 顺序。
36. Acknowledge 不等于 Audit Pass、Purchase 或 Shipment。
37. System Sync Service 负责读 / 同步 Platform Fact；Acknowledge 等写操作由有权限的 Human / Digital Actor 执行。
38. Shipment 成功请求后必须真实回读 Walmart Order / Order Line 确认平台 Shipped 状态。
39. Cancellation 以 Order Line 为主要操作粒度，并通过真实回读形成 Effect Verification 闭环。
40. Refund 使用统一 ERP 业务对象，Walmart Adapter 区分 Order Refund / Return Refund Execution Route。
41. Refund 可以没有 Return；Return 与 Refund 仍是两个独立对象。
42. Seller-Fulfilled Return 目标能力包括 View / Approve / Reject / Refund。
43. WFS Return 当前按 Read / Sync Only 处理。
44. Return / Refund Platform Fact 与 ERP 主动操作请求分开。
45. Returns / Orders 写操作必须经过 Actor / Permission / Audit 边界。

---

## 21. Settlement / Reports / Performance / Store Platform State

## 21.1 Recon Report 是数据来源，不是 ERP 财务对象

正式确认：

> **Walmart Recon Report 只是外部数据来源；Settlement Period + Settlement Entry 才是 ERP 正式财务业务记录。**

因此：

~~~text
Walmart Recon Report
↓ Parse / Import
Settlement Period
+
0..N Settlement Entry
~~~

Walmart 后续如果改变 Report Format / Version，不应迫使 Finance Domain 改变业务对象语义。

Settlement Entry 仍需要支持：

- Order Line 可匹配；
- Order Line 无法匹配；
- Store-level Fee；
- Adjustment；
- Period Charge；
- Refund / Commission / Tax 等平台财务明细。

无法匹配 Order Line 的 Entry 仍必须保留。

---

## 21.2 Payment Statement 是资金 / 付款周期 Snapshot

Payment Statement 表示：

> 某个时间点 Walmart 返回的账户资金 / 付款周期观察结果。

它可以用于：

- 当前预计回款查看；
- 付款周期状态；
- 现金流观察；
- Store Financial State 展示。

但：

> **Payment Statement Snapshot 不能按每天数值累加计算 Cumulative Payout。**

正式累计 Payout 仍然使用：

~~~text
Cumulative Payout
=
Σ Settlement Period.Total Payable
~~~

因此必须区分：

~~~text
Payment Statement Snapshot
≠
Settlement Period
≠
Settlement Entry
≠
Cumulative Payout
~~~

---

## 21.3 Platform Report Request 与 ERP Job 分开

ERP 可以主动向 Walmart 创建 Report Request。

需要区分：

~~~text
ERP Job
≠
Walmart Platform Report Request
≠
Downloaded Raw Report
≠
Parsed Business Record
~~~

例如：

~~~text
ERP Settlement Sync Job
↓
Walmart Report Request
↓
Report Ready
↓
Download Raw Report
↓
Parse
↓
Settlement Period / Settlement Entry
~~~

ERP Job 后续清理不能删除已经形成的正式财务业务数据。

### Platform Report Request 需要保存的业务信息

至少包括：

- Store；
- Report Type；
- Report Version；
- Requested By；
- Requested Time；
- Walmart Request ID；
- Platform Request Status；
- Ready Time；
- Download Time；
- Parse Result；
- Related Import / Business Result；
- Error / Failure Reason。

Raw Report 文件受 Data Retention Policy 管理，可以比解析后的正式 Business Fact 更早 Archive / Purge。

---

## 21.4 Walmart Report Scheduler 当前不进入 ERP V1

当前明确收缩范围：

> **EasyQuickAIERP V1 不需要管理 Walmart Report Scheduler。**

ERP 当前只需要管理：

~~~text
ERP / Human / Workflow
→ Create Walmart Report Request
→ Poll / Receive Status
→ Download
→ Parse
→ Persist Business Result
~~~

不为“以后可能会用”提前增加：

- Walmart Report Schedule Object；
- Schedule CRUD；
- Scheduler 与 ERP Workflow 双层调度关系。

如果未来出现真实业务需要，再扩展 Platform Capability。

因此：

> **当前不建模 Walmart Report Scheduler。**

---

## 21.5 Performance 使用通用 Metric 模型

Store Performance 不把当前 Walmart 某一组固定指标永久写死成数据库核心结构。

正式采用概念：

~~~text
Performance Metric Definition
+
Performance Observation
+
Problem Detail
~~~

### Metric Definition

表示：

- Metric Key；
- Platform Name；
- Display Name；
- Unit；
- Threshold / Target（如平台可获取）；
- Effective Version；
- Deprecated / Active；
- Platform Metadata。

### Performance Observation

表示某次真实平台观察，例如：

~~~text
Store H006
Metric = VTR
Observed At = T
Period / Window = X
Value = 99.2%
~~~

### Problem Detail

表示导致某个指标异常的具体平台明细，例如：

- affected Order；
- affected Order Line；
- issue type；
- platform reason；
- observed time。

因此：

~~~text
Performance Summary
≠
Performance Problem Detail
~~~

ERP 可以从 Summary 下钻到具体问题订单 / 明细。

---

## 21.6 Performance 不能自动推导 Store Platform State

低 OTD、低 VTR、高 Cancellation、Return 等指标只表示：

> Store 当前绩效表现。

不能自动推导：

~~~text
Store = SUSPENDED
或
Store = TERMINATED
~~~

正式保持：

~~~text
Seller Performance
≠
Platform State
~~~

只有明确的 Walmart Platform Evidence 才能形成 Platform State Observation。

---

## 21.7 Store 的五类状态 / 观测必须分开

正式确认以下五类数据不能压缩成一个万能 Store Status：

### 1. Platform State

回答：

> 这个 Store 当前在 Walmart 平台上的账号业务状态是什么？

例如平台明确可观察到的：

- Active；
- Suspended；
- Terminated；
- Restricted；
- Unknown。

具体平台枚举后续以实际 Walmart Capability 为准。

### 2. Connection Health

回答：

> ERP 当前能否正常连接并调用 Walmart API？

例如：

- Healthy；
- Credential Invalid；
- Token Failed；
- Permission / Scope Missing；
- Proxy Failed；
- API Unreachable；
- Unknown。

### 3. Seller Performance

回答：

> Store 当前经营绩效如何？

例如 OTD / VTR / Cancellation / Return 等 Metric Observation。

### 4. Seller Program Status

回答：

> Store 当前参加 Walmart 平台项目 / 等级 / Badge 的状态是什么？

例如 Pro Seller / Badge / Incentive 等未来实际支持的数据。

### 5. Financial State

回答：

> Store 当前资金 / 账期 / 回款处于什么状态？

例如：

- Payment Statement Snapshot；
- Current Settlement Period；
- Current Payable Observation；
- 其他财务观察值。

因此：

~~~text
Platform State
≠
Connection Health
≠
Seller Performance
≠
Seller Program Status
≠
Financial State
~~~

---

## 21.8 Platform State 必须保存 Evidence Source

Platform State 不能只保存一个当前字符串。

每次重要状态观察至少需要能够知道：

- Store；
- Observed Platform State；
- Evidence Source；
- Observed Time；
- Raw / Reference Evidence（如适用）；
- Confidence / Confirmed Source（如需要）。

Evidence Source 可以来自：

- Walmart API；
- Seller Center Observation；
- Human Confirmed；
- 其他明确可信的平台来源。

例如：

~~~text
State = SUSPENDED
Evidence Source = Seller Center Observation
Observed At = T
~~~

而不能：

~~~text
API 401
↓
推断 SUSPENDED
~~~

---

## 21.9 Authentication / Connection Failure 不得推断平台封禁状态

正式确认：

~~~text
401 / 403
Credential Invalid
Token Failed
Scope Missing
Proxy Failure
API Timeout
~~~

这些首先属于：

> Connection Health

不能自动推导：

- Suspended；
- Terminated；
- Restricted。

如果平台账号状态无法明确获取，则 Platform State 保持 Unknown / Last Confirmed，而不是猜测。

---

## 21.10 Platform State History

Store 当前 Platform State 可以更新。

但重要历史观察应保留，例如：

~~~text
Oct 1  ACTIVE
Oct 8  SUSPENDED
Oct 10 ACTIVE
~~~

这样才能解释：

- 销售中断；
- Listing 异常；
- Order 下降；
- 财务变化；
- 运营责任区间。

历史按 Data Retention Policy 管理。

---

## 22. 当前新增确认原则

46. Recon Report 是外部数据来源；Settlement Period / Settlement Entry 才是 ERP 正式财务记录。
47. Payment Statement 保存为资金 / 付款周期 Snapshot，不通过每日 Snapshot 累加计算 Payout。
48. Cumulative Payout 仍来自正式 Settlement Period Total Payable 的累计。
49. ERP Job、Walmart Report Request、Raw Report、Parsed Business Record 四者分开。
50. ERP V1 只管理 ERP 主动发起的 Walmart Report Request，不管理 Walmart Report Scheduler。
51. Raw Report 可以按 Retention Policy 比解析后的核心业务记录更早归档 / 清理。
52. Seller Performance 使用 Metric Definition + Observation + Problem Detail 的通用模型。
53. 当前 Walmart 绩效指标不硬编码成永久数据库结构。
54. Performance 不能自动推导 Store Suspended / Terminated。
55. Platform State、Connection Health、Seller Performance、Seller Program Status、Financial State 五类完全分开。
56. Platform State Observation 必须带 Evidence Source + Observed Time。
57. 401 / 403、Credential / Token / Proxy 等连接异常不能推断 Store Suspended。
58. Store Platform State 的重要历史观察需要保留。

---

## 23. Walmart Business Result Handling

本节只确认：

> Walmart 返回 / 观察到某种业务结果以后，ERP 应如何理解，以及业务下一步允许做什么。

以下内容不在业务侧决定：

- HTTP Retry 次数；
- Backoff / Jitter；
- Rate Limit Bucket；
- Worker / Queue；
- 幂等技术实现；
- 网络超时后如何做传输层补偿。

这些属于开发技术设计与评审。

### 23.1 Listing / Content / Identifier Result

正式确认：

#### 可修复资料问题

例如：

- Required Attribute 缺失；
- Spec 不满足；
- Product Type / Category 错误；
- 可修复的 GTIN / UPC / Mapping 问题。

业务结果：

~~~text
Recoverable Listing Issue
→ 生成待处理项
→ 修正资料
→ 允许重新提交
~~~

#### Policy / Prohibited / IP / Brand Restriction

进入：

> Risk / Compliance Resolution

不进入普通“自动修字段再重上”路径。

#### Unknown Platform Error

保存 Walmart Raw Error，并进入：

> Unknown / Needs Classification

不猜测根因，不自动执行破坏性动作。

### 23.2 Price Result Mismatch

如果：

~~~text
ERP Target Price ≠ Walmart Observed Price
~~~

则：

- Walmart Observed Price 仍作为当前 Platform Fact；
- ERP Target Price 仍作为目标值；
- 该操作标记为 Not Confirmed / Mismatch；
- 产生 Price Exception / Needs Attention；
- 是否再次执行由 Team Policy / Workflow 决定。

批量操作中其他已确认成功的 Listing 不因个别价格异常被整体判失败。

### 23.3 Multi-node Inventory Result

Inventory Result 按 Ship Node 独立判断。

例如：

~~~text
Node A → Confirmed
Node B → Mismatch
Node C → Confirmed
~~~

整体 Operation 可以显示 Partial Success / Partial Confirmed。

已成功节点事实不因其他节点异常而被回滚或覆盖。

Node Maximum Inventory 未配置时：

> 不应用该 Maximum Policy。

未配置不等于 Maximum = 0。

### 23.4 Explicit Platform Resource Selection

如果某次操作明确选择了：

- Ship Node；
- Shipping Template；
- Return Center；
- 其他 Walmart Platform Resource；

而该 Resource 已失效 / 不可用：

> 该操作不能静默切换到另一个 Resource 或 Walmart Default。

只有用户 / Team Policy 明确选择：

> Use Platform Default

时，才允许走平台默认行为。

因此：

~~~text
Explicit Resource Selection Failure
≠
Silent Fallback to Default
~~~

### 23.5 ERP Decision 与 Walmart Execution Result

对于：

- Shipment；
- Cancellation；
- Refund；
- Return Approve / Reject；
- Acknowledge；
- Platform Resource Change；
- 其他 Walmart 写操作；

必须保持：

~~~text
ERP Decision / Requested Action
≠
Walmart Execution Result
≠
Observed Platform Fact
~~~

例如：

~~~text
ERP Cancellation Decision = Cancel
Walmart Result = Rejected
Observed Order Line = Shipped
~~~

ERP 必须保存真实结果并进入运营异常处理，不能因为 ERP 想取消就把平台事实改成 Cancelled。

---

## 24. Suspended / Terminated Business Handling

### 24.1 SUSPENDED

只有在 Walmart Platform State 已被可靠 Evidence 明确确认为 SUSPENDED 时，应用以下业务处理。

ERP 停止主动经营写操作：

- 新 Listing；
- Listing 批量维护类主动写操作；
- Price 自动经营性更新；
- Inventory 自动经营性更新。

ERP 继续：

- Order Sync；
- Return / Refund Sync；
- Settlement；
- Performance；
- Store State Sync；
- Historical Query。

现有订单仍可能来自暂停前已经产生的订单，需要继续管理。

当前明确允许的订单 / 售后侧处理包括：

- Cancel；
- Refund；
- 必要 After-sales。

其他既有订单动作是否可执行，以 Walmart 当时实际 Platform Capability 和 Team 操作为准。

这里不需要 ERP 另外“阻止新增销售”：

> Store 被 Walmart SUSPENDED 后，平台本身已经限制销售能力。

ERP 要做的是停止主动新增 / 经营性写操作，而不是伪造一个平台销售开关。

### 24.2 TERMINATED

当 Platform State 被可靠 Evidence 明确确认为 TERMINATED：

> 停止所有新的销售经营动作。

同时，如果 Team 使用 Store-scoped Brand / Product Occupancy / Allocation Claim：

> **自动释放该 Store 占用的 Brand 和 Product。**

该释放属于 ERP 内部业务动作，不是 Walmart Platform Fact。

仍然必须保留：

- Store History；
- Listing History；
- Existing Orders；
- Return / Refund；
- Settlement；
- Finance；
- Audit；
- Historical Assignment。

### 24.3 Terminated 后 API 能力不预设

当前不能确认：

> Store TERMINATED 后 Walmart API 是否仍然保持可用，以及哪些接口还能使用。

实际业务中通常可能出现 Credential / API 很快失效。

因此 ERP 不预设：

~~~text
TERMINATED
→ API 一定还能用
~~~

也不预设：

~~~text
API 失效
→ 必然是 TERMINATED
~~~

正式处理：

- Platform State 继续保存已确认的 TERMINATED Evidence；
- Connection Health 独立记录 API 是否仍可访问；
- 如果 API 仍允许既有订单 / 售后 / Settlement 操作，则继续处理；
- 如果 API 已不可用，则保留内部历史与待处理责任，不伪造平台操作已完成。

### 24.4 SUSPENDED 不自动释放 Brand / Product Occupancy

当前只确认：

> TERMINATED 自动释放 Store-scoped Brand / Product Occupancy。

SUSPENDED 不自动释放，避免短期暂停导致经营占用关系无意被其他 Store 抢占。

---

## 25. 当前新增确认原则

59. Listing 的资料 / Spec / Category 等可修复错误进入待修复处理，修复后允许再次提交。
60. Policy / Prohibited / IP / Brand Restriction 进入 Risk / Compliance Resolution，不按普通资料错误处理。
61. Unknown Walmart Error 保存原文并进入待分类，不猜测处理。
62. Price Target 与 Observed 不一致时，以 Observed 为 Platform Fact，并产生 Price Exception。
63. Multi-node Inventory 按 Node 独立确认，允许 Partial Success / Partial Confirmed。
64. Node Maximum Inventory 未配置不等于 0。
65. Explicit Platform Resource 不可用时不得静默退回 Default。
66. ERP Decision、Walmart Execution Result、Observed Platform Fact 三者分开。
67. SUSPENDED 时停止新 Listing、Listing 主动批量维护、自动 Price / Inventory 经营性更新。
68. SUSPENDED 后继续 Order / Return / Refund / Settlement / Performance / Store State Sync 与历史查询。
69. SUSPENDED 下现有订单仍需处理；当前明确保留 Cancel、Refund 与必要 After-sales 能力。
70. TERMINATED 时停止所有新的销售经营动作。
71. TERMINATED 时自动释放该 Store 的 Brand / Product Occupancy（如 Team 使用该占用能力）。
72. SUSPENDED 不自动释放 Brand / Product Occupancy。
73. TERMINATED 后 Walmart API 是否仍可用不做平台能力假设；Platform State 与 Connection Health 继续分开。
74. 即使 TERMINATED 后 API 已不可用，ERP 仍保留 Existing Orders / After-sales / Settlement / Finance / Audit 等历史责任。

---

## 26. 技术可靠性设计边界

以下属于开发负责，不需要业务方逐项批准：

- HTTP Retry；
- Retry Count；
- Backoff；
- Jitter；
- Rate Limit 调度；
- Feed 技术防重；
- Correlation ID 生成与传递；
- Queue / Worker；
- 网络 Unknown Outcome 的技术查证方法；
- Idempotency 技术方案。

业务侧只要求开发最终保证：

> **不能因为技术失败或重试机制，伪造业务成功、覆盖 Platform Fact、或者制造重复业务副作用。**

这些内容进入后续 Technical Design Review，而不是继续由业务方决定实现算法。

---

## 27. 下一步

Walmart Capability v0.1 还剩最后几个业务范围需要确认：

- Settlement 异常如何进入人工核对；
- Performance 异常只是告警，还是允许 Team 配自动动作；
- Risk / Compliance 命中后在在线 Listing 上的默认处理；
- Webhook 是否进入 V1 产品范围；
- 最终 Walmart Capability Matrix 与 v0.1 收口。
