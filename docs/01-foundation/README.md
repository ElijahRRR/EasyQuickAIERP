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
| [Foundation Architecture](./01-foundation-architecture.md) | v0.1 | Tenant、Internal ID、Domain Ownership、Platform Adapter、Business Operation、Authorization、Audit、Async Job |
| [业务模块与数据归属图](./02-domain-data-ownership.md) | v0.1 | Store、商品、Listing、订单、采购、物流、售后、财务的数据 Owner 与跨模块协作边界 |
| [核心对象业务信息需求（第一批）](./03-core-object-business-information-1.md) | v0.1 | Store、Product、Product Source、Listing 的业务信息、历史保留与数据设计分工 |

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

## 4. 下一步

> **核心对象业务信息需求（第二批）**

下一批确认：

- Sales Order；
- Order Line；
- Procurement Task；
- Purchase。

仍然只确认业务上必须保存什么、什么需要保留历史、哪些关系可以为空。

数据库物理实现继续由开发侧负责。
