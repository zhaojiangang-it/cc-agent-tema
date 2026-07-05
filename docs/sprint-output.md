# Sprint 演示产出：用户登录功能

> 由 AI Agent Team 协作完成 | 2026-07-04

---

## 团队成员与分工

| 角色 | Agent | 产出 | 耗时 |
|------|-------|------|------|
| 产品经理 | product-manager | PRD 文档 + 用户故事 | ~30s |
| UI 设计师 | ui-designer | 组件方案 + Tailwind 代码 | ~122s |
| 后端开发 | backend-dev | API 设计 + Controller 代码 | ~103s |
| 运维工程师 | devops | Docker + CI/CD + 健康检查 | ~110s |

---

## 1. 产品需求文档 (PRD)

### 功能概述
用户登录功能是平台的核心入口，允许已注册用户通过账号密码安全访问系统。包含"记住我"免密登录与"忘记密码"自助找回能力。

### 用户故事

#### US-01: 账号密码登录 (P0)
**作为** 已注册用户，**我想** 使用邮箱和密码登录系统，**以便** 访问我的个人数据。

**验收标准：**
- [ ] 正确邮箱+密码 → 2 秒内跳转首页
- [ ] 错误凭证 → 统一提示"邮箱或密码错误"，5 次错误锁定 15 分钟
- [ ] 邮箱格式实时校验 + 密码显示/隐藏切换

#### US-02: 记住我 (P1)
**作为** 经常使用的用户，**我想** 勾选"记住我"后 7 天内自动登录。

**验收标准：**
- [ ] 勾选后关闭浏览器 7 天内免登录
- [ ] 未勾选则关闭浏览器后会话失效
- [ ] 退出/改密码/异常登录时令牌自动失效

#### US-03: 忘记密码 (P1)
**作为** 忘记密码的用户，**我想** 通过邮箱验证重置密码。

**验收标准：**
- [ ] 60 秒内发送重置邮件，链接 30 分钟有效且仅可使用一次
- [ ] 新密码强度要求：≥8 位，含大小写+数字，不与前 3 次相同
- [ ] 重置后注销所有设备会话 + 发送变更通知

### API 接口需求

| 接口 | 方法 | 说明 |
|------|------|------|
| `/api/v1/auth/login` | POST | 账号密码登录 |
| `/api/v1/auth/logout` | POST | 退出登录 |
| `/api/v1/auth/refresh` | POST | 刷新 Token |
| `/api/v1/auth/forgot-password` | POST | 发起密码重置 |
| `/api/v1/auth/reset-password` | POST | 执行密码重置 |
| `/api/v1/auth/validate-token` | GET | 校验 Token |

---

## 2. UI 设计方案

### 组件清单（8 个）
1. `LoginPage` - 页面容器 + 表单状态管理
2. `BrandHeader` - 品牌展示区
3. `FormInput` - 通用输入框（含 label/icon/错误提示）
4. `PasswordInput` - 密码输入框（显示/隐藏切换）
5. `CheckboxField` - "记住我"复选框
6. `SubmitButton` - 主按钮（支持 loading 状态）
7. `AuthLinks` - 第三方登录 + 辅助链接
8. `AlertBanner` - 全局消息提示

### 设计 Tokens
- 品牌主色：`#6366F1` (brand-500)
- 错误色：`#EF4444` (state-error)
- 成功色：`#10B981` (state-success)
- 卡片阴影：`0 4px 24px rgba(0,0,0,0.08)`

### 交互状态流转
```
默认(idle) → 输入中(filling) → 验证错误(error) / 提交中(submitting) → 成功(success)
```

### 响应式策略
- Mobile (<640px): 单列全宽
- Tablet (640-1024px): 居中卡片 max-w-md
- Desktop (>1024px): 可选双列布局（品牌区+表单区）

---

## 3. 后端技术方案

### 数据库设计
- `users` 表：含登录失败计数、锁定状态、密码变更时间
- `refresh_tokens` 表：只存哈希，支持主动吊销

### JWT Token 策略
- Access Token: 15 分钟，RS256 签名，存内存
- Refresh Token: 7 天，HttpOnly Secure Cookie，数据库存哈希

### 安全措施
- bcrypt (cost=12) 密码哈希
- 防时序攻击（dummy hash）
- Rate Limiting（每 IP 每分钟 10 次）
- 统一错误响应（不泄露用户是否存在）
- Zod 输入校验

### Controller 完整实现
包含 5 个端点的完整 TypeScript 代码：
- login（登录 + 锁定 + Token 签发）
- forgotPassword（重置令牌 + 异步邮件）
- resetPassword（事务更新 + 吊销所有 Token）
- refresh（Token 刷新 + 泄露检测）
- logout（Token 吊销 + Cookie 清除）

---

## 4. DevOps 方案

### Docker Compose
4 个服务：PostgreSQL + Redis + Backend + Frontend (Nginx)

### Dockerfile
- 3 阶段构建：deps → builder → production
- 最终镜像 ~150MB，非 root 运行
- 内置 HEALTHCHECK

### GitHub Actions CI/CD
4 阶段流水线：Lint → Test → Build → Deploy
- ESLint + TypeScript 检查
- 集成测试（PostgreSQL + Redis service containers）
- Docker 多阶段构建 + GHCR 推送
- SSH 部署 + 健康检查验证

### 健康检查端点
- `/health/live` - 存活探针
- `/health/ready` - 就绪探针（检查 DB + Redis）
- `/health` - 完整报告

---

## Sprint 质量总结

| 维度 | 产出物 | 状态 |
|------|--------|------|
| 需求分析 | PRD + 3 个用户故事 + 验收标准 | ✅ 完成 |
| UI 设计 | 8 个组件 + Design Tokens + 响应式方案 | ✅ 完成 |
| 后端开发 | 5 个 API + 数据库设计 + 安全方案 | ✅ 完成 |
| 运维部署 | Docker + CI/CD + 健康检查 | ✅ 完成 |
| 代码审查 | 待 Code Reviewer Agent 执行 | ⏳ 待执行 |
