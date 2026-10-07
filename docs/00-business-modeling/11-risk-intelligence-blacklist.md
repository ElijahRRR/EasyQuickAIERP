# Risk Intelligence / Blacklist Model v0.2

> 状态：已确认  
> 所属阶段：ERP 阶段 0 — 业务建模

## 1. 模块定位

Risk Intelligence / Blacklist 用于保存 ERP 已经明确认定需要重点规避的风险对象。

它本身不等于：

- Product Audit；
- Listing Error；
- Product；
- Listing。

而是被以下能力共同查询的一层风险资料：

```text
Risk Intelligence / Blacklist
        ↓
├── Product Audit
├── Online Product Monitoring
└── Workflow Decision
```

---

## 2. 当前主要风险对象

当前需要进入 Blacklist / Whitelist 判断的对象主要包括：

### Brand

按品牌名称或后续可稳定识别的品牌标识判断。

### Product

通常使用：

- ASIN；
- 其他稳定 Product Identifier。

### Amazon Seller

通常使用 Amazon Seller ID，例如：

`A10B1K8EC1XDY3`

因此所谓 Risk Subject 只是指：

> 这条风险记录针对 Brand、Product 还是 Seller。

不单独建立复杂的 Risk Subject 业务模块。

---

## 3. 一条风险记录表达什么

一条正式风险记录至少表达：

```text
Object Type
+
Object Identifier
+
Source / Reason
+
Scope
+
Current Effective State
```

例如：

```text
Type = Brand
Value = Brand A
Source = TRO
Scope = System Public
```

或者：

```text
Type = Amazon Seller
Value = A10B1K8EC1XDY3
Source = Team Manual
Scope = Team Private
```

阶段 0 不提前决定具体数据库字段。

---

## 4. 风险记录来源

风险信息可能来自：

- 人工发现；
- TRO 信息；
- 平台违规经验；
- Walmart 后台报错记录；
- Walmart 上架报错记录；
- 外部数据导入；
- 其他后续风险来源。

但：

> 风险信号被发现，不等于自动成为正式 Blacklist Entry。

只有经过明确规则或明确加入动作以后，才成为正式黑名单记录。

---

## 5. System Public Blacklist

ERP 可以存在：

> **System Public Blacklist**

由系统运营方维护。

系统公共黑名单可以：

- 向全部 Team 开放；
- 只向指定 Team 开放；
- 作为付费能力提供。

因此：

> Team 是否能够使用 System Public Blacklist，属于 Team 的服务能力 / Entitlement。

---

## 6. 系统公共黑名单的数据来源

System Public Blacklist 可以来自：

- 系统运营方人工维护；
- TRO 等公共风险情报；
- 外部数据源；
- Walmart 后台报错记录；
- Walmart 上架报错记录；
- 从 Team 私有黑名单中由系统运营方选择后提升为公共记录。

当前已经确认：

> Team 的 Walmart 后台报错记录与 Walmart 上架报错记录，可以按明确规则自动沉淀到 System Public Blacklist。

其他 Team Private Blacklist 记录：

> 系统运营方可以查看，并选择是否加入 System Public Blacklist。

---

## 7. Team Private Blacklist

每个 Team 可以维护自己的：

> **Team Private Blacklist**

Team A 的私有黑名单默认不等于 Team B 的私有黑名单。

它表达：

> 该 Team 自己明确决定要规避的对象。

---

## 8. Team Whitelist

每个 Team 可以维护自己的：

> **Team Whitelist**

但 Whitelist 的作用范围必须明确：

> **Whitelist 只覆盖 Blacklist Resolution，不覆盖不可豁免的法律 / 平台硬规则。**

例如：

```text
System Public Blacklist:
Brand A = Blacklisted

Team X Whitelist:
Brand A = Allowed
```

如果该记录只是系统风险经验 / 普通黑名单：

> 对 Team X 不按这条 Blacklist Hit 拒绝。

但是，如果当前 Product Audit 另外确认：

- 命中 TRO；
- 命中目标平台明确禁售 / 禁止销售规则；

则仍然必须按硬规则 Reject。

因此：

> Whitelist 可以覆盖“名单结论”，不能把已确认的不可覆盖硬风险变成 Allowed。

---

## 9. Blacklist Resolution 优先级

原优先级：

```text
Team Whitelist
>
Team Private Blacklist
>
System Public Blacklist
```

继续成立，但其含义限定为：

> **Blacklist Resolution Priority**

即只解决“对于当前 Team，这条 Blacklist 是否有效命中”。

执行顺序概念上应为：

```text
第一层：不可覆盖硬规则
        ↓
第二层：Team Whitelist / Team Private Blacklist / System Public Blacklist
        ↓
第三层：其他辅助风险
        ↓
Product Audit Decision
```

因此：

1. 先判断不可覆盖硬规则；
2. 在 Blacklist 层内，Team Whitelist 可以覆盖 Team Private / System Public Blacklist；
3. 如果没有 Whitelist Override，再依次判断 Team Private Blacklist、System Public Blacklist；
4. 其他辅助风险按 Team Policy 参与最终判断。

风险判断必须带 Team Context。

系统不能只问：

> “Brand A 是否在黑名单？”

而应该问：

> “对于 Team X，Brand A 当前的 Effective Blacklist Result 是什么？”

### 9.1 当前不可被 Whitelist 覆盖的硬规则

当前至少包括：

- 确认命中的 TRO；
- Target Platform 明确禁止销售 / 禁售的商品规则。

例如 System Public Blacklist Entry 的 Source 标记为 TRO，同时 Team 对该 Brand 设置 Whitelist：

- Blacklist 层可以得到 Whitelist Override；
- 但 Product Audit 仍必须独立执行当前 TRO Hard Rule；
- 如果当前证据确认 TRO 仍命中，最终仍为 Reject。

这样最终结果不依赖代码“先判断 Whitelist 还是先判断 TRO”的偶然执行顺序。

---

## 10. Audit 与 Blacklist 分开

重要边界：

```text
Product Audit
≠
Blacklist
```

Product Audit 可以查询 Blacklist。

但：

> 一次 Audit Reject 不自动创建 Blacklist Entry。

例如商品因为数据不足、平台政策或其他原因 Reject，不代表 Brand / Product / Seller 必须进入黑名单。

---

## 11. 上架前命中风险

当前团队在 Workflow 模式下：

```text
Product Audit
↓
Effective Blacklist Check
↓
Hit
↓
Reject
```

即：

> 上架前命中当前 Team 的有效黑名单以后，直接 Reject / 不允许继续上架。

这是当前 Team Workflow / Risk Policy。

其他 Team 可以定义不同策略。

---

## 12. 在线商品命中新风险

如果商品已经在线，后来新增：

- Brand Blacklist；
- Product Blacklist；

当前团队默认：

```text
New Effective Risk Match
↓
Pending / Operational Recommendation
↓
Operations Decision
```

即先产生：

> 待处理风险建议 / 运营建议。

不会因为 Risk Intelligence 更新本身就强制所有 Team 自动下架。

---

## 13. 自动下架属于 Workflow

如果某个 Team 希望：

```text
Online Listing
↓
Blacklist Hit
↓
Automatic Take Down / Exit
```

可以通过 Team Workflow 配置。

因此：

> Risk Intelligence 提供风险事实；Workflow 决定命中以后执行什么动作。

---

## 14. Amazon Seller Blacklist 的作用范围

Amazon Seller Blacklist 主要影响：

> Source / Procurement Selection。

例如某 Seller 被拉黑以后：

- Product Source Selection 可以排除该 Seller；
- Procurement 可以避免使用该 Seller；
- 新采集商品可以产生风险提示。

它不自动意味着：

> Product 本身永久不可销售。

因此 Brand / Product Risk 与 Source Seller Risk 可以统一存在 Risk Intelligence 中，但影响的业务环节不同。

---

## 15. 当前确认原则

1. 风险对象当前主要包括 Brand、Product、Amazon Seller。
2. Product 通常按 ASIN / Product Identifier 识别。
3. Amazon Seller 通常按 Seller ID 识别。
4. 风险信号只有明确加入后才成为正式黑名单。
5. 风险来源可以是人工、TRO、平台违规经验、Walmart 报错记录和外部数据。
6. ERP 可以存在 System Public Blacklist。
7. System Public Blacklist 可按 Team 开放，也可以成为付费服务。
8. Team 可以拥有 Team Private Blacklist。
9. Team 可以拥有 Team Whitelist。
10. Team Whitelist > Team Private Blacklist > System Public Blacklist 仅表示 Blacklist Resolution Priority。
11. Whitelist 不能覆盖确认命中的 TRO 或 Target Platform 明确禁售等不可覆盖硬规则。
12. 风险判断必须结合具体 Team。
12. Audit 可以查询 Blacklist，但 Audit Reject 不自动写入 Blacklist。
13. 当前团队上架前命中有效黑名单时，Workflow 直接 Reject。
14. 在线商品命中新风险时，当前默认先产生运营待处理建议。
15. 自动下架 / Exit 属于 Team Workflow，而不是 Risk Intelligence 固定动作。
16. Seller Blacklist 主要作用于 Source / Procurement Selection。
17. Brand / Product Blacklist 主要作用于 Product Audit 与 Online Listing Risk。
18. 系统运营方可以选择将 Team 风险记录提升到 System Public Blacklist。

---

## 16. 建模结论

Risk Intelligence / Blacklist 真正回答的是：

> **对于某个 Team，这个 Brand / Product / Seller 当前是否属于需要规避的风险对象？**

它不负责决定：

- Product Audit 最终如何结束；
- Listing 如何下架；
- Purchase 如何处理。

这些动作分别由：

- Audit；
- Procurement；
- Workflow；

根据 Effective Risk Result 继续执行。
