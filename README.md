# EasyQuickAIERP

EasyQuickAIERP 是一个面向多团队、多销售平台、多商品来源的可扩展 ERP 新项目。

当前阶段：**阶段 1 — ERP 地基**。

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

## 当前权威文档

阶段 0 业务基线：

[`docs/00-business-modeling/`](./docs/00-business-modeling/)

阶段 1 ERP 地基：

[`docs/01-foundation/`](./docs/01-foundation/)

当前关键入口：

- [阶段 0 — 业务建模 Baseline v1](./docs/00-business-modeling/README.md)
- [阶段 0 收口审查](./docs/00-business-modeling/12-stage-0-closure-review.md)
- [阶段 1 — ERP 地基索引](./docs/01-foundation/README.md)
- [Foundation Architecture v0.2](./docs/01-foundation/01-foundation-architecture.md)
- [业务对象关系与开发交接约束图 v0.1](./docs/01-foundation/06-business-object-relations-handoff-constraints.md)

## 文档原则

1. 当前团队的流程不等于所有团队的强制流程；
2. ERP 优先提供独立、可组合的基础业务能力；
3. Workflow 决定各团队如何组合这些能力；
4. 当前团队的流程可以成为 Workflow Template；
5. AI、Workflow 和人工操作最终应复用相同业务能力；
6. 已确认的业务事实进入仓库；未确认内容继续讨论，不提前固化；
7. 业务规则、平台事实和技术实现保持分层。

## 下一步

继续阶段 1：

> **开发侧 PostgreSQL Schema 设计与技术评审**

开发人员基于阶段 0 Business Baseline 与阶段 1 Foundation / Data Ownership / Core Object / Handoff Constraints 输出第一版 Logical / Physical Schema；业务侧重点评审是否违反已确认业务关系和历史约束。
