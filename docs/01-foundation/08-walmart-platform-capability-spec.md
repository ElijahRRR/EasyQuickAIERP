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

### 11.3 第一次回读不一致时不立即重复写

平台状态可能存在传播延迟。

因此第一次 Read-back 仍看到旧值时：

- 不立即把原 Operation 判定为写失败；
- 不立即重复提交同一个写操作；
- 进入 Verification Pending / Not Yet Confirmed；
- 按 Team / Platform Verification Policy 再次查询；
- 超过确认窗口仍不一致，再形成 Mismatch / Needs Attention。

这样避免：

~~~text
平台其实已经接受写入
↓
读取暂时还是旧值
↓
ERP 误判失败
↓
再次写入
~~~

造成重复或乱序操作。

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

## 19. 下一步

下一轮建议继续 Walmart Capability：

> **Orders / Shipment / Cancellation / Returns / Refund**

重点确认：

- Order Sync 与单单详情；
- Platform Order Fact；
- Order Line；
- Confirm Shipment；
- 一个 Order Line 多 Shipment；
- Cancellation；
- Return；
- Refund；
- ERP 主动 Refund 与平台同步 Refund 的区别；
- 哪些操作需要 Effect Verification。
