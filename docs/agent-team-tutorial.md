# Claude Code Agent Team 使用教程

> 从零开始，手把手教你启用和使用 Claude Code 的多 Agent 协作功能。

---

## 目录

1. [前置条件](#1-前置条件)
2. [启用 Agent Team](#2-启用-agent-team)
3. [配置 Subagent（子代理）](#3-配置-subagent子代理)
4. [使用方式详解](#4-使用方式详解)
5. [Agent Team 分屏模式](#5-agent-team-分屏模式)
6. [斜杠命令](#6-斜杠命令)
7. [Workflow 工作流编排](#7-workflow-工作流编排)
8. [实战案例](#8-实战案例)
9. [常见问题](#9-常见问题)

---

## 1. 前置条件

### 安装 Claude Code

```bash
# 确保已安装 Node.js 18+
node -v

# 安装 Claude Code
npm install -g @anthropic-ai/claude-code

# 验证安装
claude --version
```

### 登录认证

```bash
# 首次使用需要登录
claude

# 按提示完成 Anthropic 账号认证
```

### 检查当前版本

```bash
# Agent Team 功能需要较新版本
claude --version
# 建议升级到最新版
npm update -g @anthropic-ai/claude-code
```

---

## 2. 启用 Agent Team

Agent Team 目前是**实验性功能**，默认关闭。需要手动启用。

### 方法一：通过配置文件永久启用（推荐）

```bash
# 打开配置文件
# macOS / Linux
vim ~/.claude/settings.json

# Windows
# 编辑 %APPDATA%\claude\settings.json
```

添加以下内容：

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

保存后，每次启动 Claude Code 都会自动启用 Agent Team 功能。

### 方法二：通过环境变量临时启用

```bash
# 单次启动时启用
CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 claude
```

### 验证是否启用成功

启动 Claude Code 后，输入：

```
你有 Agent Team 能力吗？可以创建团队吗？
```

如果回复中提到了 **team orchestration** 或 **teammate** 相关能力，说明已成功启用。

---

## 3. 配置 Subagent（子代理）

Subagent 是你自定义的 AI 角色。每个角色定义为一个 Markdown 文件。

### 目录结构

```
你的项目/
├── CLAUDE.md                    # 项目指令（所有 Agent 共享）
└── .claude/
    ├── agents/                  # 子代理定义目录
    │   ├── product-manager.md   # 产品经理
    │   ├── ui-designer.md       # UI 设计师
    │   ├── frontend-dev.md      # 前端开发
    │   ├── backend-dev.md       # 后端开发
    │   ├── devops.md            # 运维工程师
    │   └── code-reviewer.md     # 代码审查
    └── commands/                # 斜杠命令目录
        ├── sprint.md            # Sprint 开发命令
        └── build-team.md        # 组建团队命令
```

### Subagent 文件格式

每个 `.md` 文件由两部分组成：**YAML 头部** + **系统提示词正文**。

```markdown
---
name: agent-name               # 唯一标识（调用时使用）
description: >                  # 描述（Claude 据此自动路由）
  一句话说明这个 Agent 的职责
vibe: "一句话性格标签"           # 可选，快速了解 Agent 风格
tools: Read, Grep, Glob, ...   # 允许使用的工具
disallowedTools: Write, Edit   # 可选，禁止的工具（审查类 Agent 用）
model: opus                     # 使用的模型（haiku/sonnet/opus）
level: 3                        # 级别（2=执行 3=专家 4=战略）
---

# 系统提示词正文

这里是 Agent 的详细角色定义、工作流程、输出规范等。
```

### 三级作用域

| 目录 | 作用域 | 说明 |
|------|--------|------|
| `~/.claude/agents/` | 用户级 | 所有项目共享 |
| `项目/.claude/agents/` | 项目级 | 仅当前项目 |
| 组织托管目录 | 组织级 | 管理员统一部署 |

### 模型选择指南

| 模型 | 适用角色 | 特点 |
|------|---------|------|
| `haiku` | 搜索、文档 | 快速、便宜 |
| `sonnet` | 前端、后端、运维、UI | 平衡性价比和质量 |
| `opus` | 代码审查、架构分析 | 最高质量，适合需要深度推理的任务 |

---

## 4. 使用方式详解

### 方式一：自然语言调用（最简单）

直接在 Claude Code 会话中用自然语言指定角色：

```
你：让 backend-dev 帮我设计用户登录的 API
```

Claude 会自动匹配 `backend-dev` 子代理来处理这个请求。

```
你：让 code-reviewer 审查一下我刚才改的代码
```

```
你：让 product-manager 写一个购物车功能的 PRD
```

### 方式二：并行调用多个 Agent

```
你：帮我开发用户认证模块。
    让 product-manager 写需求，
    backend-dev 设计 API，
    frontend-dev 实现页面，
    三个角色并行工作。
```

Claude 会自动并行派发多个子代理。

### 方式三：使用斜杠命令（预定义流程）

```
你：/sprint 用户认证模块（登录、注册、忘记密码）
```

这会触发 `.claude/commands/sprint.md` 中定义的完整流程。

### 方式四：Agent Team 分屏模式（高级）

```bash
# 以 tmux 分屏模式启动
claude --teammate-mode tmux
```

在这个模式下，多个 Claude 会话会在 tmux 窗口中并行运行，可以互相通信。

---

## 5. Agent Team 分屏模式

### 安装 tmux

```bash
# macOS
brew install tmux

# Ubuntu / Debian
sudo apt install tmux

# CentOS / RHEL
sudo yum install tmux
```

### 启动分屏模式

```bash
# 方式 1：自动检测最佳方案
claude --teammate-mode auto

# 方式 2：强制使用 tmux
claude --teammate-mode tmux
```

### 在 settings.json 中设置默认模式

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  },
  "teammateMode": "tmux"
}
```

### tmux 分屏模式下的操作

启动后，你会看到终端被分成多个窗格，每个窗格是一个独立的 Claude 会话：

```
┌─────────────────────────────────────────┐
│  窗格 1：主编排器（你的主会话）           │
│  > 帮我组建一个团队来开发用户认证...      │
├──────────────────────┬──────────────────┤
│  窗格 2：后端开发     │  窗格 3：前端开发  │
│  > 正在设计 API...   │  > 正在实现组件... │
├──────────────────────┴──────────────────┤
│  窗格 4：代码审查                         │
│  > 等待开发完成后审查...                  │
└─────────────────────────────────────────┘
```

### Agent 间通信

Agent Team 模式下，Agent 之间可以通过以下工具通信：

| 工具 | 用途 |
|------|------|
| `SendMessage` | 直接给另一个 Agent 发消息 |
| `TaskCreate` | 创建任务到共享任务列表 |
| `TaskUpdate` | 更新任务状态 |
| `TaskList` | 查看所有任务 |

---

## 6. 斜杠命令

### 创建自定义命令

在 `.claude/commands/` 目录下创建 `.md` 文件：

```markdown
# .claude/commands/sprint.md

启动一个全栈 Sprint 开发流程来完成以下任务：$ARGUMENTS

按以下步骤执行：

1. **需求分析** - 让 product-manager 编写 PRD 和用户故事
2. **UI 设计** - 让 ui-designer 设计界面方案和组件规范
3. **并行开发** - 让 backend-dev 和 frontend-dev 同时开发
4. **基础设施** - 让 devops 配置 Docker 和 CI/CD
5. **代码审查** - 让 code-reviewer 进行全方位审查
6. **汇总报告** - 生成 Sprint 产出清单和质量报告
```

`$ARGUMENTS` 会被替换为你在命令后输入的参数。

### 使用斜杠命令

```
你：/sprint 用户认证模块

# Claude 会按照 sprint.md 中定义的步骤执行
```

### 常用命令建议

| 命令文件 | 用途 | 调用方式 |
|---------|------|---------|
| `sprint.md` | 全栈 Sprint 开发 | `/sprint 功能描述` |
| `build-team.md` | 查看/组建团队 | `/build-team` |
| `review.md` | 代码审查 | `/review 文件或目录` |
| `deploy.md` | 部署检查 | `/deploy 环境名` |

---

## 7. Workflow 工作流编排

对于复杂的多阶段任务，可以使用 Workflow 脚本：

```
你：用 Workflow 帮我做一个完整的代码审查流程，
    分别审查安全性、性能、代码风格，
    最后汇总所有发现。
```

Claude 会生成一个工作流脚本，编排多个子代理并行或串行执行。

### Workflow 基本概念

```
Workflow 脚本（JavaScript）
├── agent()      → 派发单个子代理
├── parallel()   → 并行执行多个任务
├── pipeline()   → 流水线执行（每个 item 经过多个阶段）
├── phase()      → 标记阶段（用于进度显示）
└── log()        → 输出进度消息
```

---

## 8. 实战案例

### 案例 1：开发一个 Todo App

```
你：帮我开发一个 Todo 应用。
    让 product-manager 先写需求，
    然后 ui-designer 设计界面，
    接着 backend-dev 和 frontend-dev 并行开发，
    最后 code-reviewer 审查。
```

### 案例 2：安全审查

```
你：让 code-reviewer 对 src/ 目录做一次全面的安全审查，
    重点关注认证、输入校验和 SQL 注入。
```

### 案例 3：性能优化

```
你：让 backend-dev 分析 src/api/ 下的所有数据库查询，
    找出 N+1 查询问题并优化。
    然后让 code-reviewer 审查优化方案。
```

### 案例 4：使用斜杠命令一键 Sprint

```
你：/sprint 支付模块（集成 Stripe，支持信用卡和支付宝）
```

---

## 9. 常见问题

### Q：Agent Team 和 Subagent 有什么区别？

| | Subagent | Agent Team |
|---|---------|------------|
| **运行方式** | 主 Agent 内部派发 | 多个独立 Claude 会话 |
| **通信能力** | 不能互相通信 | 可以互相发消息 |
| **适用场景** | 独立子任务 | 需要协调的复杂项目 |
| **启用方式** | 默认可用 | 需要启用实验性功能 |
| **并行能力** | 主 Agent 内并行 | 独立进程并行 |

### Q：自定义的 Subagent 不生效？

1. 检查文件是否在正确的目录下（`.claude/agents/` 或 `~/.claude/agents/`）
2. 检查 YAML 头部的 `name` 是否正确
3. **重启 Claude Code 会话**——Subagent 在会话启动时加载
4. 检查 `description` 是否与你的请求匹配

### Q：多个 Agent 修改同一个文件冲突了？

1. 使用 `isolation: "worktree"` 为每个 Agent 创建独立的 Git 工作树
2. 或者手动让不同 Agent 负责不同文件/目录

### Q：Agent Team 分屏不工作？

```bash
# 确认 tmux 已安装
tmux -V

# 确认配置
cat ~/.claude/settings.json

# 手动指定 tmux 模式
claude --teammate-mode tmux
```

### Q：如何查看当前有哪些可用的 Subagent？

在 Claude Code 会话中输入：

```
你：列出当前项目所有可用的 subagent
```

或使用 `/agents` 命令（如果可用）。

### Q：Subagent 可以使用哪些工具？

| 工具 | 说明 |
|------|------|
| `Read` | 读取文件 |
| `Write` | 写入文件 |
| `Edit` | 编辑文件 |
| `Grep` | 搜索文件内容 |
| `Glob` | 搜索文件名 |
| `Bash` | 执行 Shell 命令 |
| `WebSearch` | 搜索网页 |
| `WebFetch` | 获取网页内容 |
| `NotebookEdit` | 编辑 Jupyter notebook |

在 Subagent 的 frontmatter 中通过 `tools` 和 `disallowedTools` 控制权限。

---

## 快速参考卡片

```
┌─────────────────────────────────────────────────────┐
│              Claude Code Agent Team 速查             │
├─────────────────────────────────────────────────────┤
│                                                     │
│  启用 Agent Team：                                   │
│    settings.json → "CLAUDE_CODE_EXPERIMENTAL_       │
│                     AGENT_TEAMS": "1"               │
│                                                     │
│  启动分屏模式：                                      │
│    claude --teammate-mode tmux                      │
│                                                     │
│  调用 Subagent：                                     │
│    "让 [agent-name] 做 [任务]"                       │
│                                                     │
│  斜杠命令：                                          │
│    /sprint 功能描述                                  │
│    /build-team                                      │
│                                                     │
│  Agent 定义文件位置：                                 │
│    .claude/agents/xxx.md     （项目级）              │
│    ~/.claude/agents/xxx.md   （用户级）              │
│                                                     │
│  Agent 间通信工具：                                  │
│    SendMessage / TaskCreate / TaskUpdate            │
│                                                     │
│  模型选择：                                          │
│    haiku = 搜索/文档（快）                           │
│    sonnet = 开发/设计（平衡）                        │
│    opus = 审查/架构（深度）                          │
│                                                     │
└─────────────────────────────────────────────────────┘
```
