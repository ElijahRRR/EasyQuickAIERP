# Team / User / Permission / Organization Model v0.3

> 状态：已确认  
> 所属阶段：ERP 阶段 0 — 业务建模

# 1. Team 是租户与数据隔离边界

Team 表示：

> 一个独立使用 EasyQuickAIERP 的业务工作空间。

Team 不一定对应正式注册公司，也可以是一个个人卖家或独立经营主体。

新用户自主注册以后：

```text
Create User
↓
Create Default Team
↓
User becomes Team Administrator
```

即：

> 每一个独立注册用户默认自成一个 Team。

---

# 2. 一个 User 只能属于一个 Team

当前业务规则：

> 一个 User Account 永远只属于一个 Team。

不存在一个 User 同时属于多个 Team 的业务模式，也不需要普通用户的 Multi-Team Workspace Switching。

不同 Team 的数据彼此隔离。

---

# 3. 已有 Team 的成员不能自行加入

如果某个员工需要进入已有 Team：

必须由 Team 内拥有成员管理权限的账户：

- Invite；
- 或直接 Create Account。

不能由员工自己注册以后选择加入某个 Team。

因此 Team Membership 始终由 Team 内部授权产生。

---

# 4. Administrator 是 Team 最高管理身份

不单独建立 Team Owner 与 Team Administrator 两套业务概念。

统一为：

> **Team Administrator**

Team 首个创建者默认成为该 Team 的第一个 Administrator。

---

# 5. 用户离开 Team 不删除历史 Actor

用户离职以后可以：

- Disable Login；
- Remove Active Access；
- Remove Store Assignment；
- Remove Group Membership。

但必须永久保留历史业务事实，例如：

- Order Audit；
- Purchase；
- Refund；
- Listing Operation；
- Workflow Creation；
- Store Responsibility；
- 其他操作记录。

因此：

> User Disabled ≠ Historical Actor Deleted。

---

# 6. Organization 基本结构

组织结构保持简单：

```text
Team
├── Users
└── Groups
```

Group 是 Team 内的一组 User 的集合，例如：

- 运营一组；
- 运营二组；
- 采购组；
- 财务组；
- 项目组。

---

# 7. Group 本身不携带业务权限

Group 不自动拥有：

- Permission；
- Role；
- Store Scope；
- Workflow；
- Risk Policy；
- 其他业务属性。

它首先只是：

> **成员集合 / 组织分组。**

因此：

```text
Group Membership
≠
Permission Grant
```

加入某个 Group 不会自动获得对应业务权限。

---

# 8. 一个 User 可以属于多个 Group

例如：

```text
张三
├ 运营一组
└ 新店项目组
```

是允许的。

因此：

> User : Group = 多对多关系。

Group 主要用于组织人员、管理团队结构、Store 责任分组、统计与筛选，而不是权限继承主体。

---

# 9. Store Assignment 与 Permission 完全分开

Store 可以拥有：

- 一个 Primary Operator；
- 多个 Collaborators；
- 一个 Operation Group；

也可以全部不设置。

这些表示：

> 谁在业务上负责这个 Store。

但不直接决定谁拥有系统访问权限。

因此：

```text
Business Responsibility
≠
System Authorization
```

---

# 10. Store 可以完全没有人员归属

Store 不要求：

- 必须有负责人；
- 必须有运营组；
- 必须有 Collaborator。

即使没有 Assignment，ERP 仍然可以正常同步和管理 Store。

系统通过 Audit Log 记录谁实际执行了什么动作。

---

# 11. Procurement User 不要求绑定 Store

采购人员可以处理多个 Store 的采购任务。

因此采购权限更适合依据：

- 被授权的业务范围；
- Procurement Task；

而不是把采购员长期固定绑定某一家 Store。

---

# 12. 权限模型的核心是 Permission Grant

权限最终回答：

> **某个 Actor，可以在哪些业务对象上执行哪些操作。**

更准确地表达为：

```text
Actor
+
Resource Scope
+
Action
=
Permission Grant
```

---

# 13. Team Scope 是天然边界

因为一个 User 永远只属于一个 Team：

> Team Scope 不需要每一次授权都重新选择。

权限天然只能发生在当前 User 所属 Team 内。

一般权限配置可以从：

```text
Platform
↓
Store
↓
Business Function
↓
Action
```

开始。

---

# 14. 第一层：Platform Scope

管理员可以先选择用户允许访问的平台，例如：

- Walmart；
- AliExpress；
- TEMU；
- TikTok；
- 未来其他 Marketplace。

如果没有某个平台权限，则该平台下的 Store 权限自然不可用。

---

# 15. 第二层：Store Scope

在某个平台下，可以选择：

### All Stores

```text
Walmart
→ All Stores
```

或：

### Selected Stores

```text
Walmart
→ Store A
→ Store C
→ Store F
```

因此权限范围可以精确到：

> Platform + Store Set。

---

# 16. 第三层：Functional Permission

在对应 Store Scope 内，再决定能做什么。

例如：

## Product

- View；
- Edit；
- Delete；
- Import / Export；
- Product Audit。

## Listing

- View；
- Create；
- Edit；
- Retire；
- Delete；
- Update Price；
- Update Inventory。

## Order

- View；
- Audit；
- Cancel；
- Shipment。

## Procurement

- View；
- Claim Task；
- Purchase；
- Edit Purchase。

## After-sales

- View Return；
- Refund。

## Finance

- View Settlement；
- View Profit；
- View Payout。

## Administration

- Manage Store；
- Manage Store Policy；
- Manage Workflow；
- Manage Users；
- Manage Permission。

阶段 0 不确定最终权限项清单，只确认 ERP 必须能够控制具体业务能力。

---

# 17. 权限可以采用统一 Store Scope

可以提供：

> **统一设置 Store Permission**

例如：

```text
User A
Platform = Walmart
Stores = A001, A002, A003
```

之后所有被授予的功能默认都作用于这三个 Store。

---

# 18. 也可以按功能分别设置 Store Scope

还需要支持：

> **按功能单独设置。**

例如：

```text
Order View
→ All Walmart Stores
```

但是：

```text
Refund
→ Store A only
```

因此：

> Data Scope 不是 User 全局只有一个值。

它可以根据具体 Function / Permission 不同而不同。

---

# 19. 权限关系

权限可以表达为：

```text
User / Digital Actor
↓
Platform Scope
↓
Store Scope
↓
Module / Resource
↓
Action
```

---

# 20. “自己负责的数据”不是硬权限规则

普通用户看到哪些数据，由 Administrator 实际授予的 Platform / Store / Function Permission 决定。

Store Assignment 可以帮助管理员快速决定应该给谁什么权限，但 Assignment 本身不是 Authorization。

---

# 21. 历史操作不会产生权限

如果某个 User 曾经处理过某个 Store 的订单，这只产生 Audit History。

不会因此永久获得该 Store 的访问权。

因此：

```text
Historical Action
≠
Permission
```

---

# 22. Role Template 是权限配置模板

管理员不需要对每一个 User 从零重复勾选全部 Permission。

Team Administrator 可以创建：

> **Role Template / Permission Template**

例如普通运营、财务、采购等模板。

---

# 23. Role Template 不是组织 Role

Role Template 的含义是：

> 一组可复用的 Permission Preset。

它不是：

- Group；
- Department；
- Job Title；
- Organization Position。

因此：

```text
Group
≠
Role Template
```

---

# 24. Role Template 用于一键应用

管理员可以：

```text
Create Permission Template
↓
Select User
↓
Apply Template
```

之后仍然可以根据需要调整该成员的实际 Permission。

---

# 25. Group 不承担 Role Template 的职责

不采用：

```text
User joins Procurement Group
↓
Automatically gets Procurement Permissions
```

而是：

```text
User joins Procurement Group
```

仅作为组织关系。

权限另行通过 Permission Template 或直接授权完成。

---

# 26. 当前不需要复杂字段级敏感权限

当前业务中通常不存在大量同一 Team 内必须隐藏字段的需求。

因此暂不优先建设：

- Column-level Permission；
- Field Masking；
- 每个字段单独授权。

重点先放在：

> Platform / Store / Function / Action。

---

# 27. 系统 Secret 仍然不能等同普通业务数据

Store API Credential / Secret 属于系统连接秘密。

因此：

> 有 Store 管理权限不意味着可以直接查看 API Secret 明文。

这一点属于系统安全边界。

---

# 28. Human User 是一种 Actor

所有重要操作都要知道谁执行了动作，例如：

- Audit；
- Purchase；
- Refund；
- Delete Listing；
- Change Price；
- Manage Workflow。

Actor 可以是 Human User。

---

# 29. Digital Employee 也是 Actor

未来：

- Workflow；
- Scheduler；
- AI Agent；
- Automation；
- Open API Integration；

都可能执行真实业务动作。

这些都应成为可识别的：

> Digital Actor / Digital Employee。

---

# 30. Digital Employee 由 Team 授权

由拥有相应权限的人：

```text
Create Digital Employee
↓
Grant Permissions
↓
Activate
```

数字员工拥有自己的：

- Platform Scope；
- Store Scope；
- Functional Permission。

---

# 31. Digital Employee 不继承创建者权限

例如 Administrator 创建 Price Bot，不能因为 Administrator 能管理整个 Team，就让 Price Bot 自动拥有整个 Team 权限。

应该只授予其所需的最小权限范围。

---

# 32. 创建者离职不影响数字员工身份

数字员工属于 Team，而不是属于创建者个人。

因此创建它的人离职以后：

> Workflow 不需要自动失效。

Team Administrator 可以继续查看、修改权限、Disable 或 Delete。

---

# 33. Human 与 Digital Actor 使用相同权限逻辑

长期目标：

```text
Human User
Workflow
AI Agent
API Integration
```

全部遵守：

```text
Platform
+
Store Scope
+
Function
+
Action
```

同一权限边界。

---

# 33.1 System Sync Service 是独立后台 Actor

需要把系统同步身份与 Human / Digital Employee 区分。

例如：

- Order Sync；
- Return Sync；
- Settlement Sync；
- Listing State Sync；

可以由 Team / Store 级 Platform Sync Service 执行。

它不应长期冒充最初绑定 Store 的 Human User。

因此某个创建 Store 的员工离职以后：

> Store 的平台事实同步不应该因此停止。

但 System Sync Service 只能获得完成平台事实同步所需的受限能力。

不能因为它属于 System Actor，就自动拥有：

- Refund；
- Cancel Sales Order；
- Change Price；
- Change Inventory；
- Take Down Listing；
- 其他主动高价值业务动作。

---

# 33.2 后台 Actor 的权限在真正执行时重新校验

对于 User Delegated Job / Digital Employee Job：

> 入队时有权限，不代表几小时后真正执行时仍然有权限。

因此需要同时区分：

```text
Requested By
= 谁触发 / 创建了任务

Execute As
= 真正执行时按哪个 Actor 的 Permission 校验
```

实际 Business Operation 执行前，必须检查 Execute As 的当前：

- Actor Enable State；
- Platform Scope；
- Store Scope；
- Function；
- Action。

如果 Permission 在任务排队后被撤销：

> 尚未执行的后续业务动作不得继续。

已成功发生的外部平台动作保留历史，不进行假回滚。

长任务可以形成 Partial Success。

一次性用户委托任务因为撤权停止后，即使后来重新授予权限：

> 不应自动恢复旧任务，原则上应重新确认 / 重新提交。

Digital Employee 同样按其自身当前 Permission 校验，不继承创建者的历史权限。

---

# 34. 每一个操作都需要 Audit Actor

重要业务动作应记录：

- Actor Type；
- Actor Identity；
- Action；
- Target；
- Time；
- Result。

这样人工和数字员工的动作都可追踪。

---

# 35. Organization、Assignment、Permission、Template 必须分开

最终四个概念是：

## Organization

```text
Team
Group
User
```

回答：

> 人属于哪个组织。

## Assignment

```text
Store → Primary Operator / Collaborator / Group
```

回答：

> 谁负责什么业务。

## Permission

```text
Actor → Platform → Store → Function → Action
```

回答：

> 谁能操作什么。

## Role Template

```text
Reusable Permission Preset
```

回答：

> 如何快速批量配置权限。

因此：

```text
Organization
≠
Assignment
≠
Permission
≠
Role Template
```

---

# 36. 当前组织与权限结构

```text
Team
│
├── Administrators
├── Users
├── Groups
│   └── Member Collections Only
│
├── Stores
│   ├── Optional Primary Operator
│   ├── Optional Collaborators
│   └── Optional Group Assignment
│
├── Role Templates
│   └── Permission Presets
│
└── Digital Employees
    ├── Workflow
    ├── Automation
    ├── AI Agent
    └── Integration
```

---

# 37. 当前权限结构

```text
Actor
↓
Platform
↓
Store Scope
↓
Module / Resource
↓
Action
```

并允许：

- Unified Store Scope；
- Per-function Store Scope。

---

# 38. 当前确认的业务原则

1. Team 是独立租户和数据隔离边界。
2. Team 不一定必须对应公司。
3. 新注册 User 默认创建自己的 Team。
4. Team 首个创建者默认成为 Administrator。
5. 不单独建立 Owner 与 Administrator 两套身份。
6. 一个 User Account 只能属于一个 Team。
7. 已有 Team 成员只能由有权限的成员邀请或创建。
8. User 离职后历史 Actor 数据永久保留。
9. Group 是 Team 内的成员集合。
10. 一个 User 可以加入多个 Group。
11. Group 本身不承载 Permission。
12. Group Membership 不产生隐式权限。
13. Store Primary Operator / Collaborator / Group 均为可选 Assignment。
14. Store 可以完全没有人员 Assignment。
15. Assignment 不等于 Authorization。
16. Procurement User 不要求绑定固定 Store。
17. 权限必须明确到 Platform / Store / Function / Action。
18. Store Scope 可以选择 All Stores 或 Selected Stores。
19. 权限可以统一使用同一个 Store Scope。
20. 也允许按不同 Function 设置不同 Store Scope。
21. 数据可见范围由 Permission Grant 决定，而不是由负责人自动决定。
22. 历史 Action 不会自动产生持续访问权。
23. Administrator 可以创建 Role / Permission Template。
24. Role Template 是权限预设，不是组织角色。
25. Template 可以一键应用给成员，减少重复配置。
26. 当前不优先建设复杂字段级权限。
27. API Credential 等 Secret 仍由系统安全边界保护。
28. Human 与 Digital Employee 都是 Actor。
29. Digital Employee 拥有自己独立的 Permission。
30. Digital Employee 不继承创建者全部权限。
31. Digital Employee 属于 Team，创建者离职不会自动导致其停止。
32. Human / Workflow / AI / Integration 长期使用相同业务权限边界。
33. System Sync Service 是独立受限后台 Actor，不依赖最初绑定 Store 的用户持续存在。
34. User Delegated / Digital Employee Job 在真正执行 Business Operation 时必须重新校验当前 Permission。
35. 后台任务需要区分 Requested By 与 Execute As。
36. Permission 撤销后尚未执行的动作不得继续，已经发生的动作保留历史。
37. 重要操作必须留下明确 Actor Audit。

---

# 39. 建模结论

EasyQuickAIERP 的权限模型不应该简单设计成：

```text
Administrator
Operator
Procurement
Finance
```

几个硬编码角色。

更合理的模型是：

```text
Permission
=
Platform Scope
+
Store Scope
+
Business Function
+
Action
```

Role Template 只是对这一组 Permission 的可复用配置。

而 Group 只负责组织成员。

Store Assignment 只负责表达经营责任。

因此真正的边界是：

> **组织关系决定“你是谁、和谁一起工作”；Assignment 决定“你负责什么”；Permission 决定“你实际上能做什么”。**
