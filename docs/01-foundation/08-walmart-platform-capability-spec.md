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

## 9. Store Maximum Inventory Policy

旧脚本中“库存特殊设置”的正式业务含义调整为：

> **Store Maximum Inventory**

它属于 Store Policy，不是 Product Source Inventory。

必须区分：

~~~text
Source Inventory
= 来源端真实可售库存

Store Maximum Inventory
= 当前 Store 对在线 Listing 允许写给 Walmart 的最大库存
~~~

例如：

~~~text
Amazon Source Inventory = 37
Store H006 Maximum Inventory = 10
~~~

最终写给 Walmart 的库存由库存业务规则根据来源事实和 Store Policy 决定。

不同 Store 可以配置不同 Maximum Inventory。

该配置修改后需要进入 Store Configuration History，以支持历史经营分析。

---

## 10. 单个与批量 Business Operation

EasyQuickAIERP V1 同时支持 Single Listing Operation 和 Batch Listing Operation。

业务语义保持一致，例如：

- Submit Listing(s)；
- Retire Listing(s)；
- Delete Listing(s)；
- Update Price；
- Update Inventory。

Walmart Adapter 根据平台能力选择 Single-item API、Bulk Feed 或自动切片成多个 Feed。

因此：

> Business Operation 不与 Walmart Endpoint 一一绑定。

---

## 11. Risk / Permission

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

---

## 12. Data Retention

Walmart Listing / Feed 历史遵守：

> [Data Retention Policy](./07-data-retention-policy.md)

当前默认：

- Listing 完整 Content Version：24 个月；
- Listing Submission / Feed Item 完整明细：24 个月；
- Walmart API Raw Payload / Response：12 个月；
- Price / Inventory Operation History：24 个月；
- Job 技术运行日志：6 个月。

这些为默认初始值，可由 Platform Admin 后续调整。

---

## 13. 当前已确认原则

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
20. Store Maximum Inventory 是 Store Policy，与 Product Source Inventory 分开。
21. V1 同时支持单个和批量 Business Operation。
22. 历史明细按照 Data Retention Policy 管理，不要求主库无限期保存。

---

## 14. 下一步

下一轮继续 Walmart Capability：

> **Price / Inventory / Multi-node Inventory / Shipping Template Mapping / Shipping Overrides**

重点确认：

- 当前价格与价格操作历史；
- 单个 / 批量价格更新；
- Source Inventory、Store Maximum Inventory、Walmart Node Inventory 三层关系；
- 多节点库存展示与更新；
- 默认 Ship Node 与指定 Ship Node；
- Shipping Template 与 Listing / Fulfillment Center 的关系；
- Shipping Overrides；
- 哪些属于 Platform Fact，哪些属于 Team Policy。
