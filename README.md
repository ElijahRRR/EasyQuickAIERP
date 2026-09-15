# EasyQuickAIERP

EasyQuickAIERP 是一个面向多团队、多销售平台、多商品来源的可扩展 ERP 新项目。

当前阶段：**阶段 0 — 业务建模**。

项目当前不以迁移旧系统为主线。旧系统只作为业务经验、已验证能力和实现参考；新 ERP 先建立正确的业务模型与基础能力，再在后续阶段通过 Workflow 组合自动化流程。

## 建设阶段

```text
阶段 0  业务建模
   ↓
阶段 1  ERP 地基
   ↓
阶段 2  核心数据与基础运营
   ↓
阶段 3  完整业务流程
   ↓
阶段 4  Workflow / 自动化
   ↓
阶段 5  Open API + 外部 AI
   ↓
阶段 6  内置 AI
```

## 当前权威业务文档

阶段 0 文档统一保存在：

[`docs/00-business-modeling/`](./docs/00-business-modeling/)

当前已经确认：

- [阶段 0 索引与建模原则](./docs/00-business-modeling/README.md)
- [商品生命周期业务模型](./docs/00-business-modeling/01-product-lifecycle.md)
- [Listing 生命周期业务模型](./docs/00-business-modeling/02-listing-lifecycle.md)
- [Platform Error Management 业务模型](./docs/00-business-modeling/03-platform-error-management.md)

## 文档原则

1. 当前团队的流程不等于所有团队的强制流程；
2. ERP 优先提供独立、可组合的基础业务能力；
3. Workflow 决定各团队如何组合这些能力；
4. 当前团队的流程可以成为 Workflow Template；
5. AI、Workflow 和人工操作最终应复用相同业务能力；
6. 已确认的业务事实进入仓库；未确认内容继续讨论，不提前固化；
7. 业务规则、平台事实和技术实现保持分层。

## 下一步

继续阶段 0，下一业务域：

> **Order Lifecycle — 订单生命周期**
