# 项目说明

## 开发者画像（Rose）

> **所有 Agent 在产出方案前必须阅读本节。** 方案必须贴合 Rose 的实际技术栈，优先选择已掌握的技术。

- **定位**：后端为主、前端辅助的全栈工程师，有华为/阿里巴巴大厂工程化经验
- **后端主力**：Java（Spring Boot / Spring Cloud Alibaba）、Python（FastAPI）、MySQL/PostgreSQL、Redis
- **AI 方向**：阿里云百炼（DashScope API）、OpenAI 兼容 API、Anthropic Claude API
- **前端**：Vue + Element UI（首选）、React + Ant Design（会但不首选）
- **运维**：Docker、Nginx、阿里云 ECS、Linux Shell。K8s 学习中
- **协作偏好**：
  - 中文沟通
  - 偏好实用简洁，反对过度工程
  - 基础设施优先国内方案（阿里云），避免依赖海外服务
  - 快速决策，不纠结细节

## 技术栈（本模板默认，可按项目调整）

- 前端：Vue 3 + TypeScript + Element Plus（首选）/ React + Tailwind（备选）
- 后端：Java Spring Boot / Python FastAPI（AI 应用）
- 数据库：MySQL / PostgreSQL + Redis
- 部署：Docker + 阿里云 ECS + Nginx
- AI 能力：阿里云百炼 CLI（`bl` 命令）
- 测试：对应技术栈的测试框架

## 方案设计原则

1. **优先匹配**：从 Rose 已掌握的技术栈中选，给出推荐理由
2. **标注新技术**：未接触过的技术标「⚠️ 新技术」+ 学习成本评估
3. **避免过度设计**：小项目不上微服务，不过度抽象
4. **国内优先**：云服务、第三方优先阿里云等国内方案
5. **硬件约束**：MacBook Air M4 / 16GB 内存 / 256GB 内置盘，方案要考虑资源限制

## 团队规范

- 代码风格：实用简洁，不追求过度抽象
- 提交信息遵循 Conventional Commits
- 关键路径必须包含测试
- PR 需要通过 code-reviewer 审查

## Agent 协作规则

- 前端和后端通过共享类型/接口契约协作
- 所有 API 变更需要同步更新接口文档
- UI 变更需要先确认设计方案
- **产品经理**出方案时必须先读「开发者画像」，确保需求可落地
- **前端开发**优先 Vue 体系，除非项目明确要求 React
- **后端开发**Java 项目用 Spring Boot，AI 应用用 FastAPI
- **运维工程师**默认阿里云 ECS 部署，善用 `bl` 命令调用百炼能力

## 可用 Agent 角色

| 角色 | subagent_type | 职责 | 与 Rose 的协作要点 |
|------|--------------|------|-------------------|
| 产品经理 | product-manager | 需求分析、PRD | 快速决策，贴合技术栈 |
| UI 设计师 | ui-designer | 界面设计、组件规范 | 实用简洁风格，不过度设计 |
| 前端开发 | frontend-dev | Vue/React 组件开发 | 优先 Vue + Element UI |
| 后端开发 | backend-dev | API、数据库、业务逻辑 | Java Spring Boot 或 Python FastAPI |
| 运维工程师 | devops | CI/CD、Docker、阿里云 | 阿里云 ECS + `bl` CLI |
| 代码审查 | code-reviewer | 代码质量、安全审计 | 关注实用性，不追求完美主义 |

## Superpowers 开发工作流

本项目集成了 Superpowers 插件体系，所有开发任务遵循以下工作流：

```
brainstorming（需求澄清）
    ↓
writing-plans（编写实现计划）
    ↓
using-git-worktrees（创建隔离工作空间）
    ↓
subagent-driven-development（逐任务调度子 Agent 执行）
    │  ├─ 每任务一个全新子 Agent（隔离上下文）
    │  ├─ 遵循 TDD（先写失败测试 → 最小实现 → 验证通过）
    │  └─ 每任务完成后立即代码审查
    ↓
verification-before-completion（独立验证所有产出）
    ↓
finishing-a-development-branch（合并/PR/保留/丢弃）
```

### 核心原则

- **Controller 是编排者**：不写实现代码，负责调度、审查、集成
- **子 Agent 是执行者**：每个任务全新上下文，遵循 TDD
- **审查是强制的**：每任务审查 + 最终全分支审查
- **验证是信任基础**：Controller 独立验证所有子 Agent 的输出，不信任"应该可以"
- **并行调度**：多个独立任务可同时派发（dispatching-parallel-agents）
- **系统化调试**：遇到 bug 先找根因，严禁猜测性修复，3 次失败后质疑架构

## 已安装 Skills 清单

### Superpowers 工作流 Skills（14 个）

| Skill | 用途 | 触发时机 |
|-------|------|---------|
| `brainstorming` | 需求澄清与设计规格 | 任何创造性工作开始前 |
| `writing-plans` | 将规格转化为分任务计划 | brainstorming 完成后 |
| `using-git-worktrees` | 创建隔离工作空间 | 执行计划前 |
| `subagent-driven-development` | 逐任务调度子 Agent 执行 | 主力执行引擎 |
| `executing-plans` | 单 Agent 串行执行计划 | 紧耦合任务的备选方案 |
| `dispatching-parallel-agents` | 并行调度多个独立 Agent | 多个互不相关的独立任务时 |
| `test-driven-development` | TDD 铁律：先写失败测试 | 所有实现代码前 |
| `systematic-debugging` | 系统化调试：先根因后修复 | 遇到 bug/测试失败时 |
| `requesting-code-review` | 调度代码审查 | 每任务后 + 合并前 |
| `receiving-code-review` | 处理审查反馈 | 收到审查结果时 |
| `verification-before-completion` | 独立验证完成声明 | 声称完成前必须有证据 |
| `finishing-a-development-branch` | 分支完成与集成 | 所有任务完成后 |
| `writing-skills` | 创建新 Skill | 需要扩展 Agent 能力时 |
| `using-superpowers` | 元规则：skill 调用优先 | 每次对话开始时 |

### 技术领域 Skills（8 个）

| Skill | 用途 | 关联 Agent |
|-------|------|-----------|
| `vue` | Vue 3 Composition API、响应式系统 | frontend-dev |
| `fastapi-python` | FastAPI 最佳实践、异步 API | backend-dev |
| `docker-deployment` | Docker 多阶段构建、Compose | devops |
| `web-design-guidelines` | Web 界面规范审查、无障碍 | ui-designer, code-reviewer |
| `better-auth-security-best-practices` | 认证安全加固、CSRF、限流 | backend-dev, code-reviewer |
| `bailian-cli` | 阿里云百炼 CLI（`bl` 命令） | devops, 所有角色 |
| `dws` | 钉钉产品能力管理 | product-manager |
| `find-skills` | 发现和安装新 Skill | 所有角色 |
| `myself` | Rose 的开发者画像 | 所有角色（方案设计必读） |
| `skill-creator` | 创建和优化 Skill | 扩展 Agent 能力时 |
