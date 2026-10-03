# 核心对象业务信息需求（第一批）v0.1

> 状态：已确认  
> 所属阶段：阶段 1 — ERP 地基  
> 说明：本文只确认“业务上必须保存什么信息、这些信息的业务含义和历史规则”。数据库表名、字段名、数据类型、索引、外键等物理设计由开发人员负责。

## 1. Store（店铺）

Store 业务上需要长期保存：

- 所属 Team；
- Marketplace；
- 平台稳定身份（如 Walmart Partner ID / Seller ID）；
- ERP 内 Store Name；
- 当前使用的 API Credential Reference；
- Platform State；
- ERP Enable State；
- Connection Health；
- 当前负责人；
- 当前协作人员；
- 当前运营组；
- 负责人 / 运营组历史；
- Store-level 经营配置；
- Platform Resource Reference。

### 1.1 Store 经营配置需要历史

Store 配置修改后不能只保留当前值。

需要能够回看某个时间段内的经营条件，例如：

- 价格范围；
- Procurement Cost Coefficient；
- Operating Category；
- Workflow；
- Risk Policy；
- 其他 Store Policy。

目的：

> 支持分析某个时间段在某一组经营条件下的 Store 运营结果。

因此业务上需要：

```text
Current Store Configuration
+
Store Configuration History / Store Events
```

Audit Log 关注“谁改了什么”，Store Event / Configuration History 更关注“业务配置何时从什么变成什么”。

### 1.2 Walmart Platform Resource

ERP 需要保存 Walmart 当前 Platform Warehouse / Shipping Template 等资源列表。

同时，对重要 Listing / Inventory Operation，需要知道：

> 当时实际使用了哪个 Platform Warehouse / Shipping Template。

用于以后排查：

- 库存是否更新到正确 Warehouse；
- Listing 是否使用正确 Shipping Template；
- 某次平台操作为什么出现异常。

这些仍属于 Platform Resource，不是 ERP 自有实体 Warehouse。

---

## 2. Product（商品主档）

Product 表示：

> Team 认为“这是同一个商品”的内部商品身份。

Product 不需要强行维护第三套完整 Listing 文案。

Product 主要保存和承载：

- 商品内部身份；
- 核心商品属性；
- Brand；
- Category / Product Type；
- 规格、尺寸、材质、变体等用于说明“这是什么商品”的信息；
- 与 Product Source 的关系；
- Product Audit；
- Risk Result；
- 目标销售平台关系；
- 其他能够稳定描述商品身份的业务信息。

来源原始 Title / Description 等主要属于 Product Source。

最终销售 Title / Description 等属于 Listing Final Data。

因此：

```text
Source Data
→ 表示来源平台当前提供什么

Product
→ 表示 Team 认为这是什么商品

Listing Final Data
→ 表示实际在目标 Store / Platform 怎么卖
```

---

## 3. Product Source（商品来源）

Product Source 当前按：

> **Amazon ASIN 级**

进行识别。

Seller 变化不会创建新的 Product Source。

例如：

```text
Product Source = ASIN B0ABC123

Current Seller:
Seller A
↓
后来
Seller B

Product Source Identity 不变
```

Product Source 业务上至少需要知道：

- 所属 Product；
- Source Platform；
- Source Product Identifier（例如 ASIN）；
- Source URL；
- 当前 Seller / Seller ID（如果可获得）；
- 当前 Price；
- Shipping Cost；
- Stock / Availability；
- Fulfillment / Delivery Information；
- Source Product Raw / Parsed Data；
- Source 是否有效；
- Primary / Backup Relation；
- Last Sync Time。

### 3.1 Seller 不是 Source Identity

Seller 是当前来源状态和采购判断条件的一部分。

例如同一个 ASIN 当前可能由不同 Seller 提供。

Seller Blacklist 的作用是：

> 阻止从某个 Seller 实际采购，而不是自动让整个 ASIN Source 失效。

因此采购时需要检查：

```text
ASIN
↓
Current / Selected Seller
↓
Seller Risk Check
↓
Purchase Decision
```

### 3.2 一个 Product 可以关联多个 Amazon ASIN

例如：

```text
Product X
├ ASIN A
├ ASIN B
└ ASIN C
```

表示这些来源被 Team 认为可以履约同一个 Product。

但：

> 新增这种“同商品来源关系”原则上由人工确认。

系统 / AI 可以：

- 推荐可能相同的 ASIN；
- 提示差异；
- 生成合并建议。

系统 / AI 默认不能：

> 自动把两个 ASIN 合并到同一个 Product。

原因是错误合并会直接影响：

- Listing；
- Order Audit；
- Procurement；
- Customer Fulfillment。

### 3.3 Product Source Relation 可以解绑

如果人工后来发现某个 ASIN 不是同一商品，可以从当前 Product 解绑。

但：

> 解绑当前 Product Source Relation 不得删除或改写已经发生的历史 Purchase。

当前关系与历史交易必须分开。

---

## 4. Listing（销售实例）

Listing 表示：

> Product 在某个 Platform / Store 中的销售实例。

Listing 永久知道：

- 来源 Product；
- Platform；
- Store；
- ERP Listing Identity。

同时保存自己独立的 Final Listing Data。

业务上至少需要保存：

- Current SKU；
- Historical SKU；
- UPC / GTIN 及其历史；
- Platform Item ID / WPID 等外部标识；
- Final Title；
- Final Brand；
- Final Description；
- Final Images；
- Final Attributes；
- Price；
- Inventory；
- ERP Business State；
- Submission State；
- Platform State；
- Created / Submitted / Published Time；
- Platform Resource References；
- 当前 Walmart Platform Warehouse / Shipping Template 相关操作结果（按平台能力）。

Product Source 后续变化：

> 不自动修改已发布 Listing。

需要修改 Listing 时，必须通过 Listing 的正式业务操作。

---

## 5. Listing 价格 / 库存历史

Listing 主对象需要有当前：

- Price；
- Inventory。

同时重要变化需要通过：

> Product Event / Operation History

永久可追踪。

例如：

```text
2026-10-01 10:00
Store A / Listing X
Price: 39.99 → 42.99
Actor: 张三
```

或：

```text
Inventory: 5 → 0
Reason: Source OOS
Actor: Workflow A
```

这些历史用于：

- 排查问题；
- 分析价格阶段表现；
- 分析库存归零原因；
- 追踪 Workflow / User 操作；
- 还原 Listing 某个时间点的经营条件。

UI 可以继续称为“产品事件”。

系统内部事件应准确关联：

- Product；
- Listing；
- Store；
- Actor；
- Operation Type；
- Before；
- After；
- Reason；
- Time。

不要求在 Listing 主对象上保存一条无限增长的库存流水，但重要 Mutation 必须通过 Audit / Operation History 可追踪。

---

## 6. 第一批对象的关键边界

### Product ≠ Product Source

Product 表示“这是什么商品”。

Product Source 表示“从哪里获得 / 采购这个商品”。

### Product Source ≠ Purchase

Product Source 保存当前来源状态。

Purchase 保存当时真实交易事实。

### Product ≠ Listing

Product 是内部商品主档。

Listing 是具体 Store / Platform 的销售实例。

### Listing Final Data ≠ Source Current Data

来源变化不会自动覆盖已发布 Listing。

### Current Relation ≠ Historical Transaction

Product Source Relation 可以解绑，但历史 Purchase 永久保留。

### Current Listing Value ≠ Operation History

Listing 主对象保存当前状态；重要历史变化由 Operation History / Product Event 保留。

---

## 7. 数据库设计分工

Business Owner 负责确认：

- 业务对象是什么；
- 必须保存哪些业务信息；
- 哪些信息可以变化；
- 哪些历史必须永久保留；
- 哪些关系可以为空；
- 什么情况下仍然是同一个对象；
- 数据来源谁最权威。

开发人员负责：

- 表结构；
- 字段名；
- 数据类型；
- 索引；
- 外键；
- 唯一约束；
- JSON / 关系拆分；
- 迁移策略；
- 性能与容量设计。

业务方不需要逐字段决定物理数据库实现。

---

## 8. 下一步

下一批确认：

- Sales Order；
- Order Line；
- Procurement Task；
- Purchase。

仍然只讨论“业务上必须保存哪些信息和历史”，不直接设计数据库字段。
