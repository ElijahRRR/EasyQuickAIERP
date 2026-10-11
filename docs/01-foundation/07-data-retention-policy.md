# Data Retention Policy v0.1

> 状态：已确认默认策略；具体归档技术方案待开发设计  
> 所属阶段：阶段 1 — ERP 地基  
> 目标：明确不同类型业务历史的默认保留期限、归档原则，以及“历史不可被普通业务操作删除”与“物理存储不能无限增长”之间的边界。

## 1. 总原则

EasyQuickAIERP 必须同时满足两个目标：

1. 业务历史不能因为当前状态变化、解绑、停用、删除等普通业务操作而被无痕覆盖或级联删除；
2. PostgreSQL 主库不能因为所有原始 Payload、版本和技术日志无限期保留而持续无上限增长。

因此：

> **业务历史不可被普通业务操作删除，不等于所有明细都必须永久保存在 PostgreSQL 热库。**

---

## 2. “永久保留”的统一解释

此前业务文档中出现的“永久保留”，统一解释为：

> 该历史事实不能因为后续业务对象变化而被覆盖、重写或级联删除；实际物理保存期限、归档层级和最终清理时间由 Data Retention Policy 决定。

例如：

~~~text
Listing Delete
≠ 立即删除 Listing History

Store Unbind
≠ 立即删除 Order History

Product Source Unbind
≠ 立即删除 Purchase History
~~~

达到 Retention Policy 规定期限后，可以由专门的系统归档 / 清理流程处理。

普通业务 Delete / Unbind / Retire 永远不能替代 Retention Cleanup。

---

## 3. 数据生命周期

推荐采用：

~~~text
Hot Data
↓
Archive
↓
Final Purge
~~~

### Hot Data

保存在 PostgreSQL 主业务库中，可直接用于 ERP 页面、日常运营、Audit、问题排查、报表、Workflow / AI。

### Archive

超过热数据期限以后，可以迁移到低成本归档存储。

归档以后不要求所有页面实时直接查询，但应根据需要支持恢复 / 调阅。

### Final Purge

达到最终保留期限以后，可以按系统策略执行物理删除、脱敏或其他合规清理。

具体 Archive Storage、压缩格式、对象存储、分区策略等由开发人员设计。

---

## 4. 默认初始保留期限

以下为当前确认的系统默认值。

这些不是硬编码业务常量，未来 Platform Admin 可以配置。

| 数据类别 | 默认初始保留期限 |
|---|---:|
| Listing 完整 Content Version | 24 个月 |
| Listing Submission / Feed Item 完整明细 | 24 个月 |
| Walmart / Platform API 原始 Payload / Response | 12 个月 |
| Price / Inventory Operation History | 24 个月 |
| Store Configuration / Assignment History | 5 年 |
| Product ↔ Source Relation History | 5 年 |
| Order / Purchase / Refund / Settlement 等交易与财务记录 | 7 年 |
| Audit / Permission / 高风险操作记录 | 7 年 |
| Job / Worker 技术运行日志 | 6 个月 |

这些期限后续可以根据法律 / 税务要求、Team / Plan、存储成本、实际排障价值和合规要求进行调整。

---

## 5. 当前值与历史明细分开

Retention Policy 主要作用于历史明细，不应误删业务对象当前必要状态。

例如 Listing：

~~~text
Current Listing
→ 只要业务对象仍存在就继续保留

Historical Content Version
→ 按 Retention Policy 管理

Feed Raw Payload
→ 按较短 Retention Policy 管理
~~~

同理：

- Store 当前配置不因历史配置到期而消失；
- Product 当前 Source Relation 不因旧 Relation History 到期而消失；
- Current Settlement / Profit 不能因为旧 Job Log 到期而消失。

---

## 6. 原始 Payload 与业务摘要分开

高占用数据可以采用更短保留期，例如：

- Walmart API Raw JSON；
- Feed 原始 Payload；
- Feed 原始 Error Body；
- 大型 Listing Content Snapshot；
- 二进制 Report；
- 技术 Debug Log。

即使原始 Payload 最终到期清理，系统仍应尽可能保留必要的业务摘要，例如：

- Object ID；
- Operation；
- Actor；
- Time；
- Result；
- Error Code / Error Category；
- Spec Version；
- Content Version；
- Feed ID；
- 必要 Platform Identifier。

因此：

> 原始技术证据可以比核心业务摘要更早归档 / 清理。

---

## 7. Retention Cleanup 是独立系统能力

Retention Cleanup 不属于普通业务 Delete。

它必须：

- 按明确 Retention Policy 执行；
- 由受限 System Service / Platform Admin 管理；
- 自身留下 Audit；
- 不破坏仍被核心业务引用的数据；
- 不因为删除 Job / Raw Payload 而删除 Business Record；
- 支持 Dry-run / Preview 或等效安全机制。

---

## 8. Schema 设计要求

开发设计 PostgreSQL Schema 时，需要考虑：

- 历史大表的时间范围；
- Partition / Archive 的可行性；
- Raw Payload 与核心业务字段是否需要分离；
- 大型 JSON / Report 是否适合长期放 PostgreSQL；
- Retention Cleanup 如何避免破坏外键；
- Archive 后如何保留关键引用和摘要；
- Platform Admin 如何调整默认期限。

具体物理实现由开发决定。

---

## 9. 当前未锁定的技术方案

当前不决定：

- PostgreSQL Partition 具体方式；
- 对象存储技术；
- 压缩格式；
- Archive 查询方式；
- Cold Storage 恢复 SLA；
- 每个 Team 是否允许自定义期限；
- 合规删除 API 的最终形式。

这些由后续开发设计与技术评审确定。

---

## 10. 当前核心原则

~~~text
普通业务删除
≠ Retention Cleanup

历史不可无痕覆盖
≠ PostgreSQL 永久无限保存

Business Record
≠ Raw Payload
≠ Job Log

Hot Data
→ Archive
→ Final Purge
~~~
