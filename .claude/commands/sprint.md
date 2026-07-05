启动一个全栈 Sprint 开发流程来完成以下任务：$ARGUMENTS

## 执行流程（基于 Superpowers 工作流）

### 阶段 0：预检
- 检查是否已有隔离工作空间（worktree），没有则创建
- 运行项目基线测试，确保从干净状态开始

### 阶段 1：需求分析（brainstorming）
- 让 product-manager 编写 PRD 和用户故事
- 如果需求模糊，先通过逐步提问澄清，一次一个问题
- 输出设计文档到 docs/specs/

### 阶段 2：设计与规划（writing-plans）
- 让 ui-designer 设计界面方案和组件规范
- 让 backend-dev 设计 API 契约和数据库 Schema
- 生成详细的分任务实现计划，每步 2-5 分钟粒度

### 阶段 3：并行开发（subagent-driven-development）
- 按计划逐任务执行，每个任务调度对应的专业 Agent：
  - 前端任务 → frontend-dev（遵循 TDD：先写失败测试 → 最小实现 → 验证通过）
  - 后端任务 → backend-dev（遵循 TDD：先写失败测试 → 最小实现 → 验证通过）
  - 基础设施 → devops（Docker + CI/CD 配置）
- 每个任务完成后立即进行代码审查（code-reviewer）
- 遇到 bug 时使用系统化调试流程（先找根因，严禁猜测性修复）

### 阶段 4：质量门禁（verification-before-completion）
- 运行完整测试套件，确认所有测试通过
- code-reviewer 进行最终全分支审查
- 独立验证：检查 git diff、运行测试，不信任任何"应该可以"的声明
- 审查反馈处理：技术验证后再实施，有理由时可 push back

### 阶段 5：完成（finishing-a-development-branch）
- 确认测试全部通过后，呈现集成选项：
  1. 本地合并到主分支
  2. 推送并创建 PR
  3. 保留分支继续迭代
  4. 丢弃变更
- 执行用户选择的操作并清理工作空间

## 注意事项
- 独立任务可并行调度多个 Agent（dispatching-parallel-agents）
- 紧耦合任务串行执行，避免文件冲突
- 每个 Agent 完成后 Controller 独立验证产出
- 连续执行不中断，除非遇到真正的阻塞
