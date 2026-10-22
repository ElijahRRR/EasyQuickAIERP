# EasyQuickAIERP

EasyQuickAIERP 是一个面向多团队、多销售平台、多商品来源的可扩展 ERP 新项目。

当前阶段：**阶段 1 — ERP 地基**。

开发团队继续 PostgreSQL schema；产品侧并行推进 **V1 范围与分期规划**，不重复设计数据库表。

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
- [业务对象关系与开发交接约束图 v0.2](./docs/01-foundation/06-business-object-relations-handoff-constraints.md)
- [Data Retention Policy v0.1](./docs/01-foundation/07-data-retention-policy.md)
- [Walmart Platform Capability Specification v0.2](./docs/01-foundation/08-walmart-platform-capability-spec.md)
- [正式业务动作清单 Baseline v1.0](./docs/01-foundation/09-business-operation-catalog.md)

产品交付规划（新增提案，待确认分期，不替代上述基线）：

- [V1 功能范围与优先级 v0.1](./docs/product-planning/01-v1-scope-and-priorities.md)
- [V1 交付与验收门槛 v0.1](./docs/product-planning/02-v1-delivery-and-acceptance.md)
- [产品规划索引](./docs/product-planning/README.md)

## 文档原则

1. 当前团队的流程不等于所有团队的强制流程；
2. ERP 优先提供独立、可组合的基础业务能力；
3. Workflow 决定各团队如何组合这些能力；
4. 当前团队的流程可以成为 Workflow Template；
5. AI、Workflow 和人工操作最终应复用相同业务能力；
6. 已确认的业务事实进入仓库；未确认内容继续讨论，不提前固化；
7. 业务规则、平台事实和技术实现保持分层。

## 下一步

继续阶段 1，并行推进两条工作：

1. **开发侧**：完成第一版 PostgreSQL Logical / Physical Schema，形成可供业务一致性审查的交接成果。
2. **产品侧**：确认 V1 必做／后补／未来范围和 M0–M3 交付门槛，再细化页面／菜单、页面动作、权限矩阵、完整验收用例及当前 Team Workflow Template。

Business Operation Catalog 已收口。产品规划不等待 schema 最终定稿，也不重复设计数据库表。后续结构审查重点仍是对象关系、历史保留、平台事实与 ERP 决策边界。

