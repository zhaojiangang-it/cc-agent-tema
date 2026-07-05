# Agent Teams 实战手册：从零开发一个 Todo List

> 本文档用最详细的步骤告诉你：打开 Claude Code 后，每一步该做什么、输入什么、会看到什么。
> 以开发一个「Todo List（待办事项）」功能为例。

---

## 目录

1. [一次性准备工作（只做一次）](#1-一次性准备工作只做一次)
2. [开始一个新项目](#2-开始一个新项目)
3. [方式一：自然语言对话（最简单）](#3-方式一自然语言对话最简单)
4. [方式二：斜杠命令 /sprint（推荐）](#4-方式二斜杠命令-sprint推荐)
5. [方式三：Agent Team 分屏模式（高级）](#5-方式三agent-team-分屏模式高级)
6. [过程中怎么和 Agent 交互](#6-过程中怎么和-agent-交互)
7. [查看和管理任务进度](#7-查看和管理任务进度)
8. [完成后的收尾工作](#8-完成后的收尾工作)
9. [常见场景速查](#9-常见场景速查)
10. [问题排查](#10-问题排查)

---

## 1. 一次性准备工作（只做一次）

### 1.1 确认 Claude Code 已安装

```bash
# 打开终端，输入：
claude --version

# 如果没安装：
npm install -g @anthropic-ai/claude-code

# 确保版本是最新的
npm update -g @anthropic-ai/claude-code
```

### 1.2 启用 Agent Team 功能

Agent Team 是实验性功能，需要手动开启。

```bash
# 方法：编辑配置文件
vim ~/.claude/settings.json
```

在文件中添加（如果已有其他配置，合并进去）：

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

保存退出。

### 1.3 安装 tmux（分屏模式需要）

```bash
# macOS
brew install tmux

# Ubuntu / Debian
sudo apt install tmux

# 验证安装
tmux -V
# 应该输出版本号，如 tmux 3.4
```

### 1.4 确认 Agent 定义文件已就位

```bash
# 检查项目级 Agent 是否存在
ls .claude/agents/
# 应该看到：backend-dev.md  code-reviewer.md  devops.md  frontend-dev.md  product-manager.md  ui-designer.md

# 如果不存在，把本项目 .claude/ 目录复制过来
# 或者检查用户级 Agent
ls ~/.claude/agents/
```

### 1.5 确认 Skills 已安装

```bash
# 检查 Superpowers 插件
ls ~/.claude/plugins/cache/claude-plugins-official/superpowers/

# 检查其他 Skills
ls ~/.claude/skills/
# 应该看到：bailian-cli  better-auth-security-best-practices  docker-deployment  dws  fastapi-python  find-skills  myself  skill-creator  vue  web-design-guidelines
```

✅ **以上全部确认后，一次性准备就完成了。以后不需要再做。**

---

## 2. 开始一个新项目

### 2.1 创建项目目录并初始化

```bash
# 创建项目
mkdir ~/code/my-todo-app
cd ~/code/my-todo-app

# 初始化 Git（必须，Agent Team 依赖 Git）
git init

# 初始化 Node.js 项目（或你的技术栈）
npm init -y

# 创建 .claude 目录结构
mkdir -p .claude/agents .claude/commands
```

### 2.2 复制 Agent 定义到项目中

```bash
# 把已有的 Agent 定义复制过来
cp /Users/rose/code/cc-agent-tema/.claude/agents/*.md .claude/agents/
cp /Users/rose/code/cc-agent-tema/.claude/commands/*.md .claude/commands/
cp /Users/rose/code/cc-agent-tema/CLAUDE.md .
```

### 2.3 修改 CLAUDE.md 适配当前项目

```bash
vim CLAUDE.md
```

根据你的项目修改「技术栈」部分。比如 Todo List 项目用 Vue + FastAPI：

```markdown
## 技术栈（本模板默认，可按项目调整）

- 前端：Vue 3 + TypeScript + Element Plus
- 后端：Python FastAPI
- 数据库：SQLite（开发）/ PostgreSQL（生产）
- 部署：Docker + 阿里云 ECS + Nginx
```

### 2.4 初始提交

```bash
git add .
git commit -m "chore: 初始化项目和 Agent Team 配置"
```

---

## 3. 方式一：自然语言对话（最简单）

这是最直观的使用方式——直接用中文告诉 Claude 你要做什么。

### 步骤 1：启动 Claude Code

```bash
cd ~/code/my-todo-app
claude
```

你会看到 Claude Code 的交互界面。

### 步骤 2：直接描述你的需求

在 Claude Code 的输入框中，输入：

```
帮我开发一个 Todo List（待办事项）功能。
要求：
- 前端用 Vue 3 + Element Plus
- 后端用 Python FastAPI
- 支持增删改查、标记完成
- 数据存 SQLite
```

### 步骤 3：Claude 会自动编排

Claude 会读取你的 CLAUDE.md 和 Agent 定义，然后自动：

1. 调用 **product-manager** 分析需求、写用户故事
2. 调用 **ui-designer** 设计界面方案
3. 调用 **backend-dev** 设计 API 和数据库
4. 调用 **frontend-dev** 实现前端组件
5. 调用 **devops** 配置部署
6. 调用 **code-reviewer** 审查代码

你会看到 Claude 的输出中不断切换角色，每个角色完成自己的工作后把结果传递给下一个。

### 步骤 4：过程中你可以随时插入指令

当某个 Agent 在工作时，你可以直接输入反馈：

```
后端的 API 设计不需要分页，这个功能很简单，直接返回全部就行
```

```
前端不要用 Element Plus 的 Table 组件，用卡片列表的方式展示
```

```
这个设计可以了，继续下一步
```

### 步骤 5：查看最终产出

所有 Agent 完成后，你会看到汇总报告。此时可以检查生成的文件：

```bash
# 在另一个终端窗口
ls src/
# 应该看到前端和后端代码
```

---

## 4. 方式二：斜杠命令 /sprint（推荐）

这是最规范的方式——使用预定义的 Sprint 流程，自动按 Superpowers 工作流执行。

### 步骤 1：启动 Claude Code

```bash
cd ~/code/my-todo-app
claude
```

### 步骤 2：输入斜杠命令

```
/sprint Todo List 待办事项功能，支持增删改查和标记完成，前端 Vue 3，后端 FastAPI，数据库 SQLite
```

> 💡 `/sprint` 后面跟的就是你的需求描述，越详细越好。

### 步骤 3：观察自动执行流程

Claude 会按照 `sprint.md` 中定义的 6 个阶段自动执行：

```
你会看到的执行过程：

阶段 0：预检
├── 创建 Git worktree 隔离工作空间
├── 运行基线测试（如果有的话）
└── 确认从干净状态开始

阶段 1：需求分析（brainstorming）
├── 调用 product-manager Agent
├── 输出 PRD 文档到 docs/specs/
└── 包含用户故事和验收标准

阶段 2：设计与规划（writing-plans）
├── 调用 ui-designer Agent → 界面方案
├── 调用 backend-dev Agent → API 设计 + 数据库 Schema
└── 生成分任务实现计划

阶段 3：并行开发（subagent-driven-development）
├── 调用 frontend-dev → Vue 组件（遵循 TDD）
├── 调用 backend-dev → FastAPI 端点（遵循 TDD）
├── 调用 devops → Docker 配置
└── 每个任务完成后 code-reviewer 审查

阶段 4：质量门禁（verification-before-completion）
├── 运行完整测试套件
├── code-reviewer 全分支审查
└── 独立验证所有产出

阶段 5：完成（finishing-a-development-branch）
├── 确认测试全部通过
└── 呈现选项：合并 / PR / 保留 / 丢弃
```

### 步骤 4：在阶段 5 做选择

当所有工作完成后，Claude 会问你：

```
所有任务已完成，测试全部通过。请选择集成方式：

1. 本地合并到主分支（main）
2. 推送并创建 Pull Request
3. 保留当前分支继续迭代
4. 丢弃所有变更
```

输入数字选择即可。

---

## 5. 方式三：Agent Team 分屏模式（高级）

这个模式下，多个 Claude 会话在 tmux 窗口中并行运行，你可以实时看到每个 Agent 在做什么。

### 步骤 1：以分屏模式启动

```bash
cd ~/code/my-todo-app
claude --teammate-mode tmux
```

或者如果你已经在 `settings.json` 里配置了 `"teammateMode": "tmux"`，直接：

```bash
claude
```

### 步骤 2：描述需求并请求组建团队

```
帮我组建一个 Agent 团队来开发 Todo List 功能：
- product-manager 负责需求和验收标准
- ui-designer 负责界面设计
- frontend-dev 负责 Vue 3 前端实现
- backend-dev 负责 FastAPI 后端实现
- devops 负责 Docker 配置
- code-reviewer 负责最终代码审查

请并行开始工作。
```

### 步骤 3：观察 tmux 分屏

你的终端会被分成多个窗格：

```
┌─────────────────────────────────────────────┐
│ 窗格 1：主编排器（你的主会话）               │
│ > 正在协调各 Agent...                        │
├──────────────────────┬──────────────────────┤
│ 窗格 2：product-     │ 窗格 3：ui-designer  │
│ manager              │                      │
│ > 正在编写 PRD...    │ > 正在设计界面...     │
├──────────────────────┼──────────────────────┤
│ 窗格 4：backend-dev  │ 窗格 5：frontend-dev │
│ > 正在设计 API...    │ > 正在实现组件...     │
└──────────────────────┴──────────────────────┘
```

你可以用 tmux 快捷键在不同窗格之间切换查看：
- `Ctrl+B` 然后 `方向键` → 切换窗格
- `Ctrl+B` 然后 `z` → 当前窗格全屏/恢复

### 步骤 4：在主窗格中监控进度

主窗格（编排器）会汇总各 Agent 的状态：

```
✅ product-manager：PRD 已完成（3 个用户故事）
✅ ui-designer：界面方案已确定（5 个组件）
🔄 backend-dev：正在实现 API 端点（3/5）
🔄 frontend-dev：正在实现组件（2/5）
⏳ devops：等待开发完成后配置
⏳ code-reviewer：等待所有开发完成后审查
```

---

## 6. 过程中怎么和 Agent 交互

### 6.1 直接点名调用某个角色

```
让 backend-dev 看一下现在数据库设计有没有问题
```

```
让 code-reviewer 审查一下 backend-dev 刚写的代码
```

```
让 devops 给这个项目写一个 docker-compose.yml
```

### 6.2 对当前工作给反馈

```
这个 API 设计太复杂了，不需要分页和过滤，简单返回全部就行
```

```
前端的卡片样式换成列表样式吧
```

```
数据库字段 created_at 改成 created_time，我习惯这个命名
```

### 6.3 要求并行处理

```
让 frontend-dev 和 backend-dev 同时开始开发，
前端先 mock 数据，后端独立实现 API，
最后联调。
```

### 6.4 跳过某个阶段

```
不需要产品经理写 PRD 了，需求已经很清楚了，直接开始开发
```

### 6.5 要求使用特定 Skill

```
用 brainstorming skill 帮我梳理一下这个需求
```

```
用 systematic-debugging 的方式帮我排查这个 bug
```

---

## 7. 查看和管理任务进度

### 7.1 查看当前所有任务

```
列出当前所有任务和它们的状态
```

Claude 会展示任务列表：

```
任务列表：
1. ✅ [已完成] 编写 PRD 和用户故事
2. ✅ [已完成] 设计 UI 组件方案
3. ✅ [已完成] 设计 API 端点和数据库 Schema
4. 🔄 [进行中] 实现后端 CRUD API
5. ⏳ [待开始] 实现前端 TodoList 组件
6. ⏳ [待开始] 配置 Docker Compose
7. ⏳ [待开始] 代码审查
```

### 7.2 查看某个 Agent 的产出

```
product-manager 写了什么？给我看看 PRD
```

```
backend-dev 设计的 API 是什么样的？
```

### 7.3 手动调整任务优先级

```
先做后端 API，前端等 API 好了再做
```

---

## 8. 完成后的收尾工作

### 8.1 检查生成的代码

```bash
# 退出 Claude Code 后，在项目目录查看
ls -la src/
ls -la docs/
cat docs/specs/todo-list-prd.md
```

### 8.2 运行测试

```bash
# 后端测试
cd backend && python -m pytest

# 前端测试
cd frontend && npm run test
```

### 8.3 启动项目试运行

```bash
# 后端
cd backend && uvicorn main:app --reload

# 前端（另一个终端）
cd frontend && npm run dev
```

### 8.4 提交代码

```bash
git add .
git commit -m "feat: 完成 Todo List 功能"
git push
```

---

## 9. 常见场景速查

### 场景 A：开发一个新功能

```
# 最简单
帮我开发 [功能描述]

# 最规范
/sprint [功能描述]

# 指定角色
让 product-manager 先分析需求，然后 backend-dev 和 frontend-dev 并行开发
```

### 场景 B：修复一个 Bug

```
# 自动系统化调试
有个 Bug：[描述 Bug 现象]。请用 systematic-debugging 的方式排查。

# 指定角色
让 backend-dev 排查这个 API 返回 500 的问题
```

### 场景 C：代码审查

```
# 审查最近的变更
让 code-reviewer 审查一下最近的代码变更

# 审查特定文件
让 code-reviewer 审查 src/api/auth.py 的安全性
```

### 场景 D：优化性能

```
让 backend-dev 分析数据库查询性能，找出 N+1 问题并优化
```

### 场景 E：写文档

```
让 product-manager 把当前的功能写成用户手册
```

### 场景 F：部署上线

```
让 devops 配置好 Docker 和 CI/CD，准备部署到阿里云 ECS
```

### 场景 G：安全审计

```
让 code-reviewer 做一次完整的安全审查，
重点关注认证、输入校验、SQL 注入
```

### 场景 H：调用阿里云百炼

```
让 devops 用 bl 命令搜索一下 FastAPI 连接 PostgreSQL 的最佳实践
```

---

## 10. 问题排查

### Q：Agent 没有被触发，Claude 自己干了

**原因**：Claude 认为任务足够简单，不需要调度子 Agent。

**解决**：在请求中明确指定角色名：
```
让 backend-dev 来实现这个 API（不要自己做）
```

### Q：自定义的 Agent 定义没生效

**原因**：Agent 定义在会话启动时加载，中途新增的文件不会被读取。

**解决**：退出 Claude Code，重新启动：
```bash
# Ctrl+C 退出
claude
```

### Q：tmux 分屏显示异常

**原因**：终端不支持或 tmux 版本太旧。

**解决**：
```bash
# 更新 tmux
brew upgrade tmux  # macOS

# 换用 auto 模式
claude --teammate-mode auto

# 或者不用分屏，直接普通模式
claude
```

### Q：多个 Agent 修改了同一个文件导致冲突

**原因**：Agent Team 不会自动隔离文件。

**解决**：
```
让 frontend-dev 只改 src/frontend/ 目录，
backend-dev 只改 src/backend/ 目录，
不要碰对方的文件。
```

### Q：Agent 输出了英文

**原因**：CLAUDE.md 中没有强制中文。

**解决**：在 CLAUDE.md 中添加：
```markdown
## 语言要求
所有输出、代码注释、文档必须使用中文。
```

### Q：怎么知道当前有哪些 Agent 可用？

```
列出当前项目所有可用的 subagent 和它们的能力
```

或者直接查看：
```bash
ls .claude/agents/
# 每个 .md 文件就是一个可用的 Agent
```

### Q：怎么在已有项目中引入 Agent Team？

```bash
cd 你的项目目录

# 1. 创建目录
mkdir -p .claude/agents .claude/commands

# 2. 复制 Agent 定义
cp /Users/rose/code/cc-agent-tema/.claude/agents/*.md .claude/agents/
cp /Users/rose/code/cc-agent-tema/.claude/commands/*.md .claude/commands/
cp /Users/rose/code/cc-agent-tema/CLAUDE.md .

# 3. 修改 CLAUDE.md 适配你的项目
vim CLAUDE.md

# 4. 提交
git add .claude/ CLAUDE.md
git commit -m "chore: 引入 Agent Team 配置"

# 5. 启动
claude
```

---

## 快速参考卡片

```
╔══════════════════════════════════════════════════════════════╗
║                    Agent Teams 速查                          ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  启动：                                                      ║
║    claude                          # 普通模式                ║
║    claude --teammate-mode tmux     # 分屏模式                ║
║                                                              ║
║  调用 Agent：                                                ║
║    "让 backend-dev 设计 API"                                  ║
║    "让 code-reviewer 审查代码"                                ║
║    "让 frontend-dev 和 backend-dev 并行开发"                  ║
║                                                              ║
║  斜杠命令：                                                  ║
║    /sprint 功能描述                # 全栈 Sprint 开发        ║
║    /build-team                   # 查看/组建团队            ║
║                                                              ║
║  Superpowers 技能：                                          ║
║    "用 brainstorming 帮我梳理需求"                            ║
║    "用 systematic-debugging 排查这个 bug"                    ║
║    "用 test-driven-development 方式开发"                     ║
║                                                              ║
║  管理任务：                                                  ║
║    "列出当前所有任务"                                        ║
║    "backend-dev 的进度怎么样"                                ║
║                                                              ║
║  百炼 AI 能力：                                              ║
║    "让 devops 用 bl 搜索最新技术方案"                         ║
║    "用 bl image generate 生成一个 logo"                       ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```
