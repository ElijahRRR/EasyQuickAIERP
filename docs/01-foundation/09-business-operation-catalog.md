# 正式业务动作清单 v0.1

> 状态：进行中；第一批已确认  
> 所属阶段：阶段 1 — ERP 地基  
> 目标：用业务语言明确 EasyQuickAIERP 允许执行哪些正式业务动作。人工操作、Workflow、AI、Open API、定时任务最终都应复用这些动作，而不是各自实现一套业务逻辑。

## 1. 这份清单解决什么问题

这份文档不定义：

- 数据库表；
- API 地址；
- HTTP 方法；
- 队列；
- 重试次数；
- 前端按钮样式。

它只回答：

> ERP 在业务上“允许做什么”。

例如：

~~~text
人工改价
Workflow 自动改价
AI 执行改价
Open API 改价

最终都应进入同一个正式“修改售价”业务动作。
~~~

---

## 2. 店铺相关业务动作

### 2.1 创建店铺

在 ERP 内创建一个新的店铺记录。

创建店铺不等于：

- API 已连接成功；
- Walmart 店铺状态为正常；
- 已经开始同步业务数据。

---

### 2.2 绑定或更换 API 凭证

给店铺绑定当前使用的 API 凭证，或更换已经失效 / 更新后的凭证。

正式原则：

> API 凭证不是店铺身份。

更换凭证不会创建一个新店铺。

---

### 2.3 测试店铺连接

检查 ERP 当前是否能使用现有凭证和连接配置访问平台。

测试成功只说明：

> 当前连接正常。

不能因此把平台店铺状态自动改成：

- ACTIVE；
- SUSPENDED；
- TERMINATED；
- 其他平台业务状态。

---

### 2.4 启用 / 停用 ERP 店铺

表示 Team 是否允许 ERP 对这个店铺继续进行正常业务处理。

它只改变 ERP 自己的启用状态，不改变 Walmart 平台真实状态。

---

### 2.5 解绑店铺

结束当前 Team 对该外部店铺的 ERP 接入关系。

解绑以后：

- 停止当前正常接入；
- 凭证可以移除 / 停用；
- 历史订单、Listing、采购、售后、结算、利润等继续保留。

---

### 2.6 同步 / 确认平台店铺状态

店铺在 Walmart 的真实平台状态不能由普通用户随意修改。

正式确认：

> 普通用户不能直接把店铺从“正常”改成“暂停”或“终止”。

平台状态只能来自：

- Walmart API；
- Seller Center 实际观察；
- 其他明确可靠的平台证据；
- 有证据支持的人工确认。

如果连接失败，只能更新“连接是否正常”，不能猜测店铺已经被暂停或终止。

---

### 2.7 修改店铺经营设置

店铺经营设置发生变化时，必须留下历史。

例如：

- 价格范围；
- 某个物流节点最大库存；
- 采购成本系数；
- 经营类目；
- Workflow；
- Risk Policy；
- 其他 Store Policy。

正式确认：

> 不能只保留最新设置。

系统需要能够回答：

> 某个历史时间段，当时这个店铺使用的经营设置是什么。

---

### 2.8 修改店铺负责人 / 运营组

负责人或运营组变更时，需要保存历史生效区间。

例如：

~~~text
1–6 月：张三 / A 组
7 月以后：李四 / B 组
~~~

历史订单仍按订单发生时的负责人 / 运营组归属。

---

### 2.9 释放店铺的品牌 / 产品占用

当店铺被平台明确确认 TERMINATED，并且 Team 使用品牌 / 产品占用机制时：

> 可以自动释放该店铺占用的品牌和产品。

这个动作必须能说明：

- 为什么释放；
- 哪个店铺触发；
- 释放了哪些对象；
- 什么时候发生。

SUSPENDED 不自动释放。

---

## 3. 商品相关业务动作

### 3.1 创建商品

在 Team 商品池中建立一个新的商品主档。

商品可以来自：

- 人工创建；
- 采集；
- CSV / Excel 导入；
- 外部来源导入；
- 后续其他入口。

---

### 3.2 导入商品

批量把外部商品资料导入当前 Team 商品池。

导入只是：

> 商品进入 ERP 商品体系。

不代表：

- 已审核通过；
- 已确定销售平台；
- 已分配店铺；
- 已经上架。

---

### 3.3 修改商品资料

修改 Team 对该商品当前保存的商品资料。

如果修改内容会影响后续商品审核，应按已确认规则使旧审核结果可能变为需要重新审核，而不是直接改写旧审核历史。

---

### 3.4 给商品增加一个来源

例如：

~~~text
Product P001
+ Amazon ASIN A
~~~

表示：

> 这个 ASIN 被人工确认可以作为该 Product 的一个来源。

正式确认：

> “增加来源”不等于“设为首选来源”。

---

### 3.5 解除商品与来源的当前关联

表示：

> 以后不再把这个来源作为当前可用来源关系。

但不能删除：

- 过去的绑定历史；
- 历史采购；
- 历史订单审核证据；
- 过去发生的真实业务记录。

---

### 3.6 设置首选来源

从已经关联的来源中选择：

> 正常情况下优先考虑的来源。

正式确认：

~~~text
增加来源
≠
设置首选来源
~~~

---

### 3.7 设置备用来源

从已经关联的来源中选择一个或多个：

> 首选来源不可用时可以考虑的备用来源。

正式确认：

~~~text
设置首选来源
≠
设置备用来源
~~~

因此当前正式区分：

~~~text
增加来源
≠
设置首选来源
≠
设置备用来源
~~~

---

### 3.8 观察来源当前采购条件

例如同一个 Amazon ASIN：

~~~text
今天：
Seller A
Price = 20
Stock = 15
FBA

明天：
Seller B
Price = 22
Stock = 8
FBA
~~~

正式确认：

> 这仍然是同一个 ASIN 商品来源。

Seller、Price、Stock、Fulfillment、Delivery 等变化：

- 不创建新的 Product Source；
- 只形成新的来源条件观察记录；
- 历史观察记录按数据保留策略管理。

因此必须保持：

~~~text
商品来源身份
≠
某个时间点的卖家 / 价格 / 库存 / 配送条件
~~~

---

### 3.9 执行商品审核

对某个 Product 面向某个目标平台进行审核。

每次审核都新增历史记录，不能用最新审核结果覆盖过去。

正式结果仍然是：

- Pass；
- Reject；
- Pending。

每次审核要能够还原当时：

- 审核的商品内容版本；
- 目标平台；
- 证据；
- 审核规则版本；
- 结果；
- 时间；
- 执行者 / 触发来源。

---

## 4. 第一批已确认原则

1. 店铺真实平台状态不能由普通用户随意修改。
2. 连接成功 / 失败与平台店铺状态分开。
3. 店铺经营设置修改必须保留历史。
4. 店铺负责人 / 运营组修改必须保留历史。
5. 增加商品来源、设置首选来源、设置备用来源是三个不同业务动作。
6. 解除当前商品来源关系不能删除历史采购和历史证据。
7. 同一个 ASIN 的 Seller、Price、Stock、Fulfillment、Delivery 变化不会创建新的商品来源。
8. 来源采购条件变化形成新的观察记录，而不是修改来源身份。
9. 商品审核每次形成独立历史记录。

---

## 5. Listing 相关业务动作

### 5.1 创建 Listing 草稿

当 Product 已确定目标销售平台后，可以创建 Listing 草稿。

草稿阶段：

- Store 可以暂时未确定；
- SKU / UPC / GTIN 可以尚未准备完成；
- Price / Inventory 可以尚未确定；
- Platform Resource 可以尚未选择；
- Listing 资料可以继续编辑。

草稿后续进入正式 Listing 时：

> 保持同一个 ERP Listing Identity。

---

### 5.2 选择 / 更换 Listing 所属店铺

Listing 真正提交平台前必须确定 Store。

在 Draft 阶段选择 Store：

> 不创建新的 Listing。

如果平台业务允许后续调整相关 Store 归属，必须按照平台和业务规则处理；不能简单通过改一个字段让历史归属失真。

---

### 5.3 保存 Listing 资料

运营、AI 或其他业务能力可以修改并保存 ERP 中准备的 Listing 资料，例如：

- Title；
- Brand；
- Description；
- Images；
- Product Type；
- Attributes；
- SKU；
- UPC / GTIN；
- 其他平台字段。

正式确认：

> **保存 ERP 资料 ≠ 立即提交 Walmart。**

保存后可以继续：

- 人工修改；
- AI 优化；
- 资料检查；
- 批量处理；
- 等待最终确认。

不能因为用户点击“保存”，就默认立刻修改线上 Walmart Listing。

---

### 5.4 检查 Listing 资料

在正式提交前，ERP 应能够检查当前准备的资料是否满足目标平台要求。

检查结果可以是：

- 可提交；
- 缺少资料；
- 存在可修复问题；
- 存在风险 / 合规问题；
- 需要人工处理。

对 Walmart：

> 检查必须基于当时正式使用的 Walmart Spec Version。

资料检查本身不等于已经提交平台。

---

### 5.5 创建全新的 Walmart Listing

表示明确选择：

> 在 Walmart 创建一个新的 Listing / 商品销售关系。

它和“通过 UPC / GTIN 匹配 Walmart 已有商品”是两种不同业务意图。

如果创建失败：

> 不允许系统静默改成“跟卖”再尝试。

---

### 5.6 通过 UPC / GTIN 匹配 Walmart 已有 Listing

即当前业务中的“跟卖”。

表示：

> 通过 UPC / GTIN 等标识匹配 Walmart 已有 Catalog 商品，并为当前 Store 建立销售关系。

如果跟卖失败：

> 不允许系统静默改成“创建全新 Listing”。

系统可以帮助搜索、推荐和提示匹配结果，但最终上架方式必须明确。

因此：

~~~text
创建全新 Listing
≠
UPC / GTIN 跟卖
~~~

---

### 5.7 修复资料后重新提交

如果一次 Listing 提交失败，例如：

~~~text
第一次提交
→ Walmart 返回缺少 Material
~~~

运营修复资料后：

~~~text
形成新的 Listing 资料版本
→ 再次提交
~~~

正式确认：

> 旧的失败记录不能被新的提交结果覆盖。

历史应能看到：

~~~text
Submission 1
→ Failed
→ 原因

Submission 2
→ Success / Failed / Pending
~~~

---

### 5.8 修改已经在线的 Listing 内容

已经在线的 Listing，如果目标平台允许，可以修改：

- Title；
- Brand；
- Description；
- Images；
- Attributes；
- Product Type；
- SKU / UPC / GTIN；
- 其他平台允许修改的内容。

业务上仍然遵守：

~~~text
先修改 / 保存 ERP 准备资料
↓
检查
↓
明确提交平台
~~~

保存资料和提交平台是两个动作。

---

### 5.9 修改售价

ERP 可以修改单个或批量 Listing 的目标售价。

必须区分：

~~~text
ERP 目标售价
≠
Walmart 当前真实售价
~~~

提交修改以后，只有真实查询 Walmart 后看到平台价格达到目标，才说明平台实际生效。

具体多久查询、如何重试由开发设计。

---

### 5.10 修改库存

ERP 可以：

- 修改默认 Walmart 节点库存；
- 修改指定节点库存；
- 一次修改多个节点；
- 批量修改多个 Listing 的节点库存。

必须区分：

- 来源库存；
- 某节点可选最大库存设置；
- ERP 目标节点库存；
- Walmart 实际节点库存。

某个节点失败不应覆盖其他已经成功节点的结果。

---

### 5.11 修改 Listing 使用的平台资源

例如：

- Fulfillment Center / Ship Node；
- Shipping Template；
- Shipping Override；
- 其他平台允许的资源关联。

如果用户明确选择了某个资源，但该资源已经无效：

> 不能偷偷改用其他资源或 Default。

只有明确选择“使用平台默认配置”时，才允许平台采用 Default。

---

### 5.12 库存设为 0

库存设为 0 是库存操作。

它不等于：

- Listing 停用；
- Listing 删除；
- Product 退出；
- ERP 历史关系删除。

库存恢复以后 Listing 可以继续销售，只要平台状态允许。

---

### 5.13 停用 Listing

“停用”表示：

> 主动停止当前 Listing 的销售，但 Listing 仍保留在 Walmart 后台。

停用成功后，ERP 仍保留同一 Listing 的历史。

---

### 5.14 恢复已停用 Listing 的销售

对于“停用”的 Listing：

> 恢复动作统一叫“恢复销售”。

恢复销售与重新创建新的 Listing 不是同一件事。

业务历史应该能看到：

~~~text
Online
→ 停用
→ 恢复销售
~~~

保持连续。

---

### 5.15 删除 Listing

“删除”与“停用”是两个不同动作。

删除表示：

> 向平台请求删除当前 Listing / SKU 销售关系。

Delete 属于高风险业务操作。

删除平台 Listing：

- 不删除 ERP Product；
- 不删除历史订单；
- 不删除历史 Listing 记录；
- 不删除过去的 Submission / Error / Profit 等历史。

---

### 5.16 删除后重新上架

Listing 已删除以后，如果以后重新销售：

> 业务上统一叫“重新上架”。

不能把它叫成“恢复销售”。

因此：

~~~text
停用
→ 恢复销售

删除
→ 重新上架
~~~

两条业务语义必须分开。

重新上架时历史必须能够追踪到原来曾经删除的 Listing。

---

### 5.17 同步 Walmart 当前 Listing 状态与资料

ERP 需要能够主动从 Walmart 读取当前真实状态，例如：

- Published / Unpublished / Retired / Deleted；
- 当前 Title / Brand / Description / Images / Attributes；
- 当前 Price；
- 当前各 Ship Node Inventory；
- 当前 Platform Resource Association；
- 其他可读取的平台事实。

这些读取结果表示：

> **Walmart 当前真实观察值。**

普通用户不能直接把这些平台事实随意改成另一个值。

---

### 5.18 ERP 准备资料与 Walmart 实际资料分开

正式确认：

~~~text
ERP 当前准备 / 希望提交的 Listing 资料
≠
Walmart 当前真实读取到的 Listing 资料
~~~

例如：

~~~text
ERP 目标标题：
Rechargeable Wireless Desk Lamp

Walmart 当前标题：
Wireless Desk Lamp
~~~

ERP 不能因为同步到了 Walmart 当前标题，就无痕覆盖运营正在准备的目标版本。

系统需要同时能够回答：

- ERP 当前准备提交什么；
- Walmart 当前实际是什么；
- 两者是否一致；
- 最近一次提交结果是什么。

---

### 5.19 查看并处理 Listing 错误

Listing 可能出现：

- 首次上架失败；
- 修改资料失败；
- Price / Inventory 未达到目标；
- Published 后 Unpublished；
- Platform Resource 无效；
- Policy / Compliance / IP 风险；
- Unknown Error。

ERP 应允许：

- 查看 Walmart 原始错误；
- 查看系统归类；
- 修复资料；
- 重新检查；
- 再次提交；
- 转人工；
- 进入 Risk / Compliance；
- 停用 / 删除等后续处理。

具体 HTTP Retry / Backoff 等技术机制由开发负责，不属于业务动作定义。

---

## 6. Listing 已确认原则

1. 保存 Listing 资料和提交 Walmart 是两个不同动作。
2. 创建全新 Listing 与 UPC / GTIN 跟卖必须明确区分，失败后不能静默互换。
3. 每次正式提交都保留独立历史；修复后重新提交不能覆盖旧失败记录。
4. Listing 资料的重要修改形成新的内容版本，并可还原每次提交使用的是哪一版。
5. 停用后叫“恢复销售”；删除后叫“重新上架”，两者不能混用。
6. 删除 Walmart Listing 不删除 ERP Product 和历史业务记录。
7. ERP 当前准备 / 希望提交的 Listing 资料，与 Walmart 当前真实读取到的资料分开。
8. Walmart 同步结果不能无痕覆盖 ERP 正在准备的目标资料。
9. Price / Inventory 的目标值与平台真实值分开。
10. Explicit Platform Resource 失效时不能静默退回 Default。
11. Listing 状态、提交结果、平台真实状态继续保持分开。
12. Listing 错误处理保留 Walmart 原始错误与后续处理历史。

---

## 7. 订单与采购相关业务动作

### 7.1 同步 Walmart 订单

ERP 从 Walmart 读取并保存真实订单资料，例如：

- Walmart Order ID；
- Order Line；
- SKU；
- 数量；
- 销售金额；
- 收货信息；
- 平台订单状态；
- 平台取消 / 发货 / 退款等事实。

正式确认：

> **这些来自 Walmart 的订单事实不能由普通运营人员直接修改。**

例如不能人工把：

- Quantity 2 改成 1；
- Sales Amount 39.99 改成 29.99；
- Walmart Shipped 改成 Cancelled。

如果 Walmart 后续状态发生变化：

> 由再次同步 Walmart 的结果更新平台事实。

ERP 内部可以另行保存：

- 运营备注；
- 审核结果；
- 负责人；
- 采购处理意见；
- 其他内部业务信息。

内部信息不能冒充 Walmart 返回的真实订单事实。

---

### 7.2 查看订单详情

人工可以查看订单和每条订单商品的：

- Walmart 原始订单信息；
- 当前平台状态；
- ERP 内部审核结果；
- 已有关联采购；
- 已有发货；
- 售后；
- 对账 / 利润等信息。

“查看”本身不修改 Walmart 订单事实。

---

### 7.3 Acknowledge 订单

Acknowledge 是独立的 Walmart 操作。

它不等于：

- 同步订单；
- 订单审核通过；
- 已采购；
- 已发货。

当前 Team 由人工在处理订单时执行 Acknowledge。

未来如果某个 Team 自动执行，由该 Team 自己决定触发条件和流程顺序。

---

### 7.4 执行订单审核

订单审核用于判断：

> 当前这条订单商品是否具备继续履约条件。

结果：

- Pass；
- Reject；
- Pending。

每次重新审核都形成新的历史记录，不能覆盖之前为什么 Pass / Reject / Pending。

---

### 7.5 取消 Walmart 订单商品

取消主要按 Order Line 处理。

必须保持：

> ERP 想取消 ≠ Walmart 已取消。

只有 Walmart 实际返回 / 回读确认以后，平台订单事实才可以记录为已取消。

---

### 7.6 创建采购任务

采购任务表示：

> 某条订单商品当前需要采购一定数量的商品来完成履约。

采购任务可以由：

- 订单审核后的 Workflow 自动创建；
- 系统规则创建；
- 有权限的人工创建。

人工创建采购任务时，至少需要明确：

- 对应哪条订单商品；
- 为什么需要采购；
- 需要采购多少。

因此丢件重购、售后补发、特殊人工处理等场景都可以使用同一采购任务能力。

---

### 7.7 分配采购任务

运营或有权限人员可以把采购任务分配给：

- 某个采购组；
- 某个采购人员。

“分配”表示：

> 谁被指定负责处理这个采购需求。

---

### 7.8 领取采购任务

采购人员可以领取被分配或可领取的采购任务。

正式确认：

> **分配任务 ≠ 采购人员领取任务。**

系统需要能够知道：

- 谁进行了分配；
- 分给了谁 / 哪个组；
- 最终谁实际领取并处理。

---

### 7.9 人工登记一次真实采购

EasyQuickAIERP V1 必须支持采购人员手工登记一次已经真实发生的采购。

例如在 Amazon 下单后，登记：

- 实际 ASIN / Variant；
- Seller；
- Fulfillment；
- Quantity；
- 商品金额；
- Tax；
- Shipping Cost；
- Actual Paid；
- Amazon / Source Order ID；
- Purchase Time；
- Promise / Estimated Delivery；
- 其他影响采购事实的重要信息。

正式确认：

> **一次真实下单行为形成一条独立 Purchase 记录。**

例如客户需要 3 件：

~~~text
Purchase A = 1 件
Purchase B = 2 件
~~~

而不是不断覆盖一条 Purchase。

因此：

~~~text
一条 Order Line
→ 可以有一个或多个 Procurement Task
→ 一个 Procurement Task 可以对应一个或多个 Purchase
~~~

以后如果接入自动采购，也必须形成相同的正式 Purchase 事实，不能建立另一套采购数据体系。

---

### 7.10 退回无法处理的采购任务

如果采购人员发现：

- 主来源无货；
- 备用来源也不可用；
- 价格过高；
- 配送速度不满足；
- 商品无法确认一致；
- 其他原因无法继续采购；

可以：

> **把采购任务退回运营处理，并填写原因。**

运营随后再决定：

- 等待；
- 寻找其他来源；
- 创建新的采购处理；
- 取消 Walmart 订单商品；
- 其他业务动作。

正式确认：

> **无法采购 ≠ 自动取消 Walmart 订单。**

采购模块不能因为“采购失败 / 无法采购”直接替运营执行 Walmart Cancel。

---

## 8. 订单与采购第一批已确认原则

1. Walmart 同步进来的 Order / Order Line 平台事实不能由普通运营直接修改。
2. ERP 内部备注、审核结果、采购意见等与 Walmart 平台订单事实分开。
3. Acknowledge 与订单同步、审核、采购、发货分开。
4. 订单审核每次形成独立历史判断。
5. ERP 取消决定与 Walmart 实际取消状态分开。
6. Procurement Task 可以由 Workflow / System 创建，也可以由有权限的人工创建。
7. 人工创建 Procurement Task 必须明确对应订单商品、原因和需求数量。
8. 采购任务的“分配”和“领取”是两个不同动作。
9. V1 必须支持人工登记真实 Purchase。
10. 一次真实下单行为形成一条独立 Purchase；一个采购需求可以由多个 Purchase 覆盖。
11. 自动采购未来也必须写入同一 Purchase 业务体系。
12. 采购任务无法处理时可以退回运营，并保留原因。
13. 无法采购不自动取消 Walmart 销售订单。

---

## 9. 采购下单后的处理与 Walmart 发货

### 9.1 修正已经登记的采购资料

采购人员已经登记真实 Purchase 后，如果发现录入错误，例如：

- Amazon / Source Order ID 填错；
- Tax 填错；
- 商品金额填错；
- Shipping Cost 填错；
- 实际支付金额填错；
- Seller / Variant 等采购事实录入错误；

允许有权限的人进行修正。

但正式确认：

> **采购事实可以纠错，但不能无痕覆盖。**

系统需要保留：

- 修改前内容；
- 修改后内容；
- 修改人；
- 修改时间；
- 修改原因（适用时）。

---

### 9.2 来源采购被取消后重新计算未覆盖数量

例如某条订单商品需要采购 3 件：

~~~text
Purchase A = 2 件，正常
Purchase B = 1 件，Amazon / Seller Cancelled
~~~

则被取消的 1 件：

> 重新成为当前仍未被有效采购覆盖的数量。

如果只是部分取消：

> 只重新释放被取消的数量。

系统需要让采购 / 运营能够看到：

> 当前还缺多少件需要解决。

但：

> **来源采购被取消，不代表 ERP 自动再下一笔采购订单。**

是否重新采购，由人工或 Team 后续配置的 Workflow 决定。

---

### 9.3 来源退款与采购履约状态分开

Amazon / 其他来源发生 Refund 时，首先记录：

> 采购侧资金变化。

不能仅根据“发生退款”推断：

- Purchase 已取消；
- 商品没有发货；
- 商品没有送达；
- 原采购数量一定失效。

例如可能存在：

- 取消后全额退款；
- 送达后退货退款；
- Partial Refund；
- 运费退款；
- 价格补偿；
- 其他 Credit / Adjustment。

因此正式保持：

~~~text
Purchase Refund
≠
Purchase Cancelled
≠
商品一定未履约
~~~

只有存在明确取消、退货、未发货或其他履约证据时，才改变相应采购履约判断。

---

### 9.4 原采购失败后的重新采购

如果原采购任务的需求没有改变，只是第一次真实采购失败：

~~~text
Procurement Task PT001
├ Purchase A → Seller Cancelled
└ Purchase B → Re-purchase
~~~

第二次重新购买：

> 仍然属于原来的 Procurement Task。

不因为每次重新采购都新建一个采购任务。

这样一个采购任务可以完整表达：

> 为满足同一个采购需求，实际尝试过哪些 Purchase。

只有出现新的采购需求，例如：

- 售后补发；
- 新增补购需求；
- 其他独立履约需求；

才创建新的 Procurement Task。

所有历史 Purchase 继续保留。

---

### 9.5 人工向 Walmart 提交发货信息

EasyQuickAIERP V1 必须支持有权限的人员人工通过 ERP 向 Walmart 提交发货信息。

人工发货时可以填写平台实际要求的信息，例如：

- 对应 Order Line；
- 发货数量；
- Carrier；
- Tracking Number；
- 其他 Walmart 要求的发货信息。

人工提交发货：

> 不依赖 ERP 必须先观察到 Amazon / Source 已发货。

采购和 Walmart 发货仍然是两个独立业务过程。

---

### 9.6 ERP 通用模型支持多个包裹 / 多个 Tracking

ERP 通用数据模型不能把：

> 一条 Order Line 永远只能有一个 Shipment / Tracking

写成全局限制。

原因是不同平台、不同国家站点、外部物流和未来能力可能需要多个包裹 / Tracking。

但是对 **Walmart 美国站 V1**：

> 当前不把“同一 Order Line 可通过 US Orders API 拆成多个包裹并分别写入不同 Tracking”声明为已确认能力。

Walmart US Adapter 必须在 Sandbox / Production 实测确认后，才能把该能力标记为 Supported。

因此：

~~~text
ERP Generic Model = 支持 0..N Shipment / Tracking
Walmart US Write Capability = 当前待实测确认
~~~

不能因为 ERP 底层支持，就让美国站页面暴露未经验证的平台写操作。

---

### 9.7 来源物流信息不自动提交 Walmart

即使 ERP 已经读取到 Amazon / Source 的：

- Shipped；
- Carrier；
- Tracking；
- ETA；

也不能因此默认自动向 Walmart Confirm Shipment。

正式保持：

> **读取来源物流事实 ≠ 向 Walmart 提交发货。**

当前 Team 可以人工发货。

未来如果 Team 要自动化，由 Workflow 明确配置触发条件、权限和执行顺序。

---

## 10. 第二批订单 / 采购已确认原则

14. 已登记 Purchase 允许纠错，但必须保留修改历史。
15. 来源采购被取消后，被取消数量重新成为未覆盖采购数量。
16. 来源采购被取消不自动触发新的采购下单。
17. Purchase Refund 首先表示采购侧资金变化，不自动等于 Purchase Cancelled 或商品未履约。
18. 同一采购需求下的重新采购继续属于原 Procurement Task，并新增独立 Purchase。
19. 新的独立采购需求才创建新的 Procurement Task。
20. V1 必须支持人工向 Walmart 提交发货。
21. ERP 通用模型必须支持一个 Order Line 关联多个 Shipment / Tracking；Walmart US 是否支持同一 Order Line 多包裹分别写入，当前待 Sandbox / Production 实测确认。
22. Source Shipment / Tracking 被 ERP 观察到，不自动提交 Walmart 发货。
23. 是否自动 Walmart 发货属于 Team Workflow，而不是 ERP 全局规则。

---

## 11. Walmart 发货修正与售后操作

### 11.1 Walmart 发货前必须已经 Acknowledge

Acknowledge 在 ERP 中仍然是独立动作，不固定绑定在采购前或采购后。

但是对 Walmart：

> **真正执行 Walmart Shipping 时，该订单必须已经 Acknowledged。**

因此：

~~~text
ERP Workflow 不固定 Acknowledge 在采购流程中的位置
但
Walmart Shipping 的平台前置条件 = 已 Acknowledge
~~~

这属于 Walmart 平台能力约束，不是 ERP 全局 Workflow。

---

### 11.2 更正已经提交的 Carrier / Tracking

Walmart 当前允许在一定平台窗口内更正已提交的 Carrier / Tracking。

ERP V1 应提供：

> 更正发货信息

而不是无痕修改旧值。

业务历史需要保留：

~~~text
第一次提交
→ Carrier / Tracking A

后续更正
→ Carrier / Tracking B
~~~

当前有效值可以变为 B，但 A 仍然保留在历史中。

是否仍处于 Walmart 允许更正的窗口：

> 由 Walmart Adapter 根据当前平台规则判断。

超过平台允许窗口时，ERP 不应假装仍然可以修改。

---

### 11.3 Walmart 拒绝发货时不能显示已发货

如果 ERP 发起 Walmart Shipping，但 Walmart 返回拒绝 / 失败：

> 平台订单状态不能因为“用户点过发货”就变成 Shipped。

系统应保留：

- 本次发货请求；
- Walmart 返回结果；
- 错误原因；
- 当前真实订单状态。

运营修正 Carrier / Tracking 等资料后，可以再次执行新的发货操作。

旧失败记录继续保留。

---

### 11.4 同步 Walmart Return

V1 支持从 Walmart 同步：

- Return Order；
- Return Line；
- Quantity；
- Return Reason；
- Return Status；
- Return Tracking；
- Refund Status / Amount；
- 其他平台可获得的售后事实。

同一个 Walmart Return 持续更新同一条 ERP Return 记录。

---

### 11.5 Seller-Fulfilled Return 的人工处理

对于 Walmart 当前允许 Seller 操作的 Seller-Fulfilled Return：

ERP V1 支持有权限的人员执行：

- Approve Return；
- Reject Return；
- Refund。

但按钮 / 操作是否可用：

> 必须根据当前 Return 类型、状态和 Walmart 实际 Capability 动态决定。

不能对所有 Return 永远显示同一组操作。

---

### 11.6 WFS Return 只读

Walmart Fulfillment Services（WFS）Return：

> V1 只允许 View / Sync。

不向卖家提供 Walmart 不允许的：

- Approve；
- Reject；
- Seller-issued Refund。

---

### 11.7 退款支持有 Return 和无 Return 两种情况

ERP 内业务上仍然只有统一的“退款”动作。

退款可以：

~~~text
Order Line
→ Refund
~~~

即没有 Return 直接退款；

也可以：

~~~text
Return
→ Refund
~~~

即针对已有 Return 退款。

两种情况使用同一个 ERP Refund 业务体系。

Walmart Adapter 根据实际场景调用平台当前允许的退款能力。

---

### 11.8 支持全额 / 部分 / 多数量退款

V1 退款需要能够表达：

- Full Refund；
- Partial Refund；
- Refund Quantity；
- Refund Amount；
- Refund Reason；
- 对应 Order Line；
- Return Reference（如果存在）。

例如 Order Line 数量为 3，只退 2 件时：

> ERP 必须知道退款数量是 2，不能默认所有 Partial Refund 都是 1 件。

---

### 11.9 退款前先判断当前可退款范围

ERP 在提交退款前，应基于当前平台事实和 Walmart 规则判断：

- 当前 Order Line 是否允许退款；
- 当前可退款数量；
- 当前剩余可退款金额；
- Return / WFS 类型是否允许卖家执行退款。

不能把明显超过剩余可退款金额 / 数量的请求直接当作正常业务输入。

具体请求格式由开发实现。

---

### 11.10 退款失败保留历史，修正后重新发起

例如：

~~~text
Refund Attempt 1
→ Walmart Rejected
→ 金额不合法

Refund Attempt 2
→ 修正金额后重新提交
→ Success
~~~

正式确认：

> 第二次操作不能把第一次失败记录覆盖成成功。

ERP 必须同时保留：

- 每次退款请求；
- 请求数量 / 金额；
- Walmart 返回结果；
- 最终同步到的平台退款事实。

---

### 11.11 销售端退款与采购端退款继续分开

Walmart Customer Refund：

> 不自动等于 Amazon / Source Purchase 也要 Cancel / Refund。

采购侧是否取消、退货、等待或进行其他处理：

> 由 Procurement / Operations 根据真实情况决定。

两侧资金和履约生命周期继续独立。

---

## 12. Walmart 发货 / 售后已确认原则

24. Acknowledge 不固定属于 ERP 采购流程某一步，但 Walmart Shipping 本身要求订单已 Acknowledged。
25. 已提交的 Carrier / Tracking 在 Walmart 当前允许窗口内可以更正；旧值必须保留历史。
26. Walmart 拒绝发货时不能把 Platform Order Fact 标记成 Shipped。
27. ERP 通用物流模型支持多个 Shipment / Tracking，但 Walmart US 多包裹写入能力当前待实测确认。
28. Seller-Fulfilled Return 按 Walmart 实际能力支持 View / Approve / Reject / Refund。
29. WFS Return 只允许 View / Sync，不提供 Seller Approve / Reject / Refund。
30. Refund 统一建模，可以有关联 Return，也可以没有 Return。
31. V1 支持 Full / Partial Refund，并支持多数量 Order Line 的 Refund Quantity。
32. 退款前应判断当前剩余可退款数量 / 金额和平台可操作条件。
33. 退款失败历史不能被后续成功请求覆盖。
34. Walmart Refund 与 Source Purchase Cancel / Refund 不自动联动。
35. Return / Refund 最终平台事实仍由 Walmart 后续同步 / 回读确认。

---

## 13. 结算、利润与风险相关业务动作

### 13.1 同步 / 导入 Walmart 结算数据

ERP 可以从 Walmart 获取 Recon / Settlement 数据，并形成：

- Settlement Period；
- Settlement Entry；
- 当前累计已入账金额；
- Store 级 Total Payable / Payout 观察。

这些来自 Walmart 的账单与回款事实：

> **普通用户不能直接修改成另一个金额。**

如果发现数据不对：

- 可以重新同步；
- 可以补充内部备注；
- 可以修正 ERP 自己录入错误的内部成本；

但不能人工篡改 Walmart 返回的 Settlement / Payout Platform Fact。

---

### 13.2 重新计算预计利润

预计利润不是人工填写的数字。

它由当前业务事实自动计算，例如：

- Sales Amount；
- 预计 / 实际采购成本；
- 第三方物流费用；
- Team Cost Policy；
- 售后事实；
- 其他已确认成本。

如果利润看起来不对：

> 应修正造成结果错误的业务事实或成本数据，再由 ERP 重新计算。

普通运营不能直接把：

~~~text
Estimated Profit = 12.50
~~~

手工改成：

~~~text
Estimated Profit = 20.00
~~~

---

### 13.3 重新计算当前对账利润

Walmart Settlement 到达后：

> ERP 根据当前累计 Settlement + 实际采购成本 + 物流成本等重新计算当前对账利润。

Current Reconciled Profit 同样：

> **不是人工可直接编辑的结果字段。**

如果后续出现：

- Refund；
- Adjustment；
- 新 Settlement Period；
- Purchase Refund；
- Logistics Cost 修正；

ERP 根据新的真实事实继续重新计算。

---

### 13.4 人工登记 / 修正第三方物流费用

EasyQuickAIERP V1 允许有权限人员人工登记第三方物流费用。

例如：

~~~text
Order Line A
Third-party Logistics Cost = 3.20
~~~

如果录入错误：

> 允许修正，但必须保留修改历史。

至少需要能够追踪：

- 原金额；
- 新金额；
- 修改人；
- 修改时间；
- 修改原因（适用时）。

物流费用变化后，相关 Estimated Profit / Current Reconciled Profit 自动重新计算。

---

### 13.5 修改成本系数 / 经营成本规则

例如某 Store / 采购方的采购核算规则：

~~~text
实际采购金额 × 1.02
~~~

以后改为：

~~~text
实际采购金额 × 1.03
~~~

正式确认：

> 新规则不能无痕重写过去已经发生业务所适用的旧规则。

系统必须保留：

- 规则内容；
- 生效时间；
- 失效时间 / 新版本生效时间；
- 适用范围。

历史采购 / 订单原则上继续按其业务发生时适用的成本规则解释。

---

### 13.6 查询风险结果

ERP 可以针对：

- Brand；
- Product；
- Amazon Seller；

查询当前 Team 的有效风险结果。

风险判断必须结合：

- 当前 Team；
- System Public Blacklist；
- Team Private Blacklist；
- Team Whitelist；
- 不可覆盖硬规则。

---

### 13.7 维护 Team 黑名单 / 白名单

有权限人员可以：

- 添加 Team Private Blacklist；
- 移除 / 停用 Team Private Blacklist；
- 添加 Team Whitelist；
- 移除 / 停用 Team Whitelist。

这些操作必须保留历史和 Actor。

Whitelist 仍然不能覆盖：

- 已确认 TRO；
- 目标平台明确禁售 / 禁止销售等硬规则。

---

### 13.8 扫描在线 Listing 风险

ERP 可以定期或人工触发在线 Listing 风险扫描。

扫描结果首先形成：

> 当前风险事实 / 命中结果。

之后执行什么动作：

> 由 Team Risk Policy / Workflow 决定。

#### 当前 Team

当前 Team 明确采用：

> **在线商品扫描命中需要下架的风险后，自动执行下架动作，而不是只提醒人工。**

#### 其他 Team

其他 Team 可以自行配置：

- 只告警；
- 人工审核后处理；
- 自动下架；
- 其他后续 Workflow。

因此：

> **自动下架是当前 Team 的策略，不是 ERP 对所有 Team 的全局强制行为。**

### 13.9 当前 Team 风险自动下架使用“删除”

当前 Team 已确认：

> **在线 Listing 扫描命中当前 Team 认定需要自动下架的风险后，Walmart 下架动作使用“删除”。**

因此当前 Team 的风险处理流程为：

~~~text
Online Listing
↓
Risk Scan
↓
Effective Risk Match
↓
Current Team Policy 判断需要自动下架
↓
Delete Listing
↓
同步 / 回读 Walmart 实际结果
~~~

这里的“删除”与：

- 库存设为 0；
- 停用；

不是同一个动作。

删除属于高风险平台操作，因此即使由 Workflow 自动执行，也必须继续遵守：

- 当前 Team 已明确启用该自动化策略；
- 执行身份具有 Delete Listing 权限；
- 保留触发的风险证据；
- 保留实际执行结果；
- 不删除 ERP 内 Product / Listing / Order / Audit 等历史数据。

其他 Team 不继承该规则，可以自行选择：

- 只提醒；
- 人工处理；
- 库存设为 0；
- 停用；
- 删除；
- 其他允许的 Workflow。

---

## 14. 财务 / 风险已确认原则

36. Estimated Profit 和 Current Reconciled Profit 都是计算结果，普通用户不能直接手工改数字。
37. 利润不正确时，应修正采购成本、物流费用、Settlement 等原始业务事实，再重新计算。
38. Walmart Settlement / Payout 属于平台财务事实，普通用户不能直接修改。
39. V1 支持人工登记和修正第三方物流费用，修正必须保留历史。
40. Cost Coefficient / Cost Policy 修改必须有生效历史，不能无痕改写过去业务使用的旧规则。
41. 风险查询必须带 Team Context。
42. Team Private Blacklist / Whitelist 的增删改需要正式操作与历史。
43. 在线 Listing 风险扫描本身只产生风险命中事实；后续动作由 Team Policy / Workflow 决定。
44. 当前 Team 选择在线风险命中后自动执行下架，不只做人工提醒。
45. 其他 Team 可以自行决定只提醒、人工处理或自动执行。
46. 当前 Team 风险自动下架的 Walmart 动作为“删除”；其他 Team 不继承该策略。

---

## 15. 下一步

Business Operation Catalog 已覆盖主要经营域。

当前还需要补齐两件事：

1. 确认当前 Team 的风险自动下架具体使用哪一种 Walmart 动作；
2. 做一次完整收口审查，确认是否还有遗漏的正式业务动作，再决定是否把本清单标记为 Baseline。
