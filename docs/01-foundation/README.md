# EasyQuickAIERP — 阶段 1：ERP 地基

> 状态：进行中  
> 前置基线：[阶段 0 — 业务建模 Baseline v1](../00-business-modeling/README.md)

## 1. 阶段目标

阶段 1 不再讨论“业务是什么”，而是定义：

> **这些业务事实如何被系统稳定承载，并为后续人工操作、Workflow、Open API、AI 共用同一套基础能力。**

阶段 1 仍然遵守：

- 不直接照搬旧系统表结构；
- 不把当前 Team Workflow 硬编码成唯一流程；
- 不为了“大而全”提前设计没有真实业务需求的模块；
- 技术设计必须服从阶段 0 已确认业务边界。

## 2. 当前已确认文档

| 文档 | 版本 | 内容 |
|---|---:|---|
| [Foundation Architecture](./01-foundation-architecture.md) | v0.2 | Tenant、Internal ID、Domain Ownership、Platform Adapter、Business Operation、Authorization、Audit、Async Job |
| [业务模块与数据归属图](./02-domain-data-ownership.md) | v0.1 | Store、商品、Listing、订单、采购、物流、售后、财务的数据 Owner 与跨模块协作边界 |
| [核心对象业务信息需求（第一批）](./03-core-object-business-information-1.md) | v0.3 | Store、Product、Product Source、Listing 的业务信息、历史保留与数据设计分工 |
| [核心对象业务信息需求（第二批）](./04-core-object-business-information-2.md) | v0.3 | Sales Order、Order Line、Order Audit、Procurement Task、Purchase 的业务信息与历史快照 |
| [核心对象业务信息需求（第三批）](./05-core-object-business-information-3.md) | v0.1 | Platform Shipment、Tracking、Return、Refund、Settlement Entry、Settlement Period / Payout 的业务信息与历史保留 |
| [业务对象关系与开发交接约束图](./06-business-object-relations-handoff-constraints.md) | v0.1 | 核心对象基数、可空关系、历史保留、禁止级联删除、Job/权限、Platform Fact 与 ERP Decision 开发约束 |

## 3. Foundation Architecture 核心原则

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
Persist + Audit
```

异步任务通过统一 Job Foundation 执行。

其中必须保持：

```text
Job ≠ Business Record
Queued Permission ≠ Execution Permission
System Sync ≠ User Delegation
```

用户委托 / Digital Employee 的后台任务在真正产生业务副作用时重新校验当前权限；系统同步身份只在受限范围内维护 Platform Fact。

## 4. 下一步

> **开发侧 PostgreSQL Schema 设计与技术评审**

阶段 1 当前已经完成核心业务对象的上层关系与开发交接约束。

下一步由开发人员基于本目录和阶段 0 Baseline 输出第一版 PostgreSQL Logical / Physical Schema。

业务侧不逐字段决定数据库实现；后续评审重点检查：

- Schema 是否违反已确认对象关系；
- 是否错误设置必填 / 唯一关系；
- 是否存在危险 Cascade Delete；
- 是否丢失历史快照 / Rule Version / Evidence；
- 是否把 Current Value 与 Historical Fact 混为一体；
- 是否把 Platform Fact 与 ERP Decision 混为一个状态；
- 是否满足 Tenant、Authorization、Audit、Job 等 Foundation 边界。
