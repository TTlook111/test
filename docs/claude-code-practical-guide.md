# Claude Code 实战使用指南

> 从安装到高级用法，覆盖日常开发、代码审查、Git 工作流、团队协作等全部实战场景。

---

## 目录

- [1. 快速开始](#1-快速开始)
- [2. 核心交互模式](#2-核心交互模式)
- [3. 代码开发实战](#3-代码开发实战)
- [4. 代码审查与安全](#4-代码审查与安全)
- [5. Git 工作流](#5-git-工作流)
- [6. 项目管理与持久化](#6-项目管理与持久化)
- [7. 高级功能](#7-高级功能)
- [8. 多模块项目实战：一个模块一个会话](#8-多模块项目实战一个模块一个会话)
- [9. 缓存优化：怎么用才省钱](#9-缓存优化怎么用才省钱)
- [10. 团队协作](#10-团队协作)
- [11. 常见问题与排错](#11-常见问题与排错)
- [12. 最佳实践：社区经验总结](#12-最佳实践社区经验总结)
- [附录：速查卡片](#附录速查卡片)

---

## 1. 快速开始

### 1.1 安装

```bash
# macOS / Linux / WSL
npm install -g @anthropic-ai/claude-code

# Windows (原生 PowerShell)
npm install -g @anthropic-ai/claude-code

# 验证安装
claude --version
```

**前置要求：**
- Node.js >= 18
- 有效的 Anthropic API Key 或 Claude Pro/Max 订阅

### 1.2 认证

```bash
# 方式一：交互式登录（推荐，浏览器跳转）
claude

# 方式二：API Key 环境变量
export ANTHROPIC_API_KEY="sk-ant-..."
claude

# 方式三：在项目中配置 .env
echo "ANTHROPIC_API_KEY=sk-ant-..." > .env
```

### 1.3 首次使用

```bash
# 进入项目目录，启动 Claude Code
cd your-project
claude

# 直接提问
> 这个项目是做什么的？帮我梳理一下架构

# 执行任务
> 帮我给 UserService 添加一个根据邮箱查找用户的方法
```

### 1.4 常用启动参数

```bash
# 非交互模式（适合脚本/CI）
claude -p "运行所有测试并报告结果"

# 管道输入
cat error.log | claude -p "分析这个错误日志"

# 指定模型
claude --model claude-sonnet-4-6

# 继续上次对话（自动恢复最近一次会话）
claude --continue

# 恢复指定对话（弹出历史会话列表，或指定 session-id）
claude --resume
claude --resume <session-id>

# 恢复与 PR 关联的会话
claude --from-pr <pr-number>

# 创建新会话分支（配合 --resume 使用，不修改原会话）
claude --resume <session-id> --fork-session

# 给会话命名（在 /resume 列表中显示）
claude --name "用户认证模块开发"

# 创建 git worktree 隔离环境
claude --worktree

# 设置最大花费限额
claude -p "运行测试" --max-budget-usd 0.5
```

---

## 2. 核心交互模式

### 2.1 斜杠命令速查

**会话与上下文管理：**

| 命令 | 用途 |
|------|------|
| `/clear` | 清除当前对话上下文，重新开始 |
| `/compact` | 压缩对话历史，释放上下文空间（可附加自定义提示指导压缩方式） |
| `/resume` | 恢复之前的对话会话（弹出历史会话列表供选择） |
| `/rewind` | 回退到当前会话中的某个历史节点（逐步回退检查点） |

**配置与认证：**

| 命令 | 用途 |
|------|------|
| `/config` | 查看/修改配置 |
| `/login` | 登录 / 切换账号 |
| `/logout` | 登出 |
| `/model` | 切换模型（如 opus、sonnet、haiku） |
| `/permissions` | 查看/管理权限设置 |
| `/vim` | 切换 Vim 键绑定模式 |

**开发工具：**

| 命令 | 用途 |
|------|------|
| `/init` | 初始化 CLAUDE.md 文件 |
| `/review` | 审查 PR |
| `/security-review` | 安全审查当前分支的变更 |
| `/simplify` | 审查已修改代码的质量和效率，并修复问题 |
| `/fast` | 切换快速模式（Opus 加速输出） |
| `/mcp` | 管理 MCP 服务器连接 |
| `/terminal-setup` | 设置终端集成（如 Shift+Enter 换行） |

**信息查看：**

| 命令 | 用途 |
|------|------|
| `/help` | 查看帮助信息 |
| `/cost` | 查看当前会话的 token 消耗和费用 |

### 2.2 Plan 模式

当任务复杂时，先进入计划模式对齐方案，再动手实现：

```
> 帮我重构认证模块，支持 JWT 和 OAuth2 两种方式
```

Claude Code 会自动进入 Plan 模式：
1. 探索代码库，理解现有结构
2. 设计实现方案
3. 呈现计划供你审批
4. 审批后逐步执行

**手动触发计划模式：** 在对话中说"先别动手，帮我做个计划"。

### 2.3 权限模式

Claude Code 有三种权限模式：

| 模式 | 说明 | 适用场景 |
|------|------|----------|
| **Ask** | 每次写操作都询问 | 初次使用、敏感项目 |
| **Auto-edit** | 自动编辑文件，运行命令仍需确认 | 日常开发 |
| **Yolo** | 全部自动执行 | CI/CD、信任度高的场景 |

切换方式：启动时 `claude --dangerously-skip-permissions`，或在对话中用 `/permissions` 调整。

---

## 3. 代码开发实战

### 3.1 编写新功能

```
> 帮我实现一个文件上传服务，要求：
> - 支持图片和 PDF
> - 文件大小限制 10MB
> - 存储到本地 uploads/ 目录
> - 返回文件访问 URL
> - 写完整的单元测试
```

Claude Code 会：
1. 阅读现有代码结构和依赖
2. 创建 `FileUploadService.java`
3. 编写对应的 `FileUploadServiceTest.java`
4. 如需修改 `pom.xml` 添加依赖，会一并处理

### 3.2 修复 Bug

```
> 用户反馈登录时偶尔会报 NullPointerException，
> 错误发生在 LoginController.java:45 行附近，帮我排查并修复
```

或者直接贴错误日志：

```
> 帮我修复这个错误：
>
> java.lang.NullPointerException: Cannot invoke "String.length()" because "username" is null
>     at com.example.service.UserService.validateUsername(UserService.java:32)
>     at com.example.controller.AuthController.login(AuthController.java:45)
```

### 3.3 代码重构

```
> 把 UserController 里的数据库查询逻辑抽到 UserRepository，
> 遵循现有的 Repository 模式（参考 UserRepository 的写法）
```

```
> 这个方法太长了（150 行），帮我拆分成更小的方法，保持语义清晰
```

### 3.4 编写测试

```
> 给 OrderService 写单元测试，覆盖以下场景：
> - 正常下单
> - 库存不足时下单失败
> - 用户不存在时抛异常
> - 并发下单的幂等性
> 使用 Mockito mock 依赖
```

### 3.5 代码解释

```
> 解释一下这个正则表达式的含义：
> ^(?=.*[A-Z])(?=.*[a-z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$
```

```
> 帮我读懂这个项目的认证流程，从登录请求到返回 Token 的完整链路
```

### 3.6 多文件批量修改

```
> 把项目中所有的 System.out.println 替换为 SLF4J 日志调用
> - 已有 Logger 的类直接用
> - 没有的自动添加 private static final Logger log = LoggerFactory.getLogger(XXX.class)
```

### 3.7 生成配置文件

```
> 帮我生成 docker-compose.yml，需要：
> - MySQL 8.0（端口 3306，数据持久化）
> - Redis 7（端口 6379）
> - 本项目的 Spring Boot 应用（依赖 MySQL 和 Redis）
```

---

## 4. 代码审查与安全

### 4.1 PR 审查

```bash
# 在终端中直接审查
claude /review

# 或者审查特定 PR
claude -p "审查 PR #42 的代码变更，重点关注安全性和性能"
```

Claude Code 会自动：
- 获取 PR 的 diff
- 分析代码质量、潜在 bug、安全问题
- 给出改进建议

### 4.2 安全审查

```bash
claude /security-review
```

检查项包括：
- SQL 注入
- XSS 漏洞
- 硬编码密钥/密码
- 不安全的反序列化
- 权限校验缺失
- OWASP Top 10

### 4.3 代码质量审查

```
> 审查 src/main/java/com/example/service/ 目录下所有代码，
> 重点关注：
> - 是否有资源泄漏（未关闭的流、连接）
> - 异常处理是否合理
> - 是否有线程安全问题
```

### 4.4 简化已修改代码

```bash
# 审查当前分支的改动，寻找可简化之处
claude /simplify
```

---

## 5. Git 工作流

### 5.1 智能提交

```
> 帮我提交当前改动
```

Claude Code 会：
1. 运行 `git status` 和 `git diff` 分析变更
2. 根据项目提交历史风格生成 commit message
3. 执行 `git add` 和 `git commit`

### 5.2 创建 PR

```
> 帮我从当前分支创建一个 PR 到 main
```

Claude Code 会：
1. 分析分支上的所有 commit
2. 生成 PR 标题和描述（含变更摘要、测试计划）
3. 使用 `gh pr create` 创建

### 5.3 分支管理

```
> 从 main 创建一个 feature/user-profile 分支
> 然后帮我实现用户资料编辑功能
```

### 5.4 解决冲突

```
> 当前有 merge 冲突，帮我解决
> 优先保留 main 分支的逻辑，但保留当前分支新增的 validateEmail 方法
```

### 5.5 Git 操作查询

```
> 最近一周谁改了 OrderService.java？改了什么？
```

```
> 帮我看看 feature/auth 分支相对 main 多了哪些 commit
```

---

## 6. 项目管理与持久化

### 6.1 CLAUDE.md — 项目级记忆

CLAUDE.md 是项目的"说明书"，每次对话自动加载：

```bash
# 自动生成
claude /init
```

**推荐的 CLAUDE.md 结构：**

```markdown
# 项目名称

## 构建与运行
- `mvn clean install` — 构建
- `mvn spring-boot:run` — 启动
- `mvn test` — 运行测试

## 代码规范
- 使用 SLF4J 日志，不用 System.out
- Controller 不写业务逻辑，全部委托 Service
- 数据库字段用 snake_case，Java 字段用 camelCase

## 架构说明
- 单体 Spring Boot 应用，分层：Controller → Service → Repository
- 数据库：MySQL 8.0
- 缓存：Redis

## 注意事项
- 不要修改 src/main/resources/db/migration/ 下的已执行迁移文件
- API 返回格式统一用 ResponseResult<T> 包装
```

### 6.2 Memory 系统 — 跨对话记忆

Memory 让 Claude Code 在不同对话间记住关键信息：

```
> 记住：这个项目的数据库密码在 .env 文件里，不要提交到 git
```

```
> 记住：我习惯用 Lombok 的 @Slf4j，不要手动声明 Logger
```

**Memory 的类型：**

| 类型 | 用途 | 示例 |
|------|------|------|
| `user` | 用户偏好和角色 | "我是后端开发，熟悉 Spring Boot" |
| `feedback` | 行为纠正/确认 | "不要自动提交代码，先让我确认" |
| `project` | 项目状态和决策 | "v2.0 计划 6 月上线，正在做性能优化" |
| `reference` | 外部资源指引 | "Bug 跟踪在 Jira 的 NL2SQL 项目中" |

### 6.3 Task 系统 — 任务追踪

对于多步骤任务，Claude Code 会自动创建任务列表跟踪进度：

```
> 帮我完成以下任务：
> 1. 给 User 实体添加 phone 字段
> 2. 更新数据库迁移脚本
> 3. 修改 DTO 和 VO
> 4. 更新 API 文档
> 5. 写单元测试
```

你也可以手动管理：
```
> 先做第 3 步，第 2 步暂时跳过
```

---

## 7. 高级功能

### 7.1 Hooks — 自动化行为

Hooks 在特定事件触发时自动执行 shell 命令。

**配置文件：** `.claude/settings.json`

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write $CLAUDE_FILE_PATH"
          }
        ]
      }
    ],
    "PreCommit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "mvn spotless:check"
          }
        ]
      }
    ]
  }
}
```

**常见 Hook 事件：**

| 事件 | 触发时机 |
|------|----------|
| `PreToolUse` | 工具调用前 |
| `PostToolUse` | 工具调用后 |
| `PreCommit` | git commit 前 |
| `PostCommit` | git commit 后 |
| `Notification` | Claude 发送通知时 |

**实用场景：**
- 文件保存后自动格式化（Prettier / Spotless）
- 提交前自动跑 lint 检查
- 编辑特定文件后自动运行相关测试

### 7.2 MCP Servers — 扩展工具能力

MCP (Model Context Protocol) 让 Claude Code 连接外部工具：

**配置方式（项目级）：** `.claude/settings.json`

```json
{
  "mcpServers": {
    "mysql": {
      "command": "npx",
      "args": ["-y", "@benborla29/mcp-mysql-server"],
      "env": {
        "MYSQL_HOST": "localhost",
        "MYSQL_PORT": "3306",
        "MYSQL_USER": "root",
        "MYSQL_PASSWORD": "...",
        "MYSQL_DATABASE": "nl2sql"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "./docs"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_..."
      }
    }
  }
}
```

**常用 MCP Server：**

| Server | 用途 |
|--------|------|
| `@benborla29/mcp-mysql-server` | 查询 MySQL 数据库 |
| `@modelcontextprotocol/server-postgres` | 查询 PostgreSQL |
| `@modelcontextprotocol/server-filesystem` | 安全的文件系统访问 |
| `@modelcontextprotocol/server-github` | GitHub API 操作 |
| `@modelcontextprotocol/server-slack` | Slack 消息 |
| `mcp-server-redis` | Redis 操作 |

**使用效果：**

```
> 查一下数据库里有多少张表，列出来
> （Claude Code 通过 MCP 直接查询 MySQL）

> 帮我在 Redis 里设置一个 5 分钟过期的缓存
```

### 7.3 定时任务

```bash
# 每 10 分钟检查一次构建状态
/loop 10m 检查 CI 构建状态，如果有失败就通知我

# 定时提醒
> 提醒我下午 3 点检查部署状态
```

### 7.4 子代理（Agent）

对于复杂任务，Claude Code 会自动派生子代理并行处理：

```
> 帮我同时做以下事情：
> 1. 分析项目的依赖安全性
> 2. 检查是否有已知漏洞的库
> 3. 审查代码中的硬编码密钥
```

Claude Code 可能会并行启动 3 个子代理分别处理，提高效率。

### 7.5 自定义 Skills

你可以创建自定义的 Skill（斜杠命令）：

**位置：** `.claude/skills/my-skill.md`

```markdown
---
name: gen-api
description: 生成 RESTful API 端点
---

生成一个完整的 RESTful API 端点，包括：
1. Controller（含 Swagger 注解）
2. Service 接口和实现
3. Repository
4. DTO 和 VO
5. 单元测试

遵循项目现有的代码规范。
```

使用：`/gen-api` 或 `> 帮我用 gen-api skill 生成用户管理的 CRUD 接口`

---

## 8. 多模块项目实战：一个模块一个会话

一个真实项目不可能在一个 Claude Code 会话里做完所有事。正确做法是**每个模块、每个功能、每个任务都开独立会话**。

### 8.1 为什么不能一个会话做完

```
一个会话做所有模块的问题：
  - 对话越来越长，token 越来越多，费用飙升
  - 上下文被压缩后，前面的细节丢失
  - AI 跨模块修改时容易引入 bug
  - 无法并行（一个会话只能串行）
```

### 8.2 正确的会话划分策略

```
项目：NL2SQL 平台
├── 会话 1：项目初始化 + 基础架构搭建
├── 会话 2：用户认证模块
├── 会话 3：NL2SQL 查询模块
├── 会话 4：审计日志模块
├── 会话 5：前端页面开发
├── 会话 6：Bug 修复（登录偶发 500）
├── 会话 7：性能优化（SQL 生成太慢）
└── 会话 8：部署配置（Docker + CI/CD）
```

**原则：一个会话 = 一个明确的任务边界。**

### 8.3 实际操作演示

**会话 1：项目初始化**

```bash
# 启动新会话，命名
claude --name "项目初始化"

> 帮我初始化 NL2SQL 平台的项目结构：
> - Spring Boot 后端（Maven，Java 17）
> - FastAPI AI 服务（Python，uv 管理依赖）
> - Vue 3 前端
> - 配置好 gitignore、docker-compose、README

AI: [创建项目结构...]
你: [检查、确认、提交]

# 完成后退出
/exit
```

**会话 2：用户认证模块（新会话）**

```bash
# 开新会话
claude --name "用户认证模块"

> 我要做用户认证模块，需求文档在 docs/nl2sql-单库多表版-项目构想.md 的"用户管理"章节。
> 请先读一下，然后帮我实现：
> 1. 用户注册（邮箱 + 密码）
> 2. 用户登录（返回 JWT）
> 3. Token 刷新
> 4. 基于角色的权限校验（ADMIN / USER）
>
> 参考现有的项目结构和代码风格。

AI: [读构想文档、读现有代码、开始实现...]
你: [逐步确认、跑测试、验证]
```

**会话 3：NL2SQL 查询模块（又一个新会话）**

```bash
claude --name "NL2SQL查询模块"

> 我要做 NL2SQL 查询模块，需求在 docs/nl2sql-单库多表版-项目构想.md 的"智能查询"章节。
> 用户认证模块已经做完了（在 com.example.auth 包下），
> 请基于现有的认证体系，实现：
> 1. 自然语言输入接口
> 2. Schema 检索（RAG）
> 3. SQL 生成（LangGraph）
> 4. SQL 安全沙箱
> 5. 查询结果返回

AI: [读文档、读认证代码、实现查询模块...]
```

**会话 6：Bug 修复（针对性会话）**

```bash
claude --name "修复登录500"

> 用户反馈登录偶尔报 500，错误日志如下：
> [粘贴日志]
>
> 请定位原因并修复。只动 AuthService，不要改其他模块。

# 修完确认后，立刻结束会话
```

### 8.4 跨会话如何保持上下文

不同会话之间是隔离的，但有几种方式保持连贯：

**方式 1：CLAUDE.md（最重要）**

每次会话 Claude Code 都会自动读取 CLAUDE.md。把关键信息写进去：

```markdown
# NL2SQL 平台

## 已完成模块
- [x] 项目初始化（Spring Boot + FastAPI + Vue）
- [x] 用户认证（JWT + RBAC，见 com.example.auth）

## 当前进行中
- [ ] NL2SQL 查询模块

## 关键约定
- API 返回格式：ResponseResult<T>
- 异常处理：BusinessException + 错误码
- 数据库：MySQL 8.0，字段 snake_case
```

**方式 2：Memory 系统**

```
# 在会话 2 中
> 记住：用户认证模块的 JWT 密钥配置在 application.yml 的 jwt.secret 字段，
> Token 有效期 2 小时，Refresh Token 有效期 7 天

# 在会话 3 中，Claude Code 自动读取 Memory，知道 JWT 的配置
```

**方式 3：构想文档作为共享上下文**

```
# 每个新会话开始时
> 先读一下 docs/nl2sql-单库多表版-项目构想.md，了解项目整体设计
> 然后帮我实现 XX 模块
```

### 8.5 会话命名规范

```bash
claude --name "模块名-具体任务"

# 示例
claude --name "auth-实现注册登录"
claude --name "query-Schema检索"
claude --name "bugfix-登录500"
claude --name "perf-SQL生成优化"
claude --name "deploy-docker配置"
```

这样 `/resume` 时能快速找到需要的会话。

### 8.6 会话恢复与分支

```bash
# 恢复之前的会话（弹出列表选择）
claude --resume

# 恢复指定会话
claude --resume <session-id>

# 基于旧会话开新分支（不修改原会话）
claude --resume <session-id> --fork-session

# 场景：昨天做了认证模块，今天想加一个功能
# 用 --fork-session 基于昨天的上下文继续，但不污染昨天的会话记录
```

### 8.7 并行开发多个模块

如果两个人同时开发不同模块：

```
开发者 A：claude --name "auth-权限校验"      # 在 feature/auth 分支
开发者 B：claude --name "query-Schema检索"   # 在 feature/query 分支

各自在自己的 git worktree 中工作，互不干扰：
claude --worktree
# 自动创建 .claude/worktrees/ 下的隔离环境
```

---

## 9. 缓存优化：怎么用才省钱

Claude Code 的费用主要来自 token 消耗。理解缓存机制，能省 50% 以上的钱。

### 9.1 缓存是怎么工作的

```
每次你发消息，Claude Code 发送给 API 的内容 = 系统提示 + 对话历史 + 你的消息

系统提示（System Prompt）包含：
  - Claude Code 的内置指令
  - CLAUDE.md 的内容
  - Memory 文件
  - 工具定义

这部分每次都会发送，但 Anthropic 有 Prompt Cache 机制：
  - 相同的前缀内容会被缓存
  - 缓存命中时，缓存部分的费用只有正常价格的 10%
  - 缓存 TTL = 5 分钟（超过 5 分钟没用就失效）
```

### 9.2 省钱的核心原则

```
原则 1：让缓存尽量命中 → 系统提示和 CLAUDE.md 不要频繁改动
原则 2：减少发送的 token 数量 → 对话不要太长
原则 3：简单任务用便宜模型 → haiku 比 opus 便宜 10-20 倍
原则 4：一次说清，减少来回 → 每轮对话都有成本
```

### 9.3 实操优化策略

**策略 1：CLAUDE.md 精简但稳定**

```
# 不好：每次改 CLAUDE.md 都会导致缓存失效
# 不好：CLAUDE.md 太长（超过 200 行），每次都发送大量 token

# 好：CLAUDE.md 只放核心不变的信息
# 好：经常变化的信息放 Memory（不进入系统提示的缓存前缀）
```

**策略 2：用 /compact 而不是 /clear**

```
/compact  → 压缩对话历史，保留关键信息，系统提示缓存还在
/clear    → 清空一切，下次发消息时系统提示需要重新缓存

什么时候用 /compact：对话超过 20 轮，感觉 AI 回复变慢或变贵
什么时候用 /clear：要完全切换任务方向
```

**策略 3：新会话比长会话省钱**

```
一个 100 轮的长会话：
  第 100 轮时，要发送前 99 轮的历史 → 大量 token

10 个 10 轮的短会话：
  每个会话只发送 10 轮历史 → 少量 token
  而且每个新会话的系统提示能命中缓存

结论：做完一个模块就开新会话，不要在一个会话里做所有事
```

**策略 4：简单任务用小模型**

```bash
# 复杂任务（架构设计、复杂 bug）
claude --model opus

# 日常开发（写接口、改代码）
claude --model sonnet

# 简单任务（改个配置、问个问题）
claude --model haiku

# 或者在会话中切换
/model haiku
> 帮我把 application.yml 里的端口改成 8081
/model sonnet
> 帮我实现用户注册接口
```

**策略 5：提示词一次说清**

```
# 不好：来回 5 轮
> 帮我写个接口
> 要 POST 的
> 接收 JSON
> 返回分页结果
> 加个参数校验

# 好：1 轮搞定
> 帮我写一个 POST /api/users 接口：
> - 接收 JSON：{ "name": string, "email": string }
> - 参数校验：name 2-50 字符，email 格式校验
> - 返回 ResponseResult<UserVO>
> - 参考现有的 /api/orders 的写法
```

**策略 6：用 --exclude-dynamic-system-prompt-sections**

```bash
# 这个选项把动态内容（当前目录、环境信息、git 状态）从系统提示中移出
# 让系统提示更稳定，提高缓存命中率
claude --exclude-dynamic-system-prompt-sections
```

**策略 7：设置花费限额**

```bash
# 设置最大花费（非交互模式）
claude -p "运行测试" --max-budget-usd 0.5

# 随时查看当前会话花费
/cost
```

### 9.4 不同场景的推荐配置

| 场景 | 模型 | 会话策略 | 预估花费 |
|------|------|----------|----------|
| 问一个简单问题 | haiku | 单轮 | < $0.01 |
| 改一个配置文件 | haiku | 单轮 | < $0.01 |
| 写一个接口（3-5 个文件）| sonnet | 单会话 | $0.05-0.15 |
| 实现一个模块（10+ 文件）| sonnet | 单会话 | $0.2-0.5 |
| 架构设计 + 方案讨论 | opus | 单会话 | $0.3-1.0 |
| 复杂 bug 排查 | sonnet | 单会话 | $0.1-0.3 |
| 代码审查 | sonnet | 单会话 | $0.05-0.2 |

### 9.5 实际花费参考

```
一天的典型开发：
  会话 1：项目初始化（sonnet）        ~$0.15
  会话 2：实现用户注册（sonnet）      ~$0.10
  会话 3：写单元测试（sonnet）        ~$0.08
  会话 4：改配置（haiku）             ~$0.01
  会话 5：代码审查（sonnet）          ~$0.05
  ─────────────────────────────────
  合计                                ~$0.39/天

如果用 opus 全程：                    ~$2-5/天（贵 5-10 倍）
```

---

## 10. 团队协作

### 10.1 共享项目配置

将以下文件提交到 Git 仓库，团队共享：

```
.claude/
├── settings.json       # 权限、MCP、Hooks 配置
├── skills/             # 自定义 Skills
CLAUDE.md               # 项目说明和规范（最重要）
```

### 10.2 统一代码规范

在 CLAUDE.md 中定义规范，所有团队成员使用 Claude Code 时自动遵循：

```markdown
## 代码规范
- 命名：类名 PascalCase，方法名 camelCase，常量 UPPER_SNAKE_CASE
- 注释：公共 API 必须有 Javadoc，私有方法不需要
- 异常：业务异常用 BusinessException，包含错误码
- 日志：入参出参用 DEBUG，业务异常用 WARN，系统异常用 ERROR
- 测试：覆盖率要求 > 80%，Service 层必须写单元测试
```

### 10.3 Code Review 一致性

```
> 按照 CLAUDE.md 中定义的代码规范审查这个 PR
```

---

## 11. 常见问题与排错

### 11.1 安装与环境

**Q: `claude` 命令找不到？**
```bash
# 检查 npm 全局 bin 目录
npm bin -g
# 确保该目录在 PATH 中
export PATH="$(npm bin -g):$PATH"
```

**Q: Node.js 版本太低？**
```bash
# 使用 nvm 升级
nvm install 20
nvm use 20
```

**Q: API Key 无效？**
```bash
# 检查环境变量
echo $ANTHROPIC_API_KEY
# 重新登录
claude /login
```

### 11.2 使用问题

**Q: Claude Code 似乎"忘记"了之前说的话？**
- 对话太长会自动压缩上下文
- 使用 `/compact` 手动压缩
- 重要信息写入 CLAUDE.md 或 Memory，不会丢失

**Q: 权限弹窗太多？**
- 使用 `/permissions` 添加常用操作到允许列表
- 或在 `.claude/settings.json` 中预配置：

```json
{
  "permissions": {
    "allow": [
      "Bash(npm test*)",
      "Bash(mvn *)",
      "Bash(git *)",
      "Read",
      "Write",
      "Edit"
    ]
  }
}
```

**Q: 修改不符合预期怎么办？**

部分回退技巧（只撤销有问题的修改，保留正确的部分）：

```bash
# 方法一：交互式暂存（推荐）
git add -p                    # 逐块选择要保留的修改
git stash                     # 暂存已选择的部分
git checkout -- .             # 回退剩余的修改
git stash pop                 # 恢复之前暂存的正确部分

# 方法二：先提交再选择性回退
git add -A && git commit -m "wip: save progress"
git revert HEAD               # 回退整个提交
git cherry-pick <commit-id>   # 再把正确的部分单独应用

# 方法三：交互式取消暂存
git reset -p                  # 逐块取消暂存
git checkout -- <file>        # 只回退特定文件
```

预防措施（避免只能全回退）：
1. **分步确认** — 复杂任务要求 Claude Code "先改 A，确认后再改 B"
2. **先创建分支** — `git checkout -b claude/temp` 在隔离环境尝试
3. **阶段性提交** — 每完成一步让 Claude Code 提交一次，保留回退点
4. **明确边界** — 提示词中写清"只修改 Service 层，不要动 Controller"

**Q: 如何减少 token 消耗？**

详见第 9 章"缓存优化"。核心要点：
1. 做完一个模块就开新会话，不要一个会话做所有事
2. 用 `/compact` 而不是 `/clear`
3. 简单任务用 `--model haiku`
4. 提示词一次说清，减少来回
5. 用 `/cost` 随时监控花费

### 11.3 CI/CD 集成

**在 GitHub Actions 中使用：**

```yaml
name: Claude Code Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          review_mode: "security"
```

---

## 12. 最佳实践：社区经验总结

以下是从 Anthropic 官方文档、社区讨论和实际项目中总结出的最佳实践。每条都是实际踩坑后得出的。

### 12.1 CLAUDE.md 怎么写

CLAUDE.md 是你和 Claude Code 之间最重要的"合同"。每次新会话都会自动加载。

**写什么：**

```markdown
# 项目名称

## 一句话描述
NL2SQL 智能数据查询平台，让业务人员用自然语言查数据库

## 构建与运行
- `mvn clean install` — 构建
- `mvn spring-boot:run` — 启动后端
- `cd ai-service && uv run uvicorn main:app` — 启动 AI 服务
- `cd frontend && npm run dev` — 启动前端
- `mvn test` — 运行测试

## 项目结构
- backend/ — Spring Boot 网关（Java 17，Maven）
- ai-service/ — FastAPI AI 服务（Python 3.11，uv）
- frontend/ — Vue 3 前端

## 代码规范
- API 返回统一用 ResponseResult<T>
- 异常用 BusinessException + 错误码
- 日志用 SLF4J，禁止 System.out
- 数据库字段 snake_case，Java 字段 camelCase

## 禁止项
- 禁止 catch(Exception e) 吞异常
- 禁止硬编码配置值
- 禁止在 Controller 里写 SQL
- 禁止修改 src/main/resources/db/migration/ 下已执行的迁移文件

## 已完成模块
- [x] 项目初始化
- [x] 用户认证（JWT + RBAC）
- [ ] NL2SQL 查询（进行中）
- [ ] 审计日志
```

**不要写什么：**

```
# 不要写太多（超过 200 行会被截断）
# 不要写会频繁变化的信息（用 Memory 代替）
# 不要写代码示例（Claude Code 能读源码）
# 不要写通用编程知识（它本来就知道）
```

**CLAUDE.md 的黄金法则：**
1. 精简 — 只放 Claude Code 需要知道但无法从代码中推断的信息
2. 稳定 — 不经常变的内容才放这里
3. 可执行 — 写"禁止 XXX"比写"建议不要 XXX"更有效
4. 放在最前面 — 最重要的信息放前面（缓存命中时这部分不变）

### 12.2 任务拆解策略

**大任务必须拆，但不要拆太碎。**

```
# 太大（AI 一次改 50 个文件，质量下降）
> 帮我实现整个后端

# 太碎（每步都要等 AI 重新读代码，浪费 token）
> 帮我创建 User.java
> 帮我创建 UserDTO.java
> 帮我创建 UserRepository.java

# 刚好（一个会话能完成的完整功能）
> 帮我实现用户注册接口：
> - Controller: POST /api/users/register
> - Service: 校验邮箱唯一性，密码加密存储
> - 返回 ResponseResult<UserVO>
> - 写单元测试
> - 参考现有的 /api/auth/login 的代码风格
```

**拆解原则：**
- 一个任务 = 一个会话能完成
- 一个任务 = 一个独立的功能点（可单独测试）
- 一个任务涉及的文件数控制在 3-10 个

### 12.3 测试驱动开发（TDD）

让 Claude Code 先写测试，再写实现。这样你能验证它的输出是否正确。

```
# 不好：先写实现，测试随便写
> 帮我实现 UserService，然后写点测试

# 好：先写测试定义行为，再写实现
> 帮我给 UserService 写单元测试，覆盖以下场景：
> 1. 正常注册 → 返回用户信息
> 2. 邮箱已存在 → 抛 BusinessException(USER_ALREADY_EXISTS)
> 3. 密码少于 8 位 → 抛 BusinessException(PASSWORD_TOO_SHORT)
>
> 先写测试，写完我看看，确认后再写实现
```

**验证流程：**
```
AI 写完测试 → 你审查测试用例 → 确认覆盖了所有场景
AI 写完实现 → 跑测试 → 全过 → 提交
```

### 12.4 上下文管理

**什么时候该做什么：**

| 场景 | 操作 | 原因 |
|------|------|------|
| 对话超过 20 轮 | `/compact` | 压缩历史，释放上下文空间 |
| 切换到完全不同的任务 | 开新会话 | 避免无关上下文干扰 |
| AI 开始"忘记"之前说的 | `/compact` | 压缩后关键信息保留 |
| 要做下一个模块 | 开新会话 | 保持上下文干净 |
| 对话跑偏了 | `/rewind` 回退到正确节点 | 不用 `/clear` 重头来 |

**CLAUDE.md vs Memory vs 对话上下文：**

```
CLAUDE.md（每次会话都加载）：
  → 项目架构、代码规范、构建命令、禁止项
  → 放"永远需要"的信息

Memory（跨会话持久化）：
  → "JWT 密钥在 .env 里"、"数据库密码不要提交"
  → 放"会变但需要记住"的信息

对话上下文（当前会话有效）：
  → "刚才那个 UserService"、"沿用之前的写法"
  → 放"只在当前任务有用"的信息
```

### 12.5 提示词工程

**原则 1：说清楚你要什么，而不是不要什么**

```
# 不好
> 不要写太复杂的代码

# 好
> 代码保持简单，每个方法不超过 30 行，用清晰的命名而不是注释来解释意图
```

**原则 2：给参考而不是给规范**

```
# 不好（太抽象）
> 遵循项目的代码规范

# 好（具体参考）
> 参考 src/main/java/.../auth/AuthController.java 的写法
> 用同样的异常处理方式、返回格式、注解风格
```

**原则 3：约束范围**

```
# 不好（范围不明确）
> 帮我优化这个模块

# 好（明确边界）
> 只修改 UserService.java 和 UserRepository.java
> 不要动 Controller 层
> 不要改数据库表结构
> 目标：查询性能从 2s 降到 200ms 以内
```

**原则 4：分步确认**

```
# 不好（一步到位，出错难回退）
> 帮我把认证模块从 Session 改成 JWT，一步完成

# 好（分步确认）
> 第一步：先帮我设计 JWT 的方案（密钥存储、Token 结构、刷新机制）
> 我确认后再动手改代码
```

### 12.6 Git 工作流

**让 Claude Code 帮你提交，但你来决定什么时候提交。**

```
# 不好：让 AI 改完所有东西再提交
> 帮我实现用户认证，做完了一起提交

# 好：每完成一步就提交
> 帮我实现用户注册接口
AI 完成后：
> 提交一下，message 用 feat: add user registration API
> 然后继续做登录接口
AI 完成后：
> 提交，message 用 feat: add user login API
```

**为什么分步提交：**
- 出问题能精确定位是哪一步引入的
- 能用 `git revert` 回退单个功能
- PR review 时每个 commit 职责清晰

**分支策略：**
```bash
# 开发新功能时，先创建分支
git checkout -b feature/user-auth

# 让 Claude Code 在这个分支上工作
> 帮我实现用户注册接口

# 完成后提交到分支
> 提交

# 最后合并到 main
git checkout main && git merge feature/user-auth
```

### 12.7 安全注意事项

**不要让 Claude Code 接触敏感信息：**

```
# 不好：把密钥放在 CLAUDE.md 里
# CLAUDE.md 中写 "API Key: sk-ant-xxx"

# 好：告诉 Claude Code 密钥在哪，但不暴露内容
# CLAUDE.md 中写 "API Key 在 .env 文件中，不要提交到 git"
```

**权限控制：**
```json
// .claude/settings.json
{
  "permissions": {
    "allow": [
      "Bash(mvn *)",
      "Bash(npm *)",
      "Bash(git *)",
      "Read",
      "Write",
      "Edit"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(git push --force *)"
    ]
  }
}
```

### 12.8 常见工作模式

**模式 1：探索 → 计划 → 实现**

```
第 1 轮：> 帮我梳理这个项目的认证流程（探索）
第 2 轮：> 基于梳理结果，设计一个 OAuth2 集成方案（计划）
第 3 轮：> 方案确认了，开始实现（实现）
```

**模式 2：读代码 → 改代码 → 跑测试**

```
> 读一下 src/.../OrderService.java，理解它的逻辑
> 在 calculateTotal 方法里加上折扣计算逻辑
> 跑一下 OrderServiceTest，确保没破坏现有功能
```

**模式 3：报错 → 诊断 → 修复**

```
> 启动报错了，日志如下：[粘贴]
AI: 原因是 XXX，需要修改 YYY
> 改吧
AI: 改完了
> 重新启动验证一下
```

**模式 4：需求文档 → 拆任务 → 逐个实现**

```
> 读一下 docs/nl2sql-单库多表版-项目构想.md 的"用户管理"章节
> 基于需求，帮我拆成 3-5 个开发任务
> 先做第 1 个
```

### 12.9 效率对比：好的用法 vs 差的用法

| 维度 | 差的用法 | 好的用法 |
|------|----------|----------|
| 任务描述 | "帮我写个接口" | "POST /api/users，接收 name+email，校验后存库，返回 UserVO" |
| 上下文 | 每次都重新解释项目 | CLAUDE.md 写好，直接开干 |
| 会话管理 | 一个会话做所有事 | 一个模块一个会话 |
| 验证 | AI 说完成就信了 | 跑测试、看 diff、手动验证 |
| 提交 | 改完一起提交 | 每步提交，message 清晰 |
| 模型选择 | 全程 opus | 简单任务用 haiku |
| 出错处理 | /clear 重来 | /rewind 回退到正确节点 |
| 成本意识 | 不看 /cost | 随时监控，设限额 |

---

## 附录：速查卡片

### 快捷键

| 按键 | 功能 |
|------|------|
| `Esc` | 中断当前生成 |
| `Ctrl+C` | 退出 |
| `Ctrl+L` | 清屏 |
| `↑` | 上一条历史命令 |
| `Tab` | 自动补全文件路径 |
| `Shift+Enter` | 换行（需先运行 `/terminal-setup`） |

### 斜杠命令速查

```
会话管理：  /clear  /compact  /resume  /rewind
配置认证：  /config  /login  /logout  /model  /permissions  /vim
开发工具：  /init  /review  /security-review  /simplify  /fast  /mcp  /terminal-setup
信息查看：  /help  /cost
```

### 多模块会话管理速查

```bash
# 开新会话（命名）
claude --name "模块名-任务描述"

# 恢复旧会话
claude --resume                    # 弹出列表
claude --resume <session-id>       # 指定会话

# 基于旧会话开分支
claude --resume <id> --fork-session

# 隔离环境开发
claude --worktree
```

### 省钱速查

```bash
# 选模型
claude --model haiku               # 简单任务（最便宜）
claude --model sonnet              # 日常开发（推荐）
claude --model opus                # 复杂设计（最贵）

# 控制花费
/cost                              # 查看当前花费
--max-budget-usd 0.5               # 设置花费上限

# 优化缓存
/compact                           # 压缩对话（不清空，保留缓存）
--exclude-dynamic-system-prompt-sections  # 提高缓存命中率

# 核心原则
# 1. 一个模块一个会话
# 2. 用 /compact 不用 /clear
# 3. 简单任务用 haiku
# 4. 一次说清减少来回
```

### 项目文件结构

```
your-project/
├── CLAUDE.md                    # 项目级指令（自动加载，最重要）
├── .claude/
│   ├── settings.json            # 权限、MCP、Hooks
│   ├── settings.local.json      # 个人配置（不提交）
│   ├── skills/                  # 自定义 Skills
│   └── memory/                  # Memory 文件
├── .env                         # 环境变量（不提交）
└── src/                         # 源代码
```

### 配置优先级

```
~/.claude/settings.json          # 全局（所有项目）
  ↓ 覆盖
.claude/settings.json            # 项目级（团队共享）
  ↓ 覆盖
.claude/settings.local.json      # 项目本地（个人）
  ↓ 覆盖
CLAUDE.md                        # 项目指令
  ↓ 覆盖
对话中的指令                      # 临时
```

---

> **项目地址：** https://github.com/anthropics/claude-code
>
> **一句话总结：** 一个模块一个会话，CLAUDE.md 放不变的信息，Memory 放会变的信息，简单任务用 haiku，用 /compact 保持缓存。
