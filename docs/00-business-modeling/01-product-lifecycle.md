# 商品生命周期业务模型 v0.6

> 状态：已确认业务基线  
> 适用范围：描述当前团队已确认的商品业务事实；其中部分步骤是可选业务能力，不代表所有团队必须经过。

## 1. 总原则

当前团队的商品主线可以概括为：

```text
商品发现
→ 商品资料采集 / 导入
→ 进入团队商品池
→ （可选）商品审核
→ 确定销售平台
→ （可选）店铺分配
→ 确定店铺
→ 确定主来源 / 备用来源
→ 整理并检查刊登资料
→ 按目标平台政策和 Spec 补充资料
→ 形成 Listing Draft
→ 创建 Listing
→ 平台处理
→ 在线经营与风险监控
→ 继续销售 / 调整 / 暂停 / 下架 / 永久退出
```

重要：

- 商品审核不是所有团队的必经步骤；
- 店铺分配也不是所有团队的必经步骤；
- ERP 应先提供这些独立能力，再由 Workflow 决定各团队采用哪些步骤以及执行顺序；
- 当前团队的流程未来可以作为 Workflow Template。

---

## 2. 商品发现

当前商品进入业务视野的主要方式：

1. 在卖家精灵按销量、价格等条件批量获取产品，再交给采集器采详情；
2. 运营中发现某个 Seller 的产品有销量且长期没有明显被 Walmart 判违规，则采集该 Seller；
3. 人工在 Sorftime 看 Walmart 最新关键词搜索量，再到 Amazon 按关键词、价格区间、配送方式等条件采集；
4. 从 Walmart 竞品发现商品；
5. 从 1688 等供应端发现商品；
6. Excel / CSV 批量导入。

未来还可包括 API、人工创建、其他供应平台等入口。

---

## 3. 团队商品池

商品池按团队独立。

即使两个团队采集到同一个 ASIN，也分别形成各自独立的经营对象，可以拥有不同：

- 来源关系；
- 审核记录；
- 目标销售平台；
- 店铺分配；
- Listing 资料；
- 团队经营策略；
- 风险处理结果。

进入商品池只表示：

> 商品已经进入该团队的候选经营体系。

不代表已经允许销售、确定平台、确定店铺或准备好刊登。

---

## 4. “选中商品”不是业务状态

页面勾选只是一种临时 UI 操作，可用于批量审核、分配、上架、修改等。

必须区分：

- UI Selection：临时勾选 / 筛选结果；
- Business State：待审核、通过、拒绝、待处理、待刊登、在线、暂停、退出等持久状态。

---

## 5. 商品审核（可选能力）

### 5.1 目的

商品审核回答：

> 当前商品是否支持上架到指定销售平台。

审核不负责判断商品值不值得卖、利润高不高、应该进入哪家店铺。

### 5.2 输入

最低至少包括：

- Title；
- Brand；
- Description。

更多可包括：

- Category / Product Type；
- Specifications；
- Material；
- Images；
- Bullet Points；
- Claims；
- Dimensions；
- Attributes；
- 其他能帮助理解商品属性的数据。

### 5.3 硬判断

当前正式 Pass / Reject 的硬条件主要是：

1. TRO 风险；
2. 目标平台禁用 / 禁售产品政策。

注意：某品牌存在于 USPTO 商标数据库，本身并不等于 Reject。

### 5.4 辅助知识产权风险

以下当前属于辅助风险，而非默认硬拒绝：

- 普通商标；
- 专利；
- 版权；
- 其他知识产权风险。

团队未来可以将某类辅助风险升级为自己的硬拒绝规则，但这属于 Team Policy。

### 5.5 审核结果

- Pass：未命中 TRO 硬风险，未违反目标平台禁售政策，且数据足够完成判断；
- Reject：命中至少一个硬拒绝条件；
- Pending：商品数据不足、必要字段缺失、LLM 无法判断、依赖不可用或其他信息不足。

Pass 不等于没有任何辅助风险。

### 5.6 审核不会产生黑名单

审核只产生结果、原因、辅助风险和证据。

Blacklist / Risk Intelligence 是独立体系，不因一次审核自动生成。

### 5.7 审核结果必须绑定审核上下文

Product Audit 不能只保存一个长期不变的 `Pass / Reject / Pending`。

每一次正式审核至少需要能够还原：

- Product；
- Target Platform；
- 审核时使用的 Content Version；
- 审核时使用的 Source / Evidence；
- Audit Rule / Policy Version；
- Audit Time；
- Audit Result；
- 必要的 Actor / Trigger Source。

因此：

> **审核结果是针对“某个平台 + 某一版内容 + 某一批证据 + 某一版规则”得出的历史结论。**

历史 Audit Record 不因后续变化而被覆盖。

### 5.8 Audit Result 与 Audit Validity 分开

正式 Audit Result 仍然只有：

- Pass；
- Reject；
- Pending。

另外单独判断该历史审核结论当前是否仍可用于证明当前版本，例如：

- Valid；
- Stale / Re-audit Required。

旧 Pass 变为 Stale 时：

> 不能把原 Pass 改写成 Reject，也不能删除原审核记录。

它仍然表示当时版本在当时规则下通过，只是不能继续证明当前版本已通过。

至少以下情况可能使现有审核结论失效：

1. 审核所依赖的重要内容发生实质变化，例如 Brand、Title、Description、Product Type、Category、Material、Claims、关键规格等；
2. Target Platform 改变；
3. Audit Rule / Platform Policy 版本变化，而且变化可能影响该 Product；
4. 原审核依赖的关键证据被证明错误或失效；
5. 出现新的不可覆盖硬风险，例如确认命中 TRO。

仅切换 Primary / Backup Source：

> 如果仍然确认是同一个 Product，且审核涉及的商品事实没有实质变化，不自动使 Product Audit 失效。

当前不采用“审核超过固定 30 / 60 / 90 天自动过期”的统一规则。

### 5.9 Product Audit 与 Listing 提交前检查分开

必须区分：

```text
Product Audit
= 这个商品是否允许面向目标平台经营

Listing Pre-submit Validation
= 当前这版 Final Listing Data 是否满足目标平台要求

Order Audit
= 已产生订单当前是否允许继续履约
```

Product Audit Pass 不代表以后任意 AI / Human 修改后的 Listing 文案自动合规。

Listing 提交前仍必须针对：

- 当前 Final Listing Value Version；
- 当前 Target Platform；
- 当前适用规则；

执行相应校验。

---

## 6. 在线风险扫描与重新审核

在线商品每天会扫描新增：

- TRO 风险；
- 品牌黑名单；
- 商品黑名单；
- 其他明确风险信息。

这不是每天重新执行完整审核。

完整 Re-Audit 当前没有固定周期，通常在审核规则或平台政策逻辑变化后，对指定产品或指定影响范围重新审核。

---

## 7. 确定销售平台

审核通过（如果团队启用了审核）后，可确定商品进入哪个销售平台。

未来可能包括：

- Walmart；
- Amazon；
- eBay；
- TikTok Shop；
- SHEIN；
- Temu；
- 其他平台。

一个 Product 可以进入多个平台。

---

## 8. 店铺分配（可选能力）

### 8.1 不是必经步骤

并非所有团队都需要店铺分配。

例如：

- 团队只有一家店；
- 商品从一开始就指定店铺；
- 外部系统已指定店铺；
- 团队完全人工指定。

### 8.2 系统能力与团队策略分开

ERP 底层允许：

```text
Product → 多 Platform → 多 Store
```

ERP 不统一限制：

- 一个商品能进入多少店铺；
- 一个品牌能进入多少店铺。

当前团队可能规定：一个产品只能存在一个店，一个品牌只能存在一个店；其他团队可采用不同规则。

### 8.3 店铺分配方式

最终分配是：

> 团队定义规则，系统根据规则计算推荐结果。

系统不替团队发明经营策略。

### 8.4 硬条件

当前已确认：

- 价格区间；
- 配送方式；
- 类目。

不符合任一硬条件的 Store，直接从候选中排除。

### 8.5 排序因素

通过硬条件以后，可按团队规则参考：

- 来源评分；
- 来源销量；
- 商品实际销售数据；
- 店铺销量；
- 店铺销售额；
- 当前商品结构；
- 其他经营指标。

这些属于 Ranking Factors，而不是 Eligibility。

### 8.6 推荐结果

系统输出 Store Recommendation。

未来团队可选择：

- 人工确认推荐；
- 自动接受第一推荐；
- 批量确认；
- 完全人工指定。

---

## 9. 平台资源引用（按平台需要）

Store 确定以后，某些平台操作可能需要引用平台自身已经存在的资源，例如：

- Platform Warehouse；
- Shipping Template；
- 其他平台级 Fulfillment / Inventory Resource。

这些资源：

> 不属于 Product 生命周期的固定层级，也不代表 ERP 已经拥有实体 Warehouse / WMS Domain。

因此通用关系保持为：

```text
Product
→ Platform
→ Store
→ Listing
```

Listing 上架、库存维护等操作是否需要 Platform Resource、是否可省略、以及平台默认行为，由对应 Platform Capability Specification 定义。

当前 Walmart 业务中，ERP 需要同步并保存 Walmart Platform Warehouse 等平台资源引用，但不在阶段 0 将其建模为 ERP 自有实体仓库。

---

## 10. 多来源商品

一个 Product 可以关联多个 Source，例如：

- Amazon Source A；
- Amazon Source B；
- 1688 Source C。

### 主来源

由人工选择当前主来源，主要作为：

- Cost；
- Price Reference；
- Stock；
- Shipping；
- Delivery；
- Source Product Data。

### 备用来源

允许配置多个备用来源。

### 来源切换

来源切换：

- 不改变内部 Product 身份；
- 本身不改变审核结果；
- 本身不改变已有 Listing 身份。

因此 Source Relation、Product、Listing 是三个不同层次。

### 来源商品与 Seller Offer 分开

Amazon Product Source 当前按 ASIN 级识别。

Seller、Price、Stock、Fulfillment、Delivery 等属于某个时间点的可购买条件，不构成 Source 的永久身份。

因此概念上需要区分：

```text
Product Source
= 来源商品是谁（例如 ASIN）

Source Offer Snapshot
= 某个时间点由哪个 Seller、以什么价格、库存和配送条件提供
```

同一个 ASIN 的 Seller 变化不创建新的 Product Source，但 Order Audit / Procurement 等需要保留当时实际观察到的 Offer Evidence。

### Amazon 变体关系

Amazon Source 需要能够表达 Parent / Child ASIN 与具体 Variant 信息，例如：

- Parent ASIN；
- Child / Purchasable ASIN；
- Color；
- Size；
- Pack / Quantity；
- 其他关键变体属性。

实际采购与履约应尽可能落到具体可购买的 ASIN / Variant。

当人工把多个 ASIN 关联到同一个 Product 时，需要确认：

> 这些具体来源变体确实能够履约同一个 Product，而不是仅仅标题相似。

系统 / AI 可以给出候选和差异提示，但默认不能自动建立这种同商品关系。

---

## 11. 商品资料准备

目标：形成满足目标平台政策和 Spec 的 Listing Draft。

原则：

> 能直接继承来源就继承，不符合要求再处理。

所有最终提交字段原则上都要经过平台规则检查，包括：

- Title；
- Brand；
- Description；
- Images；
- Key Features；
- Category；
- Product Type；
- Material；
- Dimensions；
- Attributes；
- 其他 Spec 字段。

“不符合要求”的典型情况：

- 来源 Title 188 字符，平台要求 ≤150；
- 来源只有 1 张图片，平台要求至少 3 张；
- 来源描述内容不足，平台要求更多结构化内容。

所有字段允许人工编辑，也允许 AI 参与优化、生成和补全。

当前团队实际主要对不符合要求的 Title、Brand、Description 做优化，其余字段若符合要求则直接继承来源。

Brand 可以由各团队自行指定，但最终仍必须满足目标平台 Brand 字段政策。

---

## 12. 商品字段的四层状态

关键 Listing Field 需要区分：

1. Source Value；
2. AI Suggested Value；
3. Human Edited Value；
4. Final Listing Value。

### Source Value

来源平台 / 供应源获得的原始数据。

### AI Suggested Value

AI 根据来源数据、目标平台政策、Spec 和业务上下文生成的建议值。

AI 建议不能覆盖 Source Value。

### Human Edited Value

人工主动修改后的值。

后续 AI 重新运行也不能静默覆盖人工意图。

### Final Listing Value

最终决定提交给平台的值。

可以来自：

- Source → Final；
- AI Suggested → Final；
- Human Edited → Final。

平台提交默认读取 Final Listing Value。

四层值并不要求每个字段都同时存在。如果 Source 已经合规，AI / Human 可为空，Final 直接采用 Source。

---

## 13. Spec 属性补全

确定 Platform / Product Type 后，根据 Spec 填写属性。

AI 可以参考：

- Title；
- Description；
- Bullet Points；
- Material；
- Specifications；
- Dimensions；
- Images；
- 其他 Source Data。

如果来源中没有明确值：

```text
AI Extraction / Inference
→ 仍无法获得
→ Field-specific Fallback
```

具体 fallback 属于平台、Product Type、Team Policy 的组合，阶段 0 不统一定义。

---

## 14. 不同 Listing 可以使用不同资料

同一个 Product 在不同 Platform / Store 中可以拥有不同：

- Title；
- Brand；
- Description；
- Images；
- Price；
- Warehouse；
- Attributes。

因此 Product Information 与 Listing Information 必须区分。

---

## 15. 在线经营

Listing 成功以后持续关注：

### 来源端

- 来源价格；
- 来源库存；
- 来源运费；
- 来源是否有效；
- 配送时间变化。

### 平台端

- 平台售价；
- 平台库存；
- Listing 状态；
- Unpublished Error。

### 风险端

- TRO；
- Brand Blacklist；
- Product Blacklist；
- Platform Policy Change。

### 经营端

- Orders；
- Sales；
- Profit。

在线经营不断循环：

```text
更新来源
→ 同步平台状态
→ 风险扫描
→ 观察销售表现
→ Continue / Change Price / Change Inventory / Change Source / Pause / Re-list / Exit
```

---

## 16. 商品退出

### 可恢复退出

例如：

- 来源缺货；
- 配送过慢；
- 人工停止；
- 店铺问题；
- 来源失效。

未来可更换 Source / Store，调整平台资源配置，或者恢复、重新上架。

### 永久或高风险退出

例如：

- TRO；
- 高风险品牌；
- 平台明确禁售；
- 其他确定性严重风险。

可能进一步影响 Listing、Product、Related Listings、Brand Risk State。

退出是业务状态变化，不等于删除 ERP 历史数据。

---

## 17. 当前已确认原则

1. 商品池按团队独立；
2. 所有候选商品都可以进入商品池；
3. 审核是可选业务能力，不是所有团队强制流程；
4. TRO + 平台禁售政策是当前审核硬判断；
5. 普通商标、专利、版权等属于辅助风险；
6. 审核本身不会产生黑名单；
7. 在线商品每天执行新增风险扫描；
8. 完整重新审核主要由规则变化触发；
9. 先确定销售平台，再决定是否进入店铺分配；
10. 店铺分配是可选能力；
11. 店铺规则由团队定义，系统给推荐结果；
12. 价格区间、配送方式、类目属于店铺分配硬条件；
13. 来源表现、商品表现、店铺经营表现属于排序因素；
14. ERP 不统一限制 Product / Brand 可以进入多少 Store；
15. 一个 Product 可以存在多个 Source；
16. 人工指定主 Source，允许备用 Source；
17. Source 切换不改变 Product / Audit / Listing 身份；
18. 所有 Listing 字段都需要目标平台规则检查；
19. 合规 Source 可直接继承；
20. 所有字段允许人工编辑和 AI 参与；
21. 字段需区分 Source / AI Suggested / Human Edited / Final；
22. AI Suggested 不等于 Final；
23. Human Edited 不应被 AI 静默覆盖；
24. Final Listing Value 是平台发布真值；
25. 正常商品上架不要求逐件人工批准；
26. 当前团队流程未来可以作为 Workflow Template，但不成为 ERP 强制流程。
27. Platform Warehouse 等平台资源不属于 Product 生命周期的固定层级。
28. 当前阶段不建立 ERP 自有 Physical Warehouse / WMS Domain；未来出现真实仓储履约需求时再独立建模。
29. Product Audit 必须绑定 Target Platform、Content Version、Evidence 和 Rule / Policy Version。
30. Audit Result 与 Audit Validity 分开，历史 Pass 可以变为 Stale，但不能被覆盖成新的结果。
31. Product Audit、Listing Pre-submit Validation、Order Audit 是三个不同判断。
32. Product Source 按来源商品身份管理；Seller / Price / Stock / Delivery 属于可变化的 Offer 条件。
33. Amazon Parent / Child ASIN 与具体 Variant 需要能够被表达。
34. 多 ASIN 归入同一 Product 原则上由人工确认具体变体是否可互相履约。
