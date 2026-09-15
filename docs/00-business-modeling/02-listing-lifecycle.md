# Listing 生命周期业务模型 v0.1

> 状态：已确认业务基线

## 1. Listing 与 Product 是两个不同对象

- Product 代表团队经营的商品资产；
- Listing 代表某个 Product 在某个销售平台、某个 Store 中的销售实例；
- Product 可以没有 Listing，也可以同时存在多个 Platform / Store Listing；
- Listing 被删除、Retire 或 Unpublished，并不等于 Product 本身退出 ERP。

---

## 2. Listing Draft

只要已经确定 Target Platform，就可以创建 Listing Draft。

此时可以尚未：

- 确定 Store；
- 确定 Warehouse；
- 生成 SKU；
- 准备 UPC / GTIN；
- 设置价格库存；
- 准备完全部资料。

Listing Draft 用于承载某个 Product 面向某个平台准备的销售资料。

后续可以不断被人工编辑、AI 补全、重新校验，并最终绑定到任意符合要求的 Store。

---

## 3. 正式 Listing

当：

- Platform 已确定；
- Store 已确定；
- 上架资料已经达到可提交状态；

即可形成正式 Listing。

之后随时可以继续补充：

- SKU；
- UPC / GTIN；
- Price；
- Inventory；
- Warehouse；
- 其他平台要求字段。

必要字段准备完成后进入 Waiting to Submit。

---

## 4. Listing 的业务身份

当前业务中，一条 Listing 对应一个 Store + 当前 SKU。

但：

- 更换 SKU 仍可视为同一条 ERP Listing；
- 更换 UPC / GTIN 仍可视为同一条 ERP Listing；
- SKU / UPC / GTIN 不是 ERP Listing 永久主身份；
- ERP 需要自己的内部 Listing ID；
- 可以保留 Current SKU 与 Historical SKU 等历史标识。

平台成功上架后由平台返回的 Product ID / Item ID / WPID 等属于 Platform Assigned Identifier，不是 ERP Listing ID。

---

## 5. Listing 至少有三套状态

不能设计成一个统一 `status`。

### ERP Business State

表示团队对该 Listing 当前的经营意图，例如：

- Draft；
- Ready；
- Operating；
- Paused；
- Exited。

### Submission State

表示最近一次平台操作的处理状态，例如：

```text
Waiting to Submit
→ Submitted
→ Processing
→ Succeeded / Failed
```

### Platform State

表示平台当前真实反馈，例如：

- Published；
- Unpublished；
- Retired；
- Deleted；
- 不存在；
- 其他平台状态。

这三套状态必须分开。

---

## 6. 运营上必须区分的五个阶段

1. 已准备但未提交；
2. 已提交但平台尚未给最终结果；
3. 上架成功；
4. 首次上架失败；
5. 已经 Published 后又被平台取消发布（Unpublished）。

Submitted 不等于 Published。

批量 Feed 完成也不代表其中每个商品都成功，最终要追踪 Listing / Item 级结果。

---

## 7. Listing 主流程

```text
Listing Draft
→ 资料准备
→ Ready / Waiting to Submit
→ Submitted
→ Processing
→ Published / Failed
```

如果首次上架失败：

```text
Failed
→ Platform Error Management
→ Fix / Retry / Exit
```

如果已经 Published：

```text
Published
→ Online Operation
→ 可能出现 Unpublished
→ 原因分析
→ Fix / Wait / Pause / Retire / Delete / Exit
```

---

## 8. Unpublished

Unpublished 是平台状态，不是 ERP Product 退出状态。

当前业务中需要区分：

1. 可修复 → 修复后恢复；
2. 店铺问题等暂时问题 → Product 本身仍可用，等待恢复；
3. 商品本身有问题 → 下架；
4. 品牌 / TRO / 平台政策风险 → 永久退出。

因此 Unpublished 只是问题触发点，必须先判断原因再决定动作。

---

## 9. Inventory = 0

库存归零只是平台销售控制动作，不表示：

- Listing 被删除；
- Listing 被 Retire；
- Product 退出；
- ERP 经营关系消失。

### 规则驱动库存归零

例如：

- 来源缺货；
- 配送方式改变；
- New 变 Used；
- 其他规则。

此时其他正常维护逻辑仍可以继续。

### 人工库存归零

表示人工判断某 Store 暂时不销售该商品，但 Listing 关系仍在。

人工库存归零不等于暂停所有自动化。

因此 Platform Inventory = 0 与 ERP Paused 必须区分。

---

## 10. Retire

Retire 是平台动作。

业务意图是主动停止当前 Listing，未来仍可能重新建立销售关系。

ERP 通用模型不能把 Retire 固定定义成“可恢复暂停”，具体平台的真实技术语义由 Platform Capability Specification 定义。

ERP 只记录：

- 团队发起了 Retire；
- 平台处理结果；
- 后续是否重新建立 Listing。

---

## 11. Delete

Delete 表示希望从该 Store 删除该平台 Listing / SKU 关系。

Delete：

- 不删除 ERP Product；
- 不代表以后永远不能在该 Store 再销售。

未来重新上架：

### 同 Store

可以使用原 SKU，也可以生成新 SKU 再提交。

### 不同 Store

创建新的 Store Listing，并创建新的 SKU。

ERP 必须保留旧 Listing 历史。

---

## 12. Re-list

重新上架的前置通常是原商品已经从 Store 删除或退出到需要重新创建销售关系的程度。

阶段 0 只确认：

> 历史必须连续可追踪。

是否技术上把 Re-list 建模为新 Listing Instance、同 Listing 新 Attempt 或其他结构，留到阶段 1 技术设计决定。

---

## 13. 上线后的修改

业务原则：只要目标平台允许修改，ERP 就应该可以提供对应修改能力。

可能包括：

- Price；
- Inventory；
- Title；
- Brand；
- Description；
- Images；
- Attributes；
- Warehouse；
- SKU；
- UPC；
- GTIN；
- 其他平台属性。

具体某字段：

- 是否可修改；
- 走什么 API / Feed；
- 是否异步；
- 是否要求重新审核；
- 是否影响平台 Catalog Identity；

必须由各 Platform Adapter 的 Capability Matrix 定义，不能写进通用 Listing 模型。

---

## 14. Listing 操作风险

当前业务粗分：

### 普通维护类

- Update Price；
- Update Inventory。

### 高风险修改类

除价格和库存以外的 Listing 修改当前统一按较高风险处理，例如：

- Title / Brand / Description / Images；
- Attributes / Product Type；
- SKU / UPC / GTIN；
- Warehouse / Fulfillment 结构；
- Retire / Delete / Re-list；
- 其他 Catalog 结构修改。

高风险不等于一定需要人工审批。未来经过团队明确授权的 Workflow 仍可以自动执行。

---

## 15. Listing 生命周期完整模型

```text
Product
→ Target Platform Selected
→ Listing Draft
→ Store Selected / Assigned
→ Listing Created
→ 补齐 SKU / UPC / GTIN / Price / Inventory / Warehouse / Final Listing Values
→ Ready
→ Waiting to Submit
→ Submitted
→ Processing
→ Failed / Published
```

失败：

```text
Failed
→ Platform Error Management
→ Fix / Retry / Exit
```

成功：

```text
Published
→ Online Operation
→ 价格 / 库存 / 内容 / 来源 / 仓库等持续维护
→ 可能 Unpublished
→ 原因判断
→ Fix / Wait / Resume / Zero Inventory / Retire / Delete / Permanent Exit
```

Delete / Exit 后未来仍可能 Re-list。

---

## 16. 当前已确认原则

1. Platform 一旦确定即可创建 Listing Draft；
2. Draft 不要求立即绑定 Store；
3. 正式 Listing 最终属于具体 Platform + Store；
4. SKU / UPC / GTIN 不是 ERP Listing 永久 ID；
5. ERP Business State、Submission State、Platform State 必须分开；
6. Submitted 不等于 Published；
7. Feed Success 不等于全部 Item 成功；
8. 首次上架失败和 Published 后 Unpublished 必须区分；
9. Unpublished 不等于 ERP Product 退出；
10. Inventory = 0 不等于 Pause / Retire / Delete；
11. Manual Zero Inventory 不自动停止其他维护规则；
12. Retire / Delete 是平台动作，不直接决定 ERP Product 生命周期；
13. Delete 后允许未来 Re-list；
14. 同 Store Re-list 可用旧 SKU 或新 SKU，不同 Store 创建新 SKU；
15. 更换 SKU / UPC / GTIN 仍可保持同一 ERP Listing 业务连续性；
16. 平台字段的具体可修改性由 Platform Capability Matrix 定义；
17. Price / Inventory 属日常普通维护，其他 Listing 修改当前按较高风险处理；
18. 高风险不等于必须人工审批。
