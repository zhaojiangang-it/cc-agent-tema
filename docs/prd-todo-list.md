# Todo List 应用 - 产品需求文档 (PRD)

> 版本：v1.0 | 日期：2026-07-04 | 作者：product-manager agent
> 状态：草案

---

## 1. 产品概述

### 1.1 产品定位

一个**个人工具级**的待办事项管理应用，帮助开发者/个人用户管理日常任务。

**核心原则**：够用就好，不过度设计。

### 1.2 目标用户

- Rose 本人（主要用户）
- 有类似需求的个人用户（潜在）

### 1.3 核心价值

- 快速记录待办事项
- 标记完成/未完成
- 按分类查看任务
- 轻量、快速、不依赖外部服务

### 1.4 MVP 功能清单

| # | 功能 | 优先级 | 说明 |
|---|------|--------|------|
| F1 | 创建待办 | P0 | 输入标题，可选描述 |
| F2 | 查看待办列表 | P0 | 默认按创建时间倒序 |
| F3 | 标记完成/取消完成 | P0 | 点击切换状态 |
| F4 | 删除待办 | P0 | 单条删除 |
| F5 | 编辑待办 | P1 | 修改标题、描述 |
| F6 | 分类/标签 | P1 | 给待办打标签，按标签筛选 |
| F7 | 筛选视图 | P1 | 全部/未完成/已完成 |
| F8 | 搜索 | P2 | 按标题关键词搜索 |
| F9 | 批量操作 | P2 | 批量删除、批量完成 |
| F10 | 暗色模式 | P2 | 跟随系统偏好 |

**不做的事（及原因）**：

| 不做 | 原因 |
|------|------|
| 用户认证/多用户 | 个人工具，单用户使用，部署在内网或个人服务器 |
| 协作/分享 | 超出个人工具范畴 |
| 提醒/通知 | 增加复杂度，不在 MVP 范围 |
| 子任务/嵌套 | 保持简单，一个层级足够 |
| 移动端 App | 响应式 Web 即可，不做原生 |
| 云同步 | 本地 SQLite 足够，不做多端同步 |

---

## 2. 用户故事

### P0 - 必须有

| 编号 | 用户故事 |
|------|---------|
| US-01 | As a 用户, I want 快速创建一条待办事项, so that 我可以随时记录想到的任务 |
| US-02 | As a 用户, I want 查看所有待办事项列表, so that 我知道还有什么要做 |
| US-03 | As a 用户, I want 标记一条待办为已完成, so that 我可以追踪进度 |
| US-04 | As a 用户, I want 取消已完成状态, so that 误操作时可以恢复 |
| US-05 | As a 用户, I want 删除一条不需要的待办, so that 列表保持整洁 |

### P1 - 应该有

| 编号 | 用户故事 |
|------|---------|
| US-06 | As a 用户, I want 编辑待办的标题和描述, so that 我可以修正或补充信息 |
| US-07 | As a 用户, I want 给待办添加标签, so that 我可以按类别组织任务 |
| US-08 | As a 用户, I want 按标签筛选待办, so that 我可以快速找到某类任务 |
| US-09 | As a 用户, I want 按状态筛选（全部/未完成/已完成）, so that 我可以聚焦当前任务 |

### P2 - 可以有

| 编号 | 用户故事 |
|------|---------|
| US-10 | As a 用户, I want 搜索待办事项, so that 我可以快速定位特定任务 |
| US-11 | As a 用户, I want 批量删除/完成待办, so that 我可以高效清理列表 |
| US-12 | As a 用户, I want 暗色模式, so that 长时间使用不刺眼 |

---

## 3. 功能需求

### 3.1 数据模型

#### Todo（待办事项）

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| id | INTEGER | PK, AUTOINCREMENT | 主键 |
| title | VARCHAR(200) | NOT NULL | 标题 |
| description | TEXT | NULLABLE | 描述，可选 |
| completed | BOOLEAN | NOT NULL, DEFAULT FALSE | 是否完成 |
| created_at | DATETIME | NOT NULL, DEFAULT NOW | 创建时间 |
| updated_at | DATETIME | NOT NULL, AUTO UPDATE | 更新时间 |

#### Tag（标签）

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| id | INTEGER | PK, AUTOINCREMENT | 主键 |
| name | VARCHAR(50) | NOT NULL, UNIQUE | 标签名 |
| color | VARCHAR(7) | NULLABLE | 颜色代码，如 #FF5733 |

#### TodoTag（关联表，多对多）

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| todo_id | INTEGER | FK -> Todo.id, NOT NULL | 待办 ID |
| tag_id | INTEGER | FK -> Tag.id, NOT NULL | 标签 ID |

> PK: (todo_id, tag_id) 联合主键

### 3.2 页面结构

```
+------------------------------------------+
|  Todo List                   [暗色切换]   |
+------------------------------------------+
|  [+ 新建待办]                            |
|                                          |
|  [全部] [未完成] [已完成]                |
|  [标签筛选 ▼]     [搜索框]              |
|                                          |
|  +--------------------------------------+|
|  | ○ 买牛奶          [生活] [编辑][删除]||
|  +--------------------------------------+|
|  +--------------------------------------+|
|  | ✓ 写周报          [工作] [编辑][删除]||
|  +--------------------------------------+|
|  +--------------------------------------+|
|  | ○ 修复登录bug     [工作] [编辑][删除]||
|  +--------------------------------------+|
|                                          |
|  共 3 条 | 已完成 1 条                   |
+------------------------------------------+
```

单个页面完成所有操作，不跳转。编辑通过弹窗（Dialog）完成。

### 3.3 API 接口清单

基础路径：`/api`

#### 待办事项

| 方法 | 路径 | 说明 | 请求体/参数 | 响应 |
|------|------|------|------------|------|
| GET | `/api/todos` | 获取待办列表 | Query: `status`(all/active/completed), `tag_id`, `keyword` | `{ items: Todo[], total: number }` |
| POST | `/api/todos` | 创建待办 | `{ title: string, description?: string, tag_ids?: number[] }` | `Todo` |
| GET | `/api/todos/:id` | 获取单条待办 | - | `Todo` |
| PUT | `/api/todos/:id` | 更新待办 | `{ title?: string, description?: string, tag_ids?: number[] }` | `Todo` |
| PATCH | `/api/todos/:id/toggle` | 切换完成状态 | - | `Todo` |
| DELETE | `/api/todos/:id` | 删除待办 | - | `{ success: true }` |
| DELETE | `/api/todos/batch` | 批量删除 | `{ ids: number[] }` | `{ success: true, deleted: number }` |
| PATCH | `/api/todos/batch/complete` | 批量完成 | `{ ids: number[] }` | `{ success: true, updated: number }` |

#### 标签

| 方法 | 路径 | 说明 | 请求体/参数 | 响应 |
|------|------|------|------------|------|
| GET | `/api/tags` | 获取所有标签 | - | `Tag[]` |
| POST | `/api/tags` | 创建标签 | `{ name: string, color?: string }` | `Tag` |
| PUT | `/api/tags/:id` | 更新标签 | `{ name?: string, color?: string }` | `Tag` |
| DELETE | `/api/tags/:id` | 删除标签 | - | `{ success: true }` |

#### 响应格式约定

成功响应：
```json
{
  "code": 0,
  "data": { ... },
  "message": "ok"
}
```

列表响应：
```json
{
  "code": 0,
  "data": {
    "items": [...],
    "total": 10
  },
  "message": "ok"
}
```

错误响应：
```json
{
  "code": 40001,
  "data": null,
  "message": "标题不能为空"
}
```

---

## 4. 非功能需求

### 4.1 性能

| 指标 | 要求 | 说明 |
|------|------|------|
| 页面首屏加载 | < 2s | 3G 网络下 |
| API 响应时间 | < 200ms | 单次请求（P95） |
| 列表渲染 | < 500ms | 100 条数据以内无卡顿 |

### 4.2 安全

- 本应用为单用户个人工具，不做用户认证
- API 部署时通过 Nginx 限制访问来源（内网或指定 IP）
- 输入校验：防止 XSS，title 和 description 做基本的转义处理
- SQLite 使用参数化查询，防止 SQL 注入

### 4.3 兼容性

- 浏览器：Chrome 90+, Firefox 90+, Safari 15+, Edge 90+
- 响应式布局：适配桌面端（主要）和平板端
- 不强制支持移动端（但基本可用）

### 4.4 可用性

- 离线可用：不做要求（需要网络连接后端 API）
- 数据持久化：SQLite 文件存储，应用重启数据不丢失

---

## 5. 技术选型

> 以下选型完全基于 Rose 的技术栈偏好，优先选择已掌握的技术。

| 层级 | 技术 | 理由 |
|------|------|------|
| 前端框架 | Vue 3 + TypeScript | Rose 首选，Composition API |
| UI 组件库 | Element Plus | Rose 首选，成熟稳定 |
| 状态管理 | Pinia | Vue 3 官方推荐，轻量 |
| HTTP 客户端 | Axios | 常用，简单 |
| 构建工具 | Vite | 快，Vue 生态标配 |
| 后端框架 | Python FastAPI | 轻量应用首选，开发速度快 |
| ORM | SQLAlchemy + asyncio | FastAPI 生态标配，支持异步 |
| 数据库 | SQLite | 个人工具不需要额外数据库服务，文件级存储 |
| 数据库迁移 | Alembic | SQLAlchemy 配套，简单 |
| 部署 | Docker + Docker Compose | 前后端打包，一键启动 |
| 反向代理 | Nginx | 生产环境前端静态资源 + API 代理 |

### 项目目录结构建议

```
todo-app/
├── frontend/               # Vue 3 前端
│   ├── src/
│   │   ├── components/     # 组件
│   │   ├── views/          # 页面
│   │   ├── stores/         # Pinia stores
│   │   ├── api/            # API 调用封装
│   │   ├── types/          # TypeScript 类型
│   │   └── App.vue
│   ├── package.json
│   └── vite.config.ts
├── backend/                # FastAPI 后端
│   ├── app/
│   │   ├── api/            # 路由
│   │   ├── models/         # SQLAlchemy 模型
│   │   ├── schemas/        # Pydantic schemas
│   │   ├── database.py     # 数据库连接
│   │   └── main.py         # 入口
│   ├── requirements.txt
│   └── todo.db             # SQLite 数据库文件
├── docker-compose.yml
├── Dockerfile.frontend
├── Dockerfile.backend
└── nginx.conf
```

---

## 6. 验收标准

### 6.1 功能验收

#### US-01：创建待办

- [ ] 输入标题后点击创建按钮，新待办出现在列表顶部
- [ ] 标题为空时，创建按钮禁用或提示"标题不能为空"
- [ ] 标题最大长度 200 字符，超出截断或提示
- [ ] 描述为可选项，不填时不展示描述区域
- [ ] 支持在创建时选择标签（P1）

#### US-02：查看待办列表

- [ ] 页面加载后，默认显示所有待办，按创建时间倒序排列
- [ ] 每条待办显示：完成状态图标、标题、标签、操作按钮
- [ ] 列表底部显示统计：共 X 条，已完成 Y 条
- [ ] 无数据时显示空状态提示

#### US-03/04：标记完成/取消完成

- [ ] 点击待办前的圆形图标，切换为已完成状态（打勾 + 文字划线）
- [ ] 再次点击，恢复为未完成状态
- [ ] 状态切换即时生效，无需额外确认
- [ ] 切换后 updated_at 字段更新

#### US-05：删除待办

- [ ] 点击删除按钮，弹出确认提示
- [ ] 确认后删除，列表即时更新
- [ ] 关联的标签关系同步删除

#### US-06：编辑待办

- [ ] 点击编辑按钮，弹出编辑弹窗
- [ ] 弹窗中可修改标题、描述、标签
- [ ] 保存后列表即时更新
- [ ] 标题为空时无法保存

#### US-07/08：标签功能

- [ ] 可以创建标签（输入名称，可选颜色）
- [ ] 标签名不能重复
- [ ] 创建/编辑待办时可选择多个标签
- [ ] 标签筛选下拉显示所有标签，点击后列表只显示含该标签的待办
- [ ] 删除标签时，不影响已关联的待办（只删除关联关系）

#### US-09：状态筛选

- [ ] 点击"全部"/"未完成"/"已完成"按钮，列表正确筛选
- [ ] 筛选条件可与标签筛选组合使用
- [ ] 当前激活的筛选按钮有高亮样式

### 6.2 测试要求

| 测试类型 | 覆盖范围 | 工具 |
|---------|---------|------|
| 后端单元测试 | 所有 API 端点的正常/异常路径 | pytest + httpx |
| 后端集成测试 | 数据库 CRUD 操作 | pytest + SQLite 内存库 |
| 前端单元测试 | 核心组件和 store 逻辑 | Vitest |
| E2E 测试 | 核心用户流程（创建、完成、删除） | Playwright |

**关键路径测试用例**：

1. 创建待办 -> 验证列表中出现新条目
2. 标记完成 -> 验证状态变更和 UI 更新
3. 删除待办 -> 验证从列表移除
4. 标签筛选 -> 验证筛选结果正确
5. 输入校验 -> 验证空标题被拒绝
6. 批量操作 -> 验证批量删除/完成生效

### 6.3 部署验收

- [ ] `docker-compose up` 一键启动成功
- [ ] 前端页面正常加载
- [ ] API 请求正常响应
- [ ] 数据持久化：重启容器后数据不丢失

---

## 7. 开发优先级与里程碑

### M1 - 核心功能（P0）

> 目标：可运行的最小可用版本

- 创建/查看/完成/删除待办
- 基本的列表 UI
- 后端 CRUD API
- Docker 部署

### M2 - 增强功能（P1）

> 目标：提升可用性

- 编辑待办
- 标签系统
- 状态筛选

### M3 - 锦上添花（P2）

> 目标：体验优化

- 搜索
- 批量操作
- 暗色模式

---

## 8. 风险与缓解

| 风险 | 影响 | 缓解措施 |
|------|------|---------|
| SQLite 并发限制 | 多请求同时写入时可能锁表 | 个人工具并发极低，不是问题；如后续扩展，迁移到 PostgreSQL |
| MacBook 内存限制（16GB） | Docker 容器占用资源 | 使用 Docker Compose 的轻量配置，限制容器内存 |
| 前端过度设计 | 开发时间膨胀 | 严格遵循 MVP，UI 使用 Element Plus 默认样式 |

---

## 附录：API 接口详细示例

### 创建待办 - POST `/api/todos`

请求：
```json
{
  "title": "买牛奶",
  "description": "全脂，两盒",
  "tag_ids": [1, 2]
}
```

响应：
```json
{
  "code": 0,
  "data": {
    "id": 1,
    "title": "买牛奶",
    "description": "全脂，两盒",
    "completed": false,
    "tags": [
      { "id": 1, "name": "生活", "color": "#67C23A" },
      { "id": 2, "name": "紧急", "color": "#F56C6C" }
    ],
    "created_at": "2026-07-04T10:30:00",
    "updated_at": "2026-07-04T10:30:00"
  },
  "message": "ok"
}
```

### 获取待办列表 - GET `/api/todos?status=active&tag_id=1`

响应：
```json
{
  "code": 0,
  "data": {
    "items": [
      {
        "id": 3,
        "title": "修复登录bug",
        "description": null,
        "completed": false,
        "tags": [{ "id": 1, "name": "工作", "color": "#409EFF" }],
        "created_at": "2026-07-04T09:00:00",
        "updated_at": "2026-07-04T09:00:00"
      }
    ],
    "total": 1
  },
  "message": "ok"
}
```

### 切换完成状态 - PATCH `/api/todos/1/toggle`

响应：
```json
{
  "code": 0,
  "data": {
    "id": 1,
    "title": "买牛奶",
    "description": "全脂，两盒",
    "completed": true,
    "tags": [],
    "created_at": "2026-07-04T10:30:00",
    "updated_at": "2026-07-04T11:00:00"
  },
  "message": "ok"
}
```
