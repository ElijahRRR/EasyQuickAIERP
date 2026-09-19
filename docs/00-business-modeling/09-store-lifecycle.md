# 店铺生命周期业务模型 v0.2

> 状态：已确认  
> 所属阶段：ERP 阶段 0 — 业务建模

# 1. Store 的业务定位

Store 表示：

> 一个 Team 在某个 Marketplace 上接入和经营的具体销售店铺。

Store 是以下业务对象的重要归属基础：

- Listing；
- Sales Order；
- Return / Refund；
- Settlement；
- Payout；
- Profit；
- Workflow；
- Store-level Policy。

---

# 2. Store 的稳定平台身份

当前 Walmart Store 的主要平台身份包括：

- Marketplace；
- Partner ID / Seller ID。

业务上可理解为：

```text
Marketplace
+
Platform Seller Identity
=
External Store Identity
```

Store Name 不属于稳定平台身份。

---

# 3. Store Name 可以改变

例如：

`A001张三`

修改为：

`A001李四`

仍然是同一个 Store。

因此：

- Store Name 可以修改；
- 当前负责人可以修改；
- 当前运营组可以修改；

这些变化都不能创建新的 Store。

---

# 4. API Credential 不属于 Store Identity

Credential 用于：

> ERP 与目标 Marketplace 建立连接。

Credential 可以：

- 更新；
- 失效；
- 重新授权；
- 被移除。

因此：

> Credential 属于 Store Connection，而不是 Store Identity。

---

# 5. 新 Store 接入流程

当前流程：

```text
选择 Marketplace
↓
创建 Store
↓
填写 Store Name
↓
填写 API Credential
↓
Test Connection
↓
Enable Store
↓
开始同步
```

当前启用前最低要求：

- Store Name；
- API Credential。

其他经营配置允许后补。

---

# 6. Store 有三类独立状态

Store 至少需要分别维护：

1. Platform State；
2. ERP Enable State；
3. Connection Health。

这三类状态不能合并成一个 `status`。

---

# 7. Platform State

表示：

> Marketplace 当前认为这个 Store 是什么状态。

例如 Walmart 可能出现：

- ACTIVE；
- SUSPENDED；
- TERMINATED；
- 其他平台状态。

这是：

> Platform Fact。

ERP 负责保存平台实际返回结果。

---

# 8. ERP Enable State

表示：

> 当前 Team 是否允许 ERP 正常使用这个 Store。

例如：

- Enabled；
- Disabled。

可能出现：

```text
Platform = ACTIVE
ERP = Disabled
```

即：

> 平台店铺正常，但 Team 主动不再让 ERP 对其执行正常经营流程。

---

# 9. Connection Health

表示：

> ERP 当前是否能够正常连接这个 Store。

例如：

- Healthy；
- Credential Invalid；
- Token Failed；
- Connection Failed；
- Unknown。

Connection Health 与 Platform State 独立。

例如：

```text
Platform State = ACTIVE
Connection Health = Token Failed
```

不能因此把店铺标记为：

- SUSPENDED；
- TERMINATED。

---

# 10. Store Ownership / Assignment 不是状态

Store 还存在组织归属关系，但它不属于 Store State。

需要分别理解：

## Team Ownership

回答：

> 这个 Store 属于哪个 Team。

一个 Store 在某一个接入关系中属于一个 Team。

Team 更接近：

> 一个公司 / 独立经营工作空间。

## Operation Group Assignment

回答：

> 当前由 Team 内哪个运营组管理。

可以变化。

## Operator Assignment

回答：

> 当前由谁负责这个 Store。

也可以变化。

因此：

```text
Store State
≠
Store Ownership
≠
Store Assignment
```

---

# 11. Team Ownership 与内部负责人变化不同

同一个 Team 内：

```text
Store H001
负责人：张三
运营组：A组
```

以后可以变为：

```text
Store H001
负责人：李四
运营组：B组
```

仍然是同一个：

- Store；
- Team Ownership。

改变的只是：

> Internal Assignment。

---

# 12. Store Configuration

每个 Store 都允许拥有自己的：

- Operating Category；
- Product Price Range；
- Fulfillment Mode；
- Product Allocation Rule；
- Brand Allocation Rule；
- Estimated Commission；
- Procurement Cost Coefficient；
- Warehouse；
- Workflow；
- Risk Policy。

因此即使多个 Store 属于同一 Team：

> Store-level Policy 仍然可以不同。

---

# 13. Suspended 不代表 ERP 停止管理

如果：

```text
Platform State = SUSPENDED
```

只表示：

> 平台经营能力受限。

ERP 仍然可能继续管理：

- Existing Orders；
- Procurement；
- Logistics；
- Return / Refund；
- Settlement；
- Payout；
- Historical Listing；
- Profit。

因此：

> `SUSPENDED ≠ Stop Store Management`

---

# 14. Terminated 也不代表数据结束

即使：

```text
Platform State = TERMINATED
```

Store 仍可能存在：

- Remaining Orders；
- After-sales；
- Refund；
- Chargeback；
- Settlement；
- Remaining Balance；
- Final Payout。

因此：

> `TERMINATED ≠ Delete Store`

也不等于：

> Historical Business Closed。

---

# 15. Store 删除的正式业务定义

已经确认：

> 人工“删除 Store”删除的是 Store 当前接入 / 激活关系。

因此删除动作更准确地表示：

```text
Unbind / Remove Store Connection
```

而不是：

```text
Delete Store Business History
```

删除 / 解绑以后：

- Store 不再出现在正常 Active Store 列表；
- Credential 被移除或停用；
- 不再进行正常同步和 Workflow；
- 当前接入关系结束。

但是历史：

- Order；
- Listing；
- Purchase；
- Return；
- Settlement；
- Payout；
- Profit；

全部继续保留。

---

# 16. 历史数据必须保留原 Store 身份

即使 Store 已经 Unbind：

历史业务对象仍然需要知道：

> 当时属于哪个 Store。

因此不能因为：

> Store 从当前接入列表删除

就破坏历史订单与 Store 的关系。

---

# 17. 不允许 Team A 直接 Transfer 到 Team B

不同公司属于不同 Team。

ERP 不支持普通意义上的：

```text
Team A
→ Transfer Store
→ Team B
```

如果外部店铺需要更换公司管理：

```text
Team A
→ Unbind Store
```

随后：

```text
Team B
→ 自己重新 Bind Store
```

这是：

> 两个独立 Team 的接入行为。

不是内部 Store Transfer。

---

# 18. Team 历史数据严格隔离

Team B 后来重新绑定同一个外部 Marketplace Seller：

> 不应该自动获得 Team A 之前的 ERP 历史业务数据。

Team A 原来的：

- Orders；
- Profit；
- Settlement；
- Internal Operations；

继续属于：

> Team A Historical Data。

---

# 19. 负责人 / 运营组需要保存历史归属

已经确认采用：

> **方案 B：保留历史责任归属。**

因此不能只保存 Store 当前：

`operator_id`

然后所有历史数据都动态显示当前负责人。

---

# 20. Assignment 需要具有时间有效范围

例如：

```text
2026-01-01 ~ 2026-06-30
Store H001
负责人 = 张三
运营组 = A组
```

之后：

```text
2026-07-01 ~
Store H001
负责人 = 李四
运营组 = B组
```

这样才能知道：

> 某个历史时间点到底是谁负责。

---

# 21. 历史订单归属按业务发生时的 Assignment

例如：

张三负责期间产生的订单：

> 继续归张三。

李四接手以后产生的新订单：

> 归李四。

不会因为 Store 当前负责人变成李四：

> 就把过去张三时期的订单一起改成李四。

---

# 22. 历史利润同样保留当时归属

例如：

张三负责期间产生的订单：

`Order A`

后续几个月后才发生 Refund 或 Settlement Adjustment。

即使这时 Store 已经归李四：

> Order A 的经营责任仍然属于原历史 Assignment。

也就是说：

> 财务结果可以以后才变化，但责任归属不应该跟着当前负责人变化。

这一点对于运营绩效尤其重要。

---

# 23. 运营组同样保留历史

例如：

```text
Jan-Jun → Group A
Jul-Dec → Group B
```

则历史统计：

- Jan-Jun → Group A；
- Jul-Dec → Group B。

不因为当前 Store 属于 Group B：

> 就把全年数据全部计算给 Group B。

---

# 24. Store 当前归属与历史归属需要同时存在

系统需要同时回答两种问题：

## Current Assignment

> 这个 Store 现在谁负责？

## Historical Assignment

> 某张订单产生时谁负责？

这两个查询用途不同，都需要支持。

---

# 25. 当前 Store 结构概念

目前 Store 在业务层可以理解为：

```text
Store
│
├── Identity
│   ├── Marketplace
│   └── Platform Seller ID
│
├── States
│   ├── Platform State
│   ├── ERP Enable State
│   └── Connection Health
│
├── Connection
│   └── Credentials
│
├── Configuration
│   ├── Category
│   ├── Fulfillment
│   ├── Cost Policy
│   ├── Warehouse
│   ├── Workflow
│   └── Risk Policy
│
└── Ownership / Assignment
    ├── Team
    ├── Operation Group History
    └── Operator History
```

---

# 26. Store 生命周期

当前生命周期可以概括为：

```text
Select Marketplace
↓
Create Store
↓
Configure Name + Credentials
↓
Test Connection
↓
Enable
↓
Operate
```

运行期间独立维护：

```text
Platform State
ERP Enable State
Connection Health
```

同时维护：

```text
Operator / Group Assignment History
```

Store 停止使用：

```text
Disable / Unbind
↓
Remove Credentials
↓
Stop Active Operations
↓
Historical Data Retained
```

---

# 27. 当前确认的 Store 业务原则

1. Store 属于某个 Marketplace。
2. Marketplace + Platform Seller Identity 是重要外部身份。
3. Store Name 不是 Store Identity。
4. API Credential 不是 Store Identity。
5. 新 Store 接入需要 Store Name + Credential。
6. Platform State、ERP Enable State、Connection Health 是三个独立状态维度。
7. Business Ownership 不属于 Store State。
8. Team Ownership、Operation Group Assignment、Operator Assignment 属于组织关系。
9. Connection Failed 不得自动推断 Platform Suspended / Terminated。
10. 不同 Store 允许不同经营配置。
11. Suspended 后 ERP 仍可继续管理现有业务。
12. Terminated 后历史及后续财务业务继续保留。
13. 人工删除 Store 实际表示 Unbind / Remove Active Connection。
14. Unbind 不删除历史数据。
15. Team A 不能直接 Transfer Store 给 Team B。
16. 跨公司更换管理主体由原 Team Unbind、新 Team 自行 Bind。
17. 不同 Team 的历史数据保持隔离。
18. Store 可以在 Team 内更换负责人。
19. Store 可以在 Team 内更换运营组。
20. 负责人和运营组变化必须保存历史 Assignment。
21. 历史订单按业务发生时的负责人 / 运营组归属。
22. 后续 Settlement / Refund 改变金额时，不改变该历史订单原来的责任归属。
23. Store 当前归属和历史归属都需要可查询。

---

# 28. Store 生命周期建模结论

Store 不应该被理解成：

> 一个名称 + 一套 API Credential。

更准确的是：

> **一个 Team 对某个平台销售主体的接入关系，以及围绕该主体持续存在的经营配置、平台状态、连接状态和组织责任历史。**

Store 的三类状态是：

```text
Platform State
ERP Enable State
Connection Health
```

组织关系则独立存在：

```text
Team Ownership
Operator Assignment History
Operation Group Assignment History
```

而 Store 删除：

> 结束的是当前 ERP 接入关系，不是历史业务数据。
