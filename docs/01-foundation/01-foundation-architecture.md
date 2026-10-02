# Foundation Architecture v0.1

> 状态：已确认  
> 所属阶段：阶段 1 — ERP 地基  
> 目标：定义数据库、API、Workflow、AI、平台接入都必须遵守的系统级架构边界。

## 1. Team 是 Tenant Boundary

绝大多数业务数据必须属于明确的 Team，例如：

```text
Team
├ Store
├ Product
├ Source
├ Listing
├ Order
├ Purchase
├ Return / Refund
├ Settlement
├ Workflow
└ Digital Employee
```

正常 Team User 只能访问所属 Team 的数据。

系统级数据可以例外，例如：

- System Public Blacklist；
- 系统平台配置；
- 套餐 / Entitlement；
- ERP 公共能力配置。

---

## 2. Platform Super Admin 可以跨 Team Support

ERP 平台超级管理员可以进入指定 Team 查看并协助处理问题。

但这种访问必须与普通 Team User 区分。

所有跨 Team 的高价值操作必须 Audit，至少记录：

- Platform Admin；
- Target Team；
- Operation；
- Target；
- Time；
- Result。

原则：

> 可以跨 Tenant Support，但不能无痕跨 Tenant。

具体是否存在只读 Support Mode / 可操作 Support Mode，后续 Permission Design 再定义。

---

## 3. 核心对象使用稳定 ERP Internal ID

核心业务对象拥有创建后不随外部字段变化而改变的 ERP Internal ID，例如：

- Store ID；
- Product ID；
- Source ID；
- Listing ID；
- Order ID；
- Order Line ID；
- Procurement Task ID；
- Purchase ID；
- Return ID；
- Settlement Entry ID。

Internal ID 主要服务系统内部关系，不要求普通用户日常查看。

---

## 4. Internal ID 与 External Identifier 分开

例如 Listing：

```text
ERP Listing ID = 稳定内部身份

External Identifiers:
- SKU
- UPC
- GTIN
- Walmart Item ID
- WPID
```

SKU / UPC / GTIN 等外部标识变化，不改变 ERP Listing Identity。

Store Name、Credential、Platform Identifier 等同样不能替代 ERP 自身稳定身份。

---

## 5. Domain Data Ownership

阶段 1 坚持：

> **谁拥有数据，谁负责修改。**

初步边界：

### Product Domain

拥有 Product、Product Source、Product Data。

### Listing Domain

拥有 Listing、Listing Field、Listing Submission、Listing State。

### Order Domain

拥有 Sales Order、Order Line、Order Audit。

### Procurement Domain

拥有 Procurement Task、Purchase。

### Logistics Domain

拥有 Platform Shipment、Tracking Observation、Logistics Exception。

### After-sales Domain

拥有 Return、Refund Fact。

### Finance Domain

拥有 Settlement、Reconciled Amount、Profit Projection / Reconciliation。

---

## 6. 跨 Domain 不直接修改对方数据

其他 Domain 可以引用事实，但不能绕过 Data Owner 直接修改其内部数据。

例如 After-sales 获得 Refund Fact 后：

```text
After-sales
↓
Refund Fact
↓
Finance Operation
↓
Recalculate Estimated Profit
```

而不是直接修改 Finance 内部数据。

原则：

> 跨 Domain 通过明确 Business Operation 协作。

---

## 7. Platform Adapter 是 Marketplace 翻译层

所有外部 Marketplace API 原则上通过 Platform Adapter 访问：

```text
ERP Business Domain
        ↓
Platform Adapter
        ↓
Walmart / Amazon / eBay / ...
```

业务模块不应散落直接调用具体平台 API。

---

## 8. Platform Adapter 的职责

主要隔离：

1. Authentication / Connection；
2. Request / Response Translation；
3. Platform State Translation；
4. Platform Capability。

例如 ERP 只表达：

```text
Update Inventory
Listing = X
Quantity = 5
```

Walmart Adapter 负责处理 Walmart 所需的 SKU、Warehouse、请求结构和 API。

---

## 9. 通用业务代码不感知平台实现细节

上层业务只使用统一 Business Capability。

平台差异由 Adapter / Capability Layer 吸收，避免整个代码库大量出现：

```python
if platform == "walmart":
    ...
elif platform == "amazon":
    ...
```

---

## 10. Business Operation 是正式业务动作的统一入口

系统中的高价值业务动作应只有一条正式执行路径，例如：

- RefundOrder；
- UpdateListingPrice；
- UpdateInventory；
- SubmitListing；
- CancelOrder；
- CreatePurchase；
- RetireListing。

这些属于 Business Operations。

---

## 11. 所有触发方式复用同一个 Operation

未来：

```text
Human UI
Workflow
AI Agent
Open API
Scheduler
        ↓
Business Operation
```

不能为不同入口重复实现同一套业务逻辑。

---

## 12. Business Operation 统一处理业务规则

典型 Operation 负责：

1. Actor Context；
2. Authorization；
3. Business Validation；
4. Idempotency；
5. Domain Logic；
6. Platform Adapter（如需要）；
7. Persist Result；
8. Audit；
9. 必要的后续业务触发。

---

## 13. Authentication、Authorization、Business Validation 分开

### Authentication

回答：

> 你是谁？

### Authorization

回答：

> 你是否有权对这个对象执行这个动作？

### Business Validation

回答：

> 即使有权限，这件事当前业务上是否允许执行？

三者不能混用。

---

## 14. HTTP / Open API 必须 Authentication

任何需要身份的 API 都必须先认证调用者。

不能因为请求来自 ERP 前端就默认可信。

Open API 同样需要 API Token、OAuth 或后续定义的认证方式。

---

## 15. Authorization 在后端强制执行

前端隐藏按钮、菜单属于 UX，不属于最终安全边界。

典型流程：

```text
HTTP Request
↓
Authentication
↓
Actor Context
↓
Business Operation
↓
Authorization
↓
Business Validation
↓
Execute
```

---

## 16. Authorization 使用阶段 0 权限模型

权限维度：

```text
Actor
↓
Platform
↓
Store Scope
↓
Function
↓
Action
```

例如：

```text
Actor = 张三
Platform = Walmart
Store = H006
Function = After-sales
Action = Refund
```

只有匹配 Permission Grant 才允许继续。

---

## 17. Digital Employee 使用相同授权边界

Workflow / AI / Scheduler 等不是系统超级权限。

Digital Employee 必须像 Human Actor 一样经过 Authorization。

自动化不得绕过权限系统。

---

## 18. Audit Log 重点记录 Mutation / 高价值操作

当前不要求所有 Read 都进入 Audit。

重点记录：

- Create；
- Update；
- Delete；
- Refund；
- Cancel；
- Purchase；
- Permission Change；
- Credential Change；
- Workflow Change；
- Blacklist / Whitelist Change；
- 高风险 Platform Operation。

---

## 19. Audit 至少回答

```text
谁
在什么时候
通过什么入口
对什么对象
执行了什么操作
结果是什么
```

概念上包含：

- Team；
- Actor Type；
- Actor ID；
- Operation；
- Target；
- Time；
- Source；
- Result；
- 关键 Change / Before-After。

Human 与 Digital Actor 使用统一 Audit Model。

---

## 20. Async Job 是 ERP 地基能力

系统从阶段 1 就提供统一异步任务模型。

核心业务中大量任务天然异步，例如：

- Order Sync；
- Return Sync；
- Settlement Sync；
- Bulk Listing；
- Feed Submit / Poll；
- Product Audit；
- Risk Scan；
- Scraping；
- Batch Price Update；
- Batch Inventory Update。

---

## 21. 各 Domain 不重复实现任务框架

不允许 Listing、Order、Scraper、Finance 各自发明不同 Task System。

统一使用 Job / Task Foundation。

---

## 22. Async Job 基础生命周期

概念上至少支持：

```text
Queued
↓
Running
↓
Succeeded
```

以及：

- Failed；
- Retry；
- Cancelled；
- Partial Success；
- Progress；
- Error；
- Retry Attempt。

具体技术状态后续设计。

---

## 23. Job 必须带业务上下文

Job 至少应能够关联：

- Team；
- Actor；
- Job Type；
- Business Target；
- Trigger Source；
- Progress；
- Result；
- Error；
- Created / Started / Finished。

---

## 24. Async Infrastructure 与业务逻辑分开

阶段 1 先确认统一 Async Job 能力。

当前不锁定：

- Celery；
- Redis；
- RabbitMQ；
- Kafka；
- PostgreSQL Queue；
- 其他技术。

---

## 25. Foundation 请求执行主链

同步动作：

```text
Client
(UI / Workflow / AI / Open API)
        ↓
Authentication
        ↓
Actor Context
        ↓
Business Operation
        ↓
Authorization
        ↓
Business Validation
        ↓
Domain Logic
        ↓
Platform Adapter（如需要）
        ↓
External Platform
        ↓
Persist Business Result
        ↓
Audit
```

异步动作：

```text
Business Operation
↓
Create Async Job
↓
Worker Executes
↓
Domain / Platform Adapter
↓
Update Job Result
↓
Persist Business Result
↓
Audit
```

---

## 26. 当前已确认原则

1. Team 是强 Tenant 数据隔离边界。
2. Platform Super Admin 可以进入指定 Team Support。
3. Platform Super Admin 的跨 Team 高价值操作必须 Audit。
4. 核心业务对象使用稳定 ERP Internal ID。
5. External Identifier 不作为唯一永久业务身份。
6. Internal ID 主要供系统内部使用。
7. Domain 遵守“Data Owner 负责修改”。
8. 跨 Domain 不直接修改对方拥有的数据。
9. Marketplace API 原则上全部通过 Platform Adapter。
10. Platform Adapter 隔离认证、API、状态、参数与 Capability 差异。
11. 正式业务动作通过 Business Operation 执行。
12. UI / Workflow / AI / Open API / Scheduler 复用同一 Business Operation。
13. Authentication、Authorization、Business Validation 是三个不同概念。
14. HTTP / Open API 必须 Authentication。
15. Authorization 在后端强制执行。
16. 前端 Permission Control 只属于 UX。
17. Authorization 使用 Actor + Platform + Store + Function + Action。
18. Digital Employee 也必须经过相同 Authorization。
19. Audit 重点记录 Mutation / 高价值业务动作。
20. Human 与 Digital Actor 使用统一 Audit Model。
21. Async Job 从阶段 1 就作为系统 Foundation。
22. 各 Domain 不自行重复实现后台任务框架。
23. 阶段 1 只确认架构边界，不提前锁死具体技术组件。

---

## 27. 当前暂不决定

Foundation Architecture v0.1 暂时不决定：

- PostgreSQL 具体表结构；
- UUID / ULID / Snowflake 具体 ID 算法；
- 后端语言；
- Web Framework；
- 单体 / 微服务最终部署形态；
- Redis / RabbitMQ / Kafka；
- ORM；
- 前端技术栈；
- API URL 规范。

这些在后续地基设计中逐层确认。

---

## 28. 下一步

下一步：

> **Domain & Data Ownership Map**

需要明确：

- 哪个 Domain 拥有哪些核心对象；
- 谁可以引用谁；
- 哪些关系只保存 ID Reference；
- 哪些变化必须通过 Business Operation；
- 哪些 Platform Fact 属于哪个 Domain；
- 哪些跨 Domain 依赖需要事件 / 调用协作。

完成 Domain Ownership 后，再进入数据库 Schema 与模块目录设计。
