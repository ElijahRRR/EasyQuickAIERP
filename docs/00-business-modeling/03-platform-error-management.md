# Platform Error Management 业务模型 v0.1

> 状态：已确认业务基线

## 1. 目标

Platform Error Management 用于管理 Listing 与销售平台交互过程中产生的异常、失败和取消发布问题。

核心需要回答：

1. 发生了什么错误；
2. 错误发生在哪个阶段；
3. 根因属于哪一类；
4. 是否可恢复；
5. 当前团队希望怎么处理；
6. 实际采取了什么动作；
7. 处理后平台是否恢复正常。

---

## 2. 首次上架失败与上线后 Unpublished 必须区分

### Initial Listing Error

```text
Listing Submit
→ Submitted / Processing
→ Failed
```

表示首次上架尝试没有成功形成正常 Published Listing。

### Post-Publish Error

```text
Published
→ Unpublished
```

表示一个原本正常经营的 Listing 后来被平台取消发布。

这两类问题业务背景不同，不能合并。

---

## 3. 报错判断依据

当前实际处理不是只看一个错误字段，而是综合：

- `unpublishedReasons`；
- 商品详情；
- Platform Status；
- Feed / Submission Result；
- 相关 Listing / Product 数据；
- 必要时原始平台响应。

这些共同构成 Error Evidence。

---

## 4. 基本处理过程

```text
Platform Error Detected
→ Collect Evidence
→ Normalize Error
→ Classify Error
→ Determine Root Cause
→ Apply Team Resolution Policy
→ Execute Resolution
→ Verify Platform Result
→ Resolved / Still Failing / Changed Error / Waiting / Listing Exit
```

---

## 5. Error Classification 与 Resolution Policy 必须分离

### Error Classification

回答：

> 这到底是什么错误？

主要由 Platform、Error Code、Error Message、Listing State、Product Data 等决定。

原则上不因 Team 不同而改变。

### Resolution Policy

回答：

> 当前 Team 遇到这个错误准备怎么办？

同一种 Error 可以在不同 Team 使用不同策略。

例如同一个 Content Error：

- Team A：自动修复并重新提交；
- Team B：暂停并等待进一步处理；
- Team C：直接退出 Listing。

因此“错误是什么”和“怎么处理”不能写死在一起。

---

## 6. 当前一级错误分类

至少包括：

1. Content / Listing Data；
2. GTIN / UPC / Product Identifier；
3. Category / Product Type / Spec；
4. Brand；
5. Compliance / Policy；
6. Inventory / Price；
7. Store / Account；
8. Platform System；
9. Time / Date / Expiration；
10. Unknown / Unclassified。

未来平台可继续增加子类型，但一级分类尽量保持稳定。

---

## 7. Content / Listing Data Error

可能涉及：

- Title；
- Description；
- Images；
- Key Features；
- Attributes；
- Required Fields；
- 字段格式；
- 字符长度；
- 数据缺失；
- 其他内容资料问题。

通常 Potentially Recoverable。

当前团队尚未建立完整的内容自动修复并重新提交逻辑。

---

## 8. GTIN / UPC / Product Identifier Error

可能涉及：

- Invalid GTIN / UPC；
- Identifier Conflict；
- Identifier Already Used；
- Product ID Mapping；
- Identifier 与商品不匹配；
- 其他产品标识问题。

不能简单定义成“换 UPC 就行”。

必须继续细分具体错误，再决定 Fix / Replace / Match / Re-submit / Exit。

---

## 9. Category / Product Type / Spec Error

可能涉及：

- Category 不正确；
- Product Type 不正确；
- Required Attribute Missing；
- Attribute Value Invalid；
- Product Type 与商品内容不一致；
- 其他 Spec 问题。

当前尚未形成完整自动修复逻辑。

未来可能通过：

- 重新确定 Product Type；
- 修复 Spec；
- AI 重新提取字段；
- 补充缺失属性；
- 重新提交。

---

## 10. Brand Error

可能涉及：

- Brand 不允许；
- Brand 与 Product 不匹配；
- Brand 权限；
- Brand Risk；
- 其他品牌问题。

当前团队默认处理：

> 放弃当前 Listing。

这只是当前 Team Policy，不是 ERP 全局规则。

---

## 11. Compliance / Policy Error

可能包括：

- Prohibited Product；
- Platform Policy Violation；
- TRO / Risk；
- 其他合规限制。

当前团队默认：

> 放弃当前 Listing。

是否进一步影响 Product、Brand、其他 Store / Platform Listing，由独立风险体系决定，Platform Error Management 不擅自扩散。

---

## 12. Inventory / Price Error

包括：

- Inventory Invalid；
- Price Invalid；
- Price Constraint；
- Inventory Submission Failure；
- 其他价格库存问题。

价格和库存属于日常维护能力，这类错误通常偏 Operational Recoverable Error。

---

## 13. Store / Account Error

可能包括：

- Store Suspended；
- API Permission；
- Authentication；
- Account Restriction；
- 店铺暂时不可销售。

特点：

> Product 本身可能完全没有问题。

因此 Store Error 不应自动改变 Product Risk State，也不应默认永久删除 Listing。

---

## 14. Platform System Error

可能包括：

- Internal Error；
- Timeout；
- Temporary Processing Failure；
- Platform Service Unavailable；
- Platform Bug。

原则：

> Platform System Error 不等于 Product Error。

通常应 Wait / Retry / Query Existing Submission / Verify Result。

尤其要避免：请求超时后直接重复提交，而实际第一次请求已经被平台接受。

---

## 15. Time / Date / Expiration Error

当前已经存在明确业务处理的一类。

例如平台提示日期过期：

```text
Expiration Error
→ 修复日期
→ 重新提交
```

这是当前已明确可由系统自动处理的错误类型之一。

---

## 16. Unknown / Unclassified

无法可靠归类时，允许进入 Unknown。

必须保留：

- 原始 Error；
- Listing / Product 上下文；
- Platform State；
- 出现次数；
- 相关 Evidence。

原则：

> 系统没有对应规则时必须承认 Unknown，不能静默猜测并执行破坏性操作。

“最终理论上都能自动判断”不等于“当前系统已经知道如何判断”。

---

## 17. 当前团队默认处理策略

| Error Category | 当前 Team 默认处理 |
|---|---|
| Time / Expiration | 自动修复并重新提交 |
| Content | 暂无完整自动修复逻辑 |
| Category / Spec | 暂无完整自动修复逻辑 |
| GTIN / UPC | 根据具体错误继续分类 |
| Brand | 放弃当前 Listing |
| Compliance | 放弃当前 Listing |
| Inventory / Price | 日常维护逻辑处理 |
| Store / Account | 等待或修复 Store 状态 |
| Platform System | 查询、等待、重试 |
| Unknown | 不自动执行破坏性动作 |

这只是当前团队模板，不是 ERP 强制规则。

---

## 18. 原则上所有错误最终都可以自动决策

当前业务观点：

> 不存在某一类错误因为“本质上只能人判断”而永久不能自动化。

如果系统能够：

1. 正确识别错误；
2. 正确理解上下文；
3. 拥有明确处理规则；
4. Team 已授权自动处理；

则理论上都可以由系统决定下一步。

人工介入通常只是：

- 分类规则尚未完善；
- 数据不足；
- 系统置信度不足；
- Team 尚未授权自动处理；
- 当前属于 Unknown。

---

## 19. 自动处理能力与 Error Type 解耦

例如 Content Error 当前没有自动修复，并不意味着：

`Content Error = Manual`

正确关系：

```text
Error Type = Content
Current Handler Capability = None / Limited
```

未来增加 AI Content Fixer 后：

- Error Type 不变；
- Handler Capability 变强。

---

## 20. Team Resolution Policy

Team 可以为错误定义不同策略。

例如：

`CONTENT.TITLE_TOO_LONG`

- Team A：AI Fix → Validate → Auto Resubmit；
- Team B：AI Suggest → Pause；
- Team C：Exit Listing。

策略未来可以按层级匹配：

1. Category；
2. Subtype；
3. Platform Error Code；
4. Listing / Store Override。

阶段 0 只确认这种业务能力需要存在，不决定技术结构。

---

## 21. Error 本身也有生命周期

```text
Detected
→ Classified
→ Action Planned
→ Handling
→ Resolved / Failed to Resolve / Replaced / Waiting / Closed by Exit
```

同一个 Listing 可以同时存在多个 Active Errors，例如：

- Image Error；
- Missing Attribute；
- Price Error。

系统需要知道哪个错误是阻断型、某次修复解决了哪些错误、是否产生了新错误。

---

## 22. Raw Error、Normalized Error、Root Cause、Resolution 分离

平台返回的 Error Message 只是 Platform Error Signal。

例如 `Title invalid` 进一步可能是：

- 字符超长；
- 禁止词；
- 品牌词；
- HTML；
- 格式问题。

因此概念上应区分：

```text
Raw Platform Error
→ Normalized Error
→ Root Cause
→ Resolution
```

---

## 23. Initial Failure 与 Post-Publish Error 的证据不同

即使 Error Category 相同：

### Initial Failure

目标是让首次上架成功。

### Post-Publish Unpublished

目标是恢复历史上已经正常经营的 Listing。

后者还可以参考：

- 上一次 Published Data；
- 最近成功资料；
- 最近修改内容；
- 状态变化时间；
- 哪次修改后变成 Unpublished。

这些是 Root Cause 判断的重要证据。

---

## 24. 自动修复的业务结构

未来不能简单设计成：

`发现报错 → 调 AI → 重新提交`

而应该：

```text
Detect Error
→ Classify
→ Select Handler
→ Generate Repair Plan
→ Validate Plan
→ Apply Changes
→ Submit
→ Wait Platform Result
→ Verify
→ Resolved / Re-classify
```

### Repair Plan

即使未来全自动，也应概念上存在 Repair Plan，记录：

- Current Value；
- Proposed Value；
- Changed Fields；
- Resolution Type；
- Expected Result。

这样才能追踪每次修复到底改了什么。

---

## 25. 与 Workflow 的关系

Error Management 提供基础能力，例如：

- Detect；
- Classify；
- Normalize；
- Resolve；
- Retry；
- Verify。

Workflow 决定何时、对谁、按什么顺序调用这些能力。

例如当前团队未来可以有：

```text
Daily Listing Error Workflow
→ 获取 Unpublished Listings
→ Classify Errors
→ 日期错误：Auto Fix
→ Brand / Compliance：Exit
→ 其他：按 Team Policy 处理
```

其他 Team 可以创建完全不同的 Workflow。

---

## 26. 当前已确认原则

1. 首次上架 Failed 与 Published 后 Unpublished 必须区分；
2. 报错判断结合 `unpublishedReasons` 与商品详情；
3. Raw Error、Normalized Error、Root Cause、Resolution 应分离；
4. Error Classification 属于平台事实；
5. Resolution Policy 属于 Team Policy；
6. 同一 Error 不同 Team 可采用不同处理；
7. Brand / Compliance 当前团队默认退出 Listing；
8. Expiration 已具备明确自动修复路径；
9. Content / Category 当前未自动修复不代表永久需要人工；
10. 原则上分类与规则足够明确后，错误都可以自动决策；
11. Unknown 必须允许存在；
12. 未知错误不能自动执行破坏性动作；
13. Store Error 不等于 Product Error；
14. Platform System Error 不等于 Product Error；
15. 一个 Listing 可以同时存在多个 Active Errors；
16. Error 自身具有生命周期；
17. 每次修复需要追踪修改内容与结果；
18. Error Handler 与 Error Classification 必须解耦；
19. Team Workflow 可以组合不同 Error Handler；
20. Error Management 是 ERP 基础能力，自动错误处理属于后续 Workflow。
