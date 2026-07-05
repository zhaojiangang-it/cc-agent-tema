# cc-agent-tema

基于 Claude Code 的多 Agent 协作开发模板，集成 Superpowers 工作流与角色化 Agent 团队。

## 内容

- `CLAUDE.md` — 项目规范、开发者画像、Agent 协作规则
- `.claude/agents/` — 角色化 Agent 定义（产品经理、前端、后端、运维、代码审查等）
- `.claude/commands/` — 自定义斜杠命令
- `docs/` — Agent 团队使用教程、PRD、安全报告等文档

## 特性

- 角色化 Agent 团队（产品经理 / 前端 / 后端 / 运维 / 代码审查 / UI 设计师）
- Superpowers 开发工作流（brainstorming → writing-plans → TDD → 验证 → 集成）
- 贴合国内技术栈（Vue + Element、Spring Boot / FastAPI、阿里云 ECS）

## 许可证

[MIT](./LICENSE)
