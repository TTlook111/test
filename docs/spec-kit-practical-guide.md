# Spec Kit 实战使用指南

> GitHub 官方开源的规范驱动开发工具包。一句话：先用规范定义"做什么"，再让 AI 按规范写代码。

---

## 目录

- [1. 它到底在解决什么问题](#1-它到底在解决什么问题)
- [2. 安装](#2-安装)
- [3. 推荐流程：构想驱动（从项目构想文档出发）](#3-推荐流程构想驱动从项目构想文档出发)
- [4. 标准流程：从 0 开始（没有构想文档时）](#4-标准流程从-0-开始没有构想文档时)
- [5. 每一步深度解析](#5-每一步深度解析)
- [6. 大型项目最佳实践](#6-大型项目最佳实践)
- [7. 生成产物全景图](#7-生成产物全景图)
- [8. 扩展与预设](#8-扩展与预设)
- [9. 踩坑集锦](#9-踩坑集锦)
- [10. 速查表](#10-速查表)

---

## 1. 它到底在解决什么问题

### 没有规范驱动时的痛苦

```
你：帮我做一个 NL2SQL 平台
AI：好的（开始写代码，用了一堆你不喜欢的技术栈）
你：不对，我要用 Spring Boot 不用 Node.js
AI：好的，改了（但架构已经歪了）
你：数据库设计也不对...
AI：（内心：你早说啊）
```

问题出在哪？**你给了 AI 太大的自由度，AI 只能猜。**

### 有规范驱动时

```
你：这是我的项目构想文档（@构想.md）
AI：我按模块拆成了 4 个 spec，你看下对不对
你：可以，生成技术方案吧
AI：方案在这，你审一下
你：没问题，拆任务开始干
```

**每一步你都有机会纠正，AI 不会跑偏。**

### 核心理念

| 原则 | 说明 |
|------|------|
| 先定义再动手 | 规范 = 你和 AI 之间的合同 |
| 多轮迭代 | 不是一次性生成，是逐步细化 |
| 人工审批 | 每一步你都能看、能改、能拒绝 |
| 规范可版本控制 | .md 文件可以 git commit，团队共享 |

---

## 2. 安装

```bash
# 需要 Python 3.11+、Git、uv

# 安装
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git

# 验证
specify version
specify check

# 升级
uv tool install specify-cli --force --from git+https://github.com/github/spec-kit.git@vX.Y.Z
```

---

## 3. 推荐流程：构想驱动（从项目构想文档出发）

**适用场景：** 你已经通过和 AI 反复对话，生成了一份项目构想文档（比如 `项目构想.md`），现在要基于它开始写代码。

这是最自然、最高效的方式。你已经在"对话迭代"中完成了需求分析，Spec Kit 的作用是**把构想文档结构化、拆分成可执行的规范和任务**。

### 3.0 你手上已有的东西

```
你的项目/
└── docs/
    └── 项目构想.md          ← 你和 AI 反复讨论后生成的完整设计文档
```

这份文档里可能包含了：技术栈选型、架构设计、功能模块划分、数据库设计、API 设计等等。它是你脑子里想法的"落地版"。

### 3.1 初始化 Spec Kit

```bash
specify init . --integration claude-code --force
```

`--force` 因为你的项目目录已经有文件了（docs/ 等），需要合并进去而不是覆盖。

初始化后新增的目录：

```
你的项目/
├── .specify/
│   ├── memory/constitution.md       # 空的，待填充
│   ├── scripts/                     # 辅助脚本
│   ├── specs/                       # 空的，待生成
│   └── templates/                   # 内置模板
├── .claude/commands/                # 斜杠命令
├── CLAUDE.md
└── docs/
    └── 项目构想.md                  # 你已有的构想文档
```

### 3.2 Step 1 — Constitution（从构想中提取规矩）

```bash
claude
> /speckit.constitution
```

**实际对话：**

```
AI: 我来帮你建立项目宪法。请告诉我项目的技术栈、代码规范、架构原则和质量标准。

你: 这是我的项目构想文档，请基于它提取这些信息：
    @docs/nl2sql-单库多表版-项目构想.md

    重点关注：
    1. 技术栈选型（构想文档里有详细对比）
    2. 架构分层原则
    3. 代码规范和约束
    4. 安全和质量要求

AI: 我从构想文档中提取了以下内容，生成项目宪法：

    ## 技术栈
    - 后端网关：Java 17 + Spring Boot 3.2 + Maven
    - AI 服务：Python 3.11 + FastAPI + LangGraph
    - 数据库：MySQL 8.0 + Milvus
    - 前端：Vue 3 + TypeScript + Element Plus

    ## 架构原则
    - 网关层只做路由和鉴权，不写业务逻辑
    - AI 服务和业务数据库隔离，通过 HTTP 通信
    - SQL 执行必须经过安全沙箱

    ## 代码规范
    - Java 遵循 Google Java Style，Python 遵循 PEP 8
    - 提交信息用 Conventional Commits
    - 所有 API 统一返回 ResponseResult<T>

    ## 质量标准
    - 单元测试覆盖率 > 80%
    - API P99 延迟 < 500ms
    - 所有 SQL 必须经过安全审查

    是否需要修改或补充？

你: 可以，再加一条：禁止 catch(Exception e) 吞异常

AI: 已更新。constitution.md 已生成。
```

**生成产物：** `.specify/memory/constitution.md`

**关键点：** 不用手动写 constitution，直接喂构想文档，AI 会自动提取。你只需要审查和补充。

---

### 3.3 Step 2 — Specify（从构想中拆分功能模块）

```bash
> /speckit.specify
```

**实际对话：**

```
AI: 请描述你要构建的功能。

你: 基于 @docs/nl2sql-单库多表版-项目构想.md，请按功能模块分别生成独立的 spec。

    模块划分：
    1. 用户认证模块 — 注册、登录、Token 管理、角色权限
    2. NL2SQL 查询模块 — 自然语言转 SQL、Schema 检索、多轮对话
    3. 数据治理模块 — 数据源管理、元数据管理、字段映射
    4. 审计与监控模块 — 查询日志、成本监控、异常告警

    每个模块单独一个 spec.md

AI: 好的，我来逐个生成。

    [生成 user-auth/spec.md]
    [生成 nl2sql-query/spec.md]
    [生成 data-governance/spec.md]
    [生成 audit-monitor/spec.md]

    已生成 4 个功能规范文件。每个 spec 包含：
    - 用户故事
    - 功能需求（FR）
    - 非功能需求（NFR）
    - 验收标准

    是否需要调整模块划分或补充需求？

你: nl2sql-query 的 spec 里，多轮对话的上下文管理要加上"最多保持最近 10 轮"
你: data-governance 先不做，Phase 2 再说

AI: 已更新 nl2sql-query/spec.md。已标记 data-governance 为 Phase 2。
```

**生成产物：**

```
.specify/specs/
├── user-auth/spec.md
├── nl2sql-query/spec.md
├── data-governance/spec.md       # 标记为 Phase 2
└── audit-monitor/spec.md
```

**关键点：**
- **你来决定拆成几个模块、按什么维度拆**，AI 不知道你的业务边界
- 构想文档越详细，生成的 spec 质量越高
- 不需要一次拆完，可以先做核心模块，后面再加

---

### 3.4 Step 3 — Clarify（逐个模块澄清模糊点）

```bash
> /speckit.clarify
```

**实际对话：**

```
你: 先澄清 nl2sql-query 这个模块

AI: 我分析了 nl2sql-query/spec.md，发现以下模糊点：

    1. [高] "多轮对话"的上下文传给 LLM 的方式？
        - 方案 A：把完整历史消息都传给 LLM
        - 方案 B：只传最近 N 轮 + 历史摘要
        - 方案 C：用向量检索相关历史对话

    2. [高] SQL 执行失败时用户看到什么？
        - 原始数据库错误
        - 友好提示 + 错误码
        - AI 生成的解释

    3. [中] Schema 检索的精度要求？
        - 只返回相关表
        - 返回相关表 + 字段描述
        - 返回相关表 + 字段描述 + 示例数据

    4. [低] 查询结果的最大行数？需要分页吗？

你: 1. 方案 B，最近 10 轮 + 摘要
    2. 友好提示 + 错误码，不暴露原始错误
    3. 返回表 + 字段描述，不返回示例数据
    4. 默认最多 1000 行，支持分页

AI: 已更新 spec.md 的 Clarifications 章节。
```

**为什么要逐个模块澄清？**
- 每个模块的模糊点不同，一起澄清容易混乱
- 你可以按优先级先澄清核心模块，其他的后面再补
- 澄清后的 spec 才是真正可以交给 AI 执行的"合同"

---

### 3.5 Step 4 — Plan（为每个模块生成技术方案）

```bash
> /speckit.plan
```

**实际对话：**

```
你: 先为 nl2sql-query 生成技术方案

AI: 基于 Constitution 和 nl2sql-query/spec.md，生成技术方案：

    ## 架构
    [用户] → [Vue 前端] → [Spring Boot 网关] → [FastAPI AI 服务] → [LLM]
                                                  ↓
                                            [MySQL 业务数据]
                                            [Milvus Schema 向量]

    ## 模块拆分
    1. 查询接口（Spring Boot）— 接收自然语言，转发到 AI 服务
    2. Schema 检索（FastAPI）— RAG 检索相关表结构
    3. SQL 生成（FastAPI + LangGraph）— 调用 LLM 生成 SQL
    4. SQL 安全沙箱（FastAPI）— 校验 SQL 安全性
    5. 结果展示（Vue）— 表格 + 图表

    ## 数据模型
    [展示 ER 图和表结构]

    ## API 设计
    [展示接口规范]

    请确认方案。

你: LangGraph 的编排流程详细画一下，我想看每个节点的输入输出

AI: [展示 LangGraph 状态图]
    意图识别 → Schema 检索 → SQL 生成 → SQL 校验 → 返回结果
    每个节点的输入输出如下...

你: 可以，方案确认
```

**生成产物：**

```
.specify/specs/nl2sql-query/
├── spec.md
├── plan.md              # 主方案
├── data-model.md        # 数据库设计
├── research.md          # 技术选型对比
├── quickstart.md        # 环境搭建
└── contracts/
    └── api-spec.json    # API 接口规范
```

---

### 3.6 Step 5 — Tasks（拆解任务）

```bash
> /speckit.tasks
```

**实际对话：**

```
你: 基于 nl2sql-query 的方案，拆解任务

AI: 拆解为以下任务：

    ## Phase 1: 基础设施
    - [ ] T-001: 初始化 Spring Boot 项目 + Maven 依赖
    - [ ] T-002: 初始化 FastAPI 项目 + pyproject.toml
    - [P] T-003: 设计 MySQL 表结构 + 迁移脚本
    - [P] T-004: 初始化 Vue 项目 + 基础布局

    ## Phase 2: 核心功能
    - [ ] T-005: 实现 Schema 检索服务（RAG）
    - [ ] T-006: 实现 SQL 生成服务（LangGraph）
    - [ ] T-007: 实现 SQL 安全沙箱
    - [ ] T-008: 实现查询 API（Spring Boot 网关层）
    - [ ] T-009: 实现查询结果展示（Vue 前端）

    ## Phase 3: 增强功能
    - [ ] T-010: 多轮对话上下文管理
    - [ ] T-011: 审计日志集成

    标注 [P] 的任务可并行执行。

你: T-003 和 T-004 合并，表结构设计放在后端项目里
你: Phase 3 先不做，聚焦 Phase 1 和 2

AI: 已调整。tasks.md 已更新。
```

**生成产物：** `.specify/specs/nl2sql-query/tasks.md`

---

### 3.7 Step 6 — Implement（按任务执行）

```bash
> /speckit.implement
```

**实际对话：**

```
AI: 开始执行 T-001: 初始化 Spring Boot 项目

    [创建 pom.xml]
    [创建 Application.java]
    [创建 application.yml]
    [运行 mvn compile 验证]

    ✅ T-001 完成，编译通过

    接下来执行 T-002...

你: 等一下，T-001 的 pom.xml 里少了 Lombok 依赖，加上

AI: 已添加 Lombok 依赖，重新编译通过。

    继续执行 T-002: 初始化 FastAPI 项目...
```

**你可以随时：**
- 跳过某个任务："T-009 先跳过，前端我后面自己做"
- 修改任务："T-006 的 LangGraph 编排逻辑不对，应该是先意图识别再生成"
- 验证任务："T-005 跑一下测试"

---

### 3.8 其他模块怎么处理

第一个模块（nl2sql-query）走完 Plan → Tasks → Implement 后，继续处理其他模块：

```
> /speckit.plan
你: 接下来为 user-auth 生成技术方案

> /speckit.tasks
你: 拆解 user-auth 的任务

> /speckit.implement
你: 开始执行
```

**每个模块独立走 Plan → Tasks → Implement 流程，但共享同一个 Constitution。**

模块之间的执行顺序由你决定：
```
你: 先做 user-auth，因为其他模块都依赖登录
你: nl2sql-query 和 audit-monitor 可以并行，一个后端一个前端
```

---

### 3.9 构想驱动流程总结

```
你已有的：  项目构想.md（通过对话迭代生成的完整设计文档）

Step 1:  /speckit.constitution  → 喂构想文档 → AI 提取规矩 → constitution.md
Step 2:  /speckit.specify       → 喂构想文档 → AI 按模块拆分 → 多个 spec.md
Step 3:  /speckit.clarify       → 逐个模块澄清模糊点 → 更新 spec.md
Step 4:  /speckit.plan          → 为每个模块生成技术方案 → plan.md + data-model.md + ...
Step 5:  /speckit.tasks         → 为每个模块拆解任务 → tasks.md
Step 6:  /speckit.implement     → 按任务逐步执行 → 写代码、写测试
```

**和标准流程的区别：**

| 维度 | 构想驱动 | 标准流程（从 0 开始） |
|------|----------|----------------------|
| 输入 | 已有项目构想.md | 从头回答 AI 的问题 |
| Constitution | 从构想文档提取 | 手动告诉 AI |
| Specify | 从构想文档拆分 | 手动描述需求 |
| 效率 | 高（已有 80% 的信息） | 低（需要逐步输入） |
| 适合 | 你这种"先想清楚再动手"的人 | 需求还没想清楚的人 |

---

## 4. 标准流程：从 0 开始（没有构想文档时）

**适用场景：** 你还没有构想文档，需求也比较模糊，需要从头开始。

### 4.1 初始化

```bash
specify init nl2sql-platform --integration claude-code
cd nl2sql-platform
```

### 4.2 Constitution — 手动告诉 AI 你的规矩

```bash
claude
> /speckit.constitution
```

```
AI: 请告诉我项目的技术栈、代码规范、架构原则和质量标准。

你: 技术栈：Java 17 + Spring Boot 3.2 + Maven（后端），
    Python 3.11 + FastAPI + LangGraph（AI 服务），
    Vue 3 + TypeScript（前端），
    MySQL 8.0 + Milvus（数据库）

    架构原则：网关层只做路由和鉴权，AI 服务和业务数据库隔离...

    [手动逐条输入]
```

### 4.3 Specify — 手动描述需求

```bash
> /speckit.specify
```

```
AI: 请描述你要构建的功能。

你: 功能名称：NL2SQL 智能查询
    背景：业务人员不会写 SQL，但需要频繁查数据...
    用户故事：1. 作为业务用户... 2. 作为数据分析师...
    验收标准：[ ] ... [ ] ...

    [手动逐条输入]
```

### 4.4 后续步骤

Clarify → Plan → Tasks → Implement，和构想驱动流程一样。

### 4.5 两种流程对比

```
构想驱动（推荐）：
  项目构想.md → /speckit.constitution（提取）→ /speckit.specify（拆分）→ ...

标准流程：
  /speckit.constitution（手动输入）→ /speckit.specify（手动输入）→ ...
```

**建议：** 即使用标准流程，也建议先花时间和 AI 对话，输出一份构想文档。这份文档的价值远超 Spec Kit 本身 — 它是你对项目的完整思考。

---

## 5. 每一步深度解析

### Constitution 的真正价值

不只是"写个文档"。它是 **AI 的行为约束**。

```
没有 Constitution：
  AI 可能用 Node.js 写后端、用 MongoDB 存数据、代码风格随心所欲

有了 Constitution：
  AI 必须用 Spring Boot、必须用 MySQL、必须遵循 Google Java Style
```

**大型项目必做这一步，而且要团队共同审阅。**

### Specify 的关键原则

**只说 What 和 Why，不说 How。**

```
# 错误（混入技术方案）
"用 Redis 缓存 Session，TTL 30 分钟，用 Spring Session 集成"

# 正确（只说需求）
"用户登录后 30 分钟不操作自动登出"
```

为什么？AI 可能有更好的方案（比如 JWT + 过期时间），你把方案写死了反而限制了它。

**但如果你已有构想文档，里面已经包含了技术方案，那也没关系。** 构想驱动的核心是"你已经想清楚了"，不需要刻意把技术细节去掉。

### Clarify 为什么不能跳

```
你说："支持多轮对话"

AI 理解 A：每轮对话独立，只是展示历史记录
AI 理解 B：上下文传给 LLM，让它理解"再按地区分组"是基于上一轮
AI 理解 C：用向量检索历史对话，RAG 方式召回上下文

你不澄清，AI 只能猜。
```

### Plan 阶段要审什么

| 审查点 | 问自己 |
|--------|--------|
| 架构 | 模块划分合理吗？耦合度高吗？ |
| 技术选型 | 用的库/框架我熟悉吗？社区活跃吗？ |
| API 设计 | 接口命名合理吗？返回格式统一吗？ |
| 数据模型 | 表结构范式合理吗？有冗余吗？ |
| 非功能需求 | 性能、安全、可扩展性考虑了吗？ |

### Tasks 的粒度

```
太粗：
  - [ ] 实现整个后端        ← AI 一次写太多，容易出错

太细：
  - [ ] 创建 User.java      ← 没必要单独一个任务
  - [ ] 创建 UserDTO.java   ← 太碎了

刚好：
  - [ ] 实现用户认证模块（注册、登录、Token 刷新）
        路径: backend/src/main/java/.../auth/
        测试: backend/src/test/java/.../auth/
```

**经验法则：一个任务 = 一个开发者半天到一天的工作量。**

---

## 6. 大型项目最佳实践

### 6.1 一个功能一个 spec

```
.specify/specs/
├── user-auth/              # 功能 1：用户认证
│   ├── spec.md
│   ├── plan.md
│   ├── tasks.md
│   └── ...
├── nl2sql-query/           # 功能 2：NL2SQL 查询
│   ├── spec.md
│   ├── plan.md
│   ├── tasks.md
│   └── ...
├── audit-log/              # 功能 3：审计日志
│   ├── spec.md
│   ├── plan.md
│   ├── tasks.md
│   └── ...
└── dashboard/              # 功能 4：数据看板
    ├── spec.md
    ├── plan.md
    ├── tasks.md
    └── ...
```

**每个功能独立走 Plan → Tasks → Implement，共享同一个 Constitution。**

### 6.2 功能之间的依赖

在 Constitution 中定义模块边界：

```markdown
## 模块依赖
- user-auth 是基础模块，所有其他模块依赖它
- nl2sql-query 依赖 user-auth（需要用户信息做审计）
- audit-log 依赖 user-auth（需要操作人信息）
- dashboard 依赖 nl2sql-query 和 audit-log 的数据接口
```

AI 在生成 tasks.md 时会自动处理依赖顺序。

### 6.3 需求变更

```
小变更（加个字段）：直接在 Implement 阶段告诉 AI
中等变更（加用户故事）：更新 spec.md → 重新 tasks
大变更（架构调整）：更新 constitution.md + spec.md → 重新 plan + tasks
```

**规范文件是活的，不是写完就不改了。**

### 6.4 团队协作

```
产品经理  → 写 Specify（需求）
技术负责人 → 写 Constitution + 审 Plan
开发者    → 执行 Tasks + Implement
```

规范文件提交到 Git，所有人通过 Review 对齐理解。

### 6.5 代码质量控制

在 Constitution 中定义质量关卡：

```markdown
## 代码审查
- 所有 AI 生成的代码必须经过人工 Review
- 重点关注：安全漏洞、性能问题、边界情况

## 测试要求
- 单元测试覆盖率 > 80%
- Service 层必须写单元测试
- API 层必须写集成测试

## 安全要求
- 所有用户输入必须校验
- SQL 必须用参数化查询
- 敏感数据必须加密存储

## 代码规范
- 禁止 System.out.println，必须用 SLF4J
- 禁止 catch(Exception e) 吞异常
- 禁止硬编码配置值，必须用 application.yml
```

### 6.6 阶段性策略

```
Phase 1（MVP）：
  Constitution → Specify 核心功能 → Plan → Tasks → Implement
  目标：跑通最小可用版本

Phase 2（增强）：
  为每个新功能单独 Specify → Plan → Tasks → Implement
  目标：逐步完善功能

Phase 3（优化）：
  更新 Constitution（加入更高要求）
  审视已有代码是否符合新标准
  目标：质量提升
```

**不要试图一次性把所有功能都 Specify 完。先做 MVP，再迭代。**

### 6.7 跨服务架构

Spring Boot 网关 + FastAPI AI 服务这种多服务架构：

```
.specify/specs/
├── gateway-api/          # Spring Boot 网关
│   ├── spec.md
│   ├── plan.md
│   └── tasks.md
├── ai-service/           # FastAPI AI 服务
│   ├── spec.md
│   ├── plan.md
│   └── tasks.md
└── shared-contract/      # 服务间接口契约
    ├── spec.md
    └── contracts/
        └── api-spec.json
```

**用 shared-contract 定义服务间的接口契约，确保两个服务的 API 对齐。**

### 6.8 遗留代码引入 Spec Kit

```bash
# 在当前目录初始化
specify init . --integration claude-code --force

# 针对新功能走流程
/speckit.specify    # 只描述新功能
/speckit.plan       # AI 会读取已有代码，基于现有架构做方案
/speckit.tasks
/speckit.implement
```

Plan 阶段 AI 会自动分析已有代码的架构和风格，新代码会尽量保持一致。

### 6.9 文档即代码

所有 .md 文件纳入版本控制：

```bash
git add .specify/
git commit -m "docs(spec): 添加 NL2SQL 查询功能规范"
```

好处：
- 新成员读 spec.md 就能理解功能设计
- 代码和设计文档永远同步
- git diff 可以看到规范的变化历史

---

## 7. 生成产物全景图

```
.specify/
├── memory/
│   └── constitution.md                    ← /speckit.constitution
├── scripts/
│   ├── check-prerequisites.sh
│   ├── common.sh
│   ├── create-new-feature.sh
│   ├── setup-plan.sh
│   └── update-claude-md.sh
├── specs/
│   └── <feature>/
│       ├── spec.md                        ← /speckit.specify + /speckit.clarify
│       ├── plan.md                        ← /speckit.plan
│       ├── data-model.md                  ← /speckit.plan
│       ├── research.md                    ← /speckit.plan
│       ├── quickstart.md                  ← /speckit.plan
│       ├── tasks.md                       ← /speckit.tasks
│       └── contracts/
│           ├── api-spec.json              ← /speckit.plan
│           └── signalr-spec.md            ← /speckit.plan
└── templates/
    ├── plan-template.md
    ├── spec-template.md
    └── tasks-template.md
```

**模板解析优先级（高 → 低）：**
1. `.specify/templates/overrides/` — 项目本地覆盖
2. `.specify/presets/templates/` — 预设模板
3. `.specify/extensions/templates/` — 扩展模板
4. `.specify/templates/` — Spec Kit 内置

---

## 8. 扩展与预设

### 扩展（Extensions）= 加新能力

```bash
specify extension search              # 搜索
specify extension add <name>          # 安装
```

| 分类 | 扩展 | 用途 |
|------|------|------|
| integration | jira, azure-devops, linear | 对接项目管理 |
| code | code-review, security-audit | 代码审查 |
| process | bugfix-workflow | Bug 修复流程 |
| docs | retrospectives | 复盘文档 |

### 预设（Presets）= 改已有行为

```bash
specify preset search                 # 搜索
specify preset add <name>             # 安装
```

适用场景：合规格式、组织标准、中文模板。

```
扩展：给你的工具箱加一把新工具
预设：调整现有工具的配置
```

---

## 9. 踩坑集锦

### 坑 1：Constitution 太笼统

```
# 踩坑
"用 Spring Boot"

# 正确
"Spring Boot 3.2+，Java 17，Maven 构建"
```

### 坑 2：一次性 Specify 所有功能

```
# 踩坑
一次写 10 个功能的 spec → 混乱、遗漏、矛盾

# 正确
一个功能一个 spec → 清晰、可独立交付
```

### 坑 3：跳过 Clarify

```
你说："支持多轮对话"
AI 理解：把历史消息都传给 LLM
你期望：只传最近 5 轮 + 摘要

不澄清 = 返工
```

### 坑 4：Tasks 粒度太粗

```
# 踩坑
- [ ] 实现整个后端

# 正确
- [ ] 实现用户注册接口（Controller + Service + 测试）
- [ ] 实现用户登录接口（Controller + Service + 测试）
```

### 坑 5：Implement 阶段不验证

```
# 踩坑
AI 说"完成" → 你信了 → 集成时发现一堆 bug

# 正确
AI 说"完成" → 你跑测试 → 确认通过 → 继续下一个
```

### 坑 6：不更新规范

```
# 踩坑
需求变了，代码改了，spec.md 还是旧的

# 正确
需求变了 → 先更新 spec.md → 再改代码
```

### 坑 7：忽略 Constitution 中的"禁止项"

```
# 只写了"用 SLF4J 日志"，没说禁止什么
# 结果 AI 到处 catch(Exception e) 吞异常

# 应该明确禁止
"禁止 catch(Exception e) 吞异常"
"禁止硬编码配置值"
"禁止在 Controller 中写数据库查询逻辑"
```

### 坑 8：构想文档太粗就直接 Specify

```
# 踩坑
构想文档只写了"做一个 NL2SQL 平台"→ AI 拆不出有意义的 spec

# 正确
构想文档要包含：技术栈对比、架构设计、功能模块详细描述、数据库设计、API 设计
越详细，Specify 越准
```

---

## 10. 速查表

### CLI 命令

```bash
specify init <name>                    # 新项目
specify init . --force                 # 合并到已有项目
specify version                        # 版本
specify check                          # 环境检查
specify integration list               # 支持的 Agent 列表
specify extension search/add           # 扩展
specify preset search/add              # 预设
```

### 斜杠命令（在 Claude Code 中使用）

```
/speckit.constitution    # 1. 项目宪法
/speckit.specify         # 2. 功能规范
/speckit.clarify         # 3. 澄清模糊点 [可选]
/speckit.plan            # 4. 技术方案
/speckit.tasks           # 5. 任务拆解
/speckit.analyze         # 一致性检查 [可选]
/speckit.checklist       # 质量清单 [可选]
/speckit.implement       # 6. 执行实现
/speckit.taskstoissues   # 转 GitHub Issues
```

### 两种流程速查

```
构想驱动（推荐）：
  构想.md → constitution（提取）→ specify（拆分）→ clarify → plan → tasks → implement

标准流程：
  constitution（手动）→ specify（手动）→ clarify → plan → tasks → implement

已有项目加功能：
  init --force → specify → plan → tasks → implement

修 bug：直接用 Claude Code，不需要 Spec Kit
```

### 工作流速查

```
Constitution  → 定规矩（技术栈、规范、约束）
Specify       → 说需求（What + Why，或从构想文档拆分）
Clarify       → 补模糊（AI 问你答）
Plan          → 出方案（架构、选型、API、数据模型）
Tasks         → 拆任务（有序、有依赖、可并行）
Implement     → 干活（按任务逐个实现、验证、标记完成）
```

---

> **项目地址：** https://github.com/github/spec-kit
>
> **一句话总结：** 用构想文档替代手动输入，让 Spec Kit 帮你结构化、拆分、执行。规范越清晰，AI 输出越准确。
