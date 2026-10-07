# EasyQuickAIERP — 阶段 0：业务建模

> 状态：已收口（Baseline v1）  
> 目标：先理解业务，再设计系统；先建设基础能力，再通过 Workflow 组合业务流程。

## 1. 本目录的定位

本目录保存 EasyQuickAIERP 在正式技术设计之前已经确认的业务模型。

这些文档描述的是：

- 真实业务对象是什么；
- 一个业务对象会经历什么生命周期；
- 哪些判断属于平台事实；
- 哪些规则属于团队经营策略；
- 哪些流程是可选能力，而不是 ERP 强制流程；
- 后续 Workflow 应该组合哪些基础业务能力。

本目录**不负责**定义数据库表、API 路由、前端页面、队列实现、AI Agent 实现等技术细节。

---

## 2. 最重要的业务建模原则

### 2.1 当前描述的流程不是所有团队的强制流程

目前梳理的流程主要来源于当前实际运营方式。

这些步骤对当前团队可能是必要流程，但对其他团队未必成立。

例如：

- 商品审核可以是当前团队的必要流程，但不是所有团队的系统必经步骤；
- 店铺分配可以是多店铺团队的重要流程，但单店团队可以完全不需要；
- 订单审核可以是当前团队采购前的必要流程，其他团队也可能采用不同履约方式；
- 资料 AI 优化可以启用，也可以不启用；
- 某个团队可以要求审核后才能上架，另一个团队可以直接人工创建 Listing。

因此 ERP 不应把当前运营流程硬编码成唯一流程。

---

### 2.2 ERP 先提供独立、可组合的基础能力

例如系统应分别提供：

- 商品采集 / 导入；
- 商品审核；
- 平台选择；
- 店铺分配；
- 平台资源引用（按平台能力，例如 Walmart Platform Warehouse / Shipping Template）；
- 来源管理；
- Listing Draft；
- 商品资料校验与生成；
- Listing 提交；
- 价格更新；
- 库存更新；
- Listing 停止 / Retire / Delete；
- 平台错误识别与修复；
- 风险扫描；
- 订单同步；
- 订单审核；
- 采购与采购关联；
- 发货与物流追踪；
- 售后管理；
- 结算 / 对账；
- 利润核算。

这些首先都应该能够被人直接操作。

---

### 2.3 Workflow 负责把基础能力串成团队自己的业务流程

当前团队的业务流程未来可以成为系统内置 Workflow Template。

例如当前商品流程可能采用：

```text
采集
→ 商品池
→ 审核
→ 确定平台
→ 店铺分配
→ 资料准备
→ 上架
→ 在线维护
```

当前订单流程可以由 Team Workflow 组合为：

```text
同步订单
→ 订单审核
├─ Pending → 人工复核 / 定时重查
├─ Reject  → 运营处理 / 取消
└─ Pass    → 允许采购

并行 / 独立能力：
- Platform Shipment
- Logistics Observation
- After-sales Sync
- Settlement / Profit
```

其中 Purchase、Source Shipment、Platform Shipment 不存在 ERP 全局固定先后关系。

其他团队可以直接引用模板，也可以创建完全不同的 Workflow。

因此：

> **基础功能定义“系统能做什么”，Workflow 定义“团队希望按照什么顺序做”。**

---

### 2.4 AI 也只能使用 ERP 已经存在的业务能力

后续 AI 不应该获得任意数据库或命令行操作能力。

AI、Workflow、前端人工操作最终应复用相同的 ERP Business Operations。

---

## 3. 当前已确认的业务模型

| 文档 | 当前版本 | 内容 |
|---|---:|---|
| [商品生命周期](./01-product-lifecycle.md) | v0.6 | 商品发现、团队商品池、审核、平台、店铺分配、多来源、资料准备、在线经营与退出 |
| [Listing 生命周期](./02-listing-lifecycle.md) | v0.3 | Listing Draft、Listing 身份、提交状态、平台状态、库存归零、Retire、Delete、重新上架 |
| [Platform Error Management](./03-platform-error-management.md) | v0.2 | 首次上架失败与 Unpublished、错误分类、根因、团队处理策略、自动修复边界 |
| [订单生命周期](./04-order-lifecycle.md) | v0.3 | 订单同步、订单审核、采购、物流、Delivered、售后、对账、利润和最终闭环 |
| [采购生命周期](./05-procurement-lifecycle.md) | v0.4 | 采购任务、拆单采购、多来源、采购限价、实际成本、外部采购结算和采购/物流边界 |
| [物流生命周期](./06-logistics-lifecycle.md) | v0.2 | Source Shipment、Platform Shipment、Tracking、可观察范围、Delivered、Delay/Lost 与物流/采购解耦 |
| [售后生命周期](./07-after-sales-lifecycle.md) | v0.3 | 平台售后数据、Return/Refund、待处理售后、采购侧独立处置、估算财务影响与对账边界 |
| [对账与利润生命周期](./08-reconciliation-profit-lifecycle.md) | v0.1 | Order Line 级预计利润、实际采购成本、跨账期 Settlement、Current Reconciled Profit 与 Store Payout |
| [店铺生命周期](./09-store-lifecycle.md) | v0.3 | Store 平台身份、三类独立状态、连接关系、经营配置、解绑与历史负责人/运营组归属 |
| [Team / User / Permission / Organization](./10-team-user-permission-organization.md) | v0.3 | Team 隔离、Group、Store Assignment、Platform/Store/Function/Action 权限、Role Template 与 Digital Employee |
| [Risk Intelligence / Blacklist](./11-risk-intelligence-blacklist.md) | v0.2 | Brand / Product / Seller 风险、System Public / Team Private / Whitelist 优先级与 Workflow 边界 |
| [阶段 0 收口审查](./12-stage-0-closure-review.md) | v1.1 | 统一修订旧结论、确认阶段 0 核心不变量并正式关闭 Baseline v1 |

---

## 4. 阶段 0 的总体顺序

已确认的 ERP 建设顺序：

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

阶段 0 的目标不是提前设计一个“大而全 ERP”，而是逐个确认核心业务域的真实运行方式。

---

## 5. 文档维护规则

后续讨论采用以下规则：

1. 只有经过明确确认的业务结论才写入本目录；
2. 未确认内容标记为“待确认”，不能伪装成既定规则；
3. 当前团队策略与 ERP 全局能力必须分开描述；
4. 平台真实限制与团队经营规则必须分开描述；
5. 新结论如果推翻旧结论，应直接更新对应模型，并保留 Git 历史作为变更记录；
6. 具体 Walmart / Amazon / eBay 等 API 技术行为放入后续 Platform Capability Specification，不污染通用业务模型；
7. 阶段 0 不提前决定数据库、微服务、队列等技术方案。

---

## 6. 阶段 0 收口状态

阶段 0 已完成：

- Product；
- Listing；
- Platform Error；
- Order；
- Procurement；
- Logistics；
- After-sales；
- Reconciliation & Profit；
- Store；
- Team / User / Permission / Organization；
- Risk Intelligence / Blacklist。

统一收口结论见：

> [阶段 0 收口审查](./12-stage-0-closure-review.md)

当前正式状态：

> **阶段 0：业务建模 = Closed / Baseline v1**

Closed 不表示业务模型以后永远不能修改。新的真实业务事实仍然可以通过更新对应 Markdown 并保留 Git 历史继续演进。

---

## 7. 当前明确不建立的业务域

现阶段不因为“ERP 应该完整”而提前建立：

- Physical Warehouse / WMS；
- 完整 CRM / Message Center；
- 完整会计 / 总账系统；
- Workflow Engine 技术实现；
- AI Agent 技术实现。

Walmart Platform Warehouse / Shipping Template 当前属于：

> Platform Resource

不是 ERP 自有 Physical Warehouse Domain。

---

## 8. 下一阶段

下一步正式进入：

> **阶段 1：ERP 地基**

阶段 1 开始讨论技术结构，但必须以本目录的 Baseline v1 作为业务事实来源。

优先需要设计的地基包括：

- Team / Tenant 数据隔离；
- 稳定业务 Identity；
- Platform Adapter / Capability Boundary；
- Business Operations；
- Permission Enforcement；
- Actor / Audit Log；
- Credential / Secret Management；
- Task / Job / Async Operation；
- 模块边界与数据所有权；
- 为未来 Workflow / Open API / AI 提供统一调用基础。

阶段 1 不应直接照搬旧项目表结构，也不能把当前 Team Workflow 硬编码成系统唯一流程。
