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
- [8. 团队协作](#8-团队协作)
- [9. 提示词技巧与最佳实践](#9-提示词技巧与最佳实践)
- [10. 常见问题与排错](#10-常见问题与排错)

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

# 继续上次对话
claude --continue

# 恢复指定对话
claude --resume <session-id>
```

---

## 2. 核心交互模式

### 2.1 斜杠命令速查

| 命令 | 用途 |
|------|------|
| `/help` | 查看帮助 |
| `/clear` | 清除当前对话上下文 |
| `/compact` | 压缩对话历史，释放上下文空间 |
| `/config` | 查看/修改配置 |
| `/cost` | 查看当前会话的 token 消耗 |
| `/doctor` | 诊断环境问题 |
| `/init` | 初始化 CLAUDE.md 文件 |
| `/login` | 切换账号 |
| `/logout` | 登出 |
| `/model` | 切换模型 |
| `/permissions` | 查看/管理权限设置 |
| `/review` | 审查 PR |
| `/security-review` | 安全审查 |
| `/simplify` | 审查并简化已修改的代码 |
| `/fast` | 切换快速模式（Opus 加速输出） |

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

## 8. 团队协作

### 8.1 共享项目配置

将以下文件提交到 Git 仓库，团队共享：

```
.claude/
├── settings.json       # 权限、MCP、Hooks 配置
├── skills/             # 自定义 Skills
│   ├── gen-api.md
│   └── review-style.md
└── shared-memory.md    # 团队共享记忆
CLAUDE.md               # 项目说明和规范
```

### 8.2 统一代码规范

在 CLAUDE.md 中定义规范，所有团队成员使用 Claude Code 时自动遵循：

```markdown
## 代码规范
- 命名：类名 PascalCase，方法名 camelCase，常量 UPPER_SNAKE_CASE
- 注释：公共 API 必须有 Javadoc，私有方法不需要
- 异常：业务异常用 BusinessException，包含错误码
- 日志：入参出参用 DEBUG，业务异常用 WARN，系统异常用 ERROR
- 测试：覆盖率要求 > 80%，Service 层必须写单元测试
```

### 8.3 Code Review 一致性

```
> 按照 CLAUDE.md 中定义的代码规范审查这个 PR
```

### 8.4 知识传递

新成员加入时：
```
> 帮我梳理这个项目的架构，重点说明：
> - 模块划分和职责
> - 核心业务流程
> - 关键设计决策的原因
> - 开发环境搭建步骤
```

---

## 9. 提示词技巧与最佳实践

### 9.1 提示词原则

**具体优于模糊：**

```
# 不好
> 帮我优化这个方法

# 好
> 这个方法在大数据量时很慢（10万条记录需要 8 秒），
> 帮我优化查询性能，目标是降到 1 秒以内。
> 可以考虑分页、索引、或缓存策略。
```

**提供上下文：**

```
# 不好
> 帮我写个接口

# 好
> 帮我写一个批量导入用户的接口：
> - POST /api/users/import
> - 接收 CSV 文件
> - 异步处理，返回任务 ID
> - 通过 WebSocket 推送进度
> - 参考现有的 /api/orders/import 的实现模式
```

**指定约束：**

```
> 帮我重构这个类，要求：
> - 保持向后兼容，不改 public 方法签名
> - 使用策略模式替代 if-else
> - 不引入新的外部依赖
```

### 9.2 高效对话策略

**逐步细化：**
```
第 1 轮：> 帮我设计一个限流模块的接口
第 2 轮：> 基于这个接口，用令牌桶算法实现
第 3 轮：> 加上 Redis 支持分布式限流
第 4 轮：> 写完整的单元测试
```

**利用上下文：**
```
> 刚才那个 UserService，再加一个批量查询的方法
> 沿用之前的返回格式和异常处理方式
```

**纠正方向：**
```
> 不对，我要的不是缓存方案，而是用数据库乐观锁来处理并发
> 请重新实现
```

### 9.3 性能优化技巧

| 技巧 | 说明 |
|------|------|
| `/compact` | 对话太长时压缩上下文，减少 token 消耗 |
| `--model haiku` | 简单任务用小模型，省钱快速 |
| 明确文件路径 | "看 src/.../UserService.java" 比 "看 UserService" 搜索更快 |
| 一次说清 | 减少来回轮数，节省 token 和时间 |
| `/cost` | 随时查看消耗，控制成本 |

### 9.4 避免的常见错误

1. **一次性改太多** — 分步骤来，每步确认
2. **不给上下文** — Claude Code 不知道你脑中的需求细节
3. **忽略 Plan 模式** — 复杂任务先对齐方案再动手
4. **不看 diff** — 始终检查 Claude Code 的修改是否符合预期
5. **不利用 CLAUDE.md** — 项目规范写一次，永久生效

---

## 10. 常见问题与排错

### 10.1 安装与环境

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

### 10.2 使用问题

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
- 对话中说"不对，还原刚才的修改"
- 或使用 `git checkout -- <file>` 还原
- Claude Code 的所有修改都可以通过 Git 回退

**Q: 如何减少 token 消耗？**
1. 简单问题用 `/fast` 切换到快速模式
2. 对话太长时 `/compact` 压缩
3. 不需要大模型的任务指定 `--model haiku`
4. 提示词尽量一次说清，减少来回

### 10.3 CI/CD 集成

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

**在本地脚本中使用：**

```bash
#!/bin/bash
# 运行测试并生成报告
claude -p "运行 mvn test，如果有失败的测试，分析原因并给出修复建议" \
  --output-file test-report.md
```

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

### 项目文件结构

```
your-project/
├── CLAUDE.md                    # 项目级指令（自动加载）
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

> **持续更新：** Claude Code 迭代很快，建议关注 [官方文档](https://docs.anthropic.com/claude-code) 和 [GitHub](https://github.com/anthropics/claude-code) 获取最新功能。
