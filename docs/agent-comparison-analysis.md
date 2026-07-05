# 三大开源项目 Agent 定义对比分析 & 改进方案

> 分析时间：2026-07-04
> 分析对象：oh-my-claudecode / agency-agents / oh-my-openagent

---

## 一、核心差距：我们之前写的 vs 专业项目

| 维度 | 我们之前的版本 | oh-my-claudecode | agency-agents | oh-my-openagent |
|------|-------------|-----------------|---------------|-----------------|
| **单文件行数** | 30-50 行 | 78-280 行 | 154-749 行 | 动态构建（500+ LOC） |
| **Prompt 结构** | Markdown 松散列表 | XML 标签结构化 | 6 层角色定义系统 | TypeScript 工厂 + 运行时拼接 |
| **人格定义** | ❌ 无 | ⚠️ 隐含在 Role 中 | ✅ Identity + Personality + Memory | ✅ 神话命名 + 性格映射 |
| **动机解释** | ❌ 无 | ✅ Why_This_Matters | ✅ Experience（"你见过系统因…"） | ⚠️ 隐含 |
| **反模式教学** | ❌ 无 | ✅ Failure_Modes + Good/Bad 对比 | ✅ VULNERABLE vs SECURE 代码对比 | ✅ 模型专属 anti-pattern |
| **交付物模板** | ❌ 无 | ✅ Output_Format 结构化 | ✅ 完整可运行代码 + 模板 | ✅ 动态交付物 |
| **成功指标** | ❌ 无 | ✅ Success_Criteria | ✅ 可量化 Metrics | ⚠️ 隐含 |
| **失败断路器** | ❌ 无 | ✅ 3-failure circuit breaker | ⚠️ 部分 | ✅ escalation chain |
| **读写分离** | ❌ 无 | ✅ disallowedTools | ⚠️ 按角色自然分离 | ✅ denied tools |
| **最终交付契约** | ❌ 无 | ✅ Final_Response_Contract | ⚠️ 隐含 | ✅ 结构化返回 |
| **沟通风格** | ❌ 无 | ⚠️ 隐含 | ✅ 具体例句模板 | ⚠️ 隐含 |
| **协作规则** | ⚠️ 简单提及 | ✅ External_Consultation | ✅ strategy/coordination/ | ✅ 委派关系图 |

---

## 二、三个项目各自的核心优势

### 🏆 oh-my-claudecode（最佳结构化 Prompt 模板）

**10 大设计模式**：
1. **XML 结构化标签** — `<Role>` `<Why_This_Matters>` `<Constraints>` `<Output_Format>` `<Failure_Modes_To_Avoid>`
2. **动机先行（Why_This_Matters）** — 解释规则背后的因果，LLM 理解后更遵守
3. **Good/Bad 对比** — 反模式教学比正面指令更有效
4. **发现-过滤分离** — 审查阶段追求覆盖率，排序交给下游
5. **最终交付契约** — "永远不要用 done/complete 结尾"
6. **自适应严厉度** — 发现严重问题后从 THOROUGH 升级到 ADVERSARIAL
7. **3-failure 断路器** — 3 次失败后升级而非死循环
8. **disallowedTools** — 工具层面强制读写分离
9. **多视角审查** — 安全工程师/新员工/运维 三个视角
10. **模型分级路由** — haiku(搜索) / sonnet(执行) / opus(审查)

### 🏆 agency-agents（最佳人格化 + 交付物模板）

**7 大设计模式**：
1. **6 层角色定义系统** — Frontmatter → Identity → Mission+Rules → Deliverables → Workflow → Metrics
2. **人格不是装饰** — "你见过 Equifax 因一个漏补丁被攻破，Log4Shell 因 JNDI 注入…"
3. **交付物含真实代码** — 完整 CSS Token、TypeScript 组件、SQL Schema、YAML Pipeline
4. **VULNERABLE vs SECURE 对比** — 安全工程师的核心教学法
5. **可量化成功指标** — "API < 200ms p95"、"SAST 误报率 < 20%"
6. **沟通风格有原话模板** — 不是"专业沟通"，而是给出具体怎么说
7. **vibe 一句话** — frontmatter 里的 vibe 字段，快速理解 agent 性格

### 🏆 oh-my-openagent（最佳编排架构）

**7 大设计模式**：
1. **神话命名系统** — Sisyphus(永不停歇)/Hephaestus(锻造)/Prometheus(先知)/Atlas(擎天)/Momus(挑刺)
2. **模型专属 Prompt 变体** — 同一 agent 对 Claude/GPT/Gemini/Kimi 有不同 prompt
3. **运行时动态 Prompt** — 根据可用 agent/tool/skill 动态构建
4. **三层 Agent 定义** — Raw Factory → Conditional Factory → Registration
5. **5-tier Hook 系统** — Session + ToolGuard + Transform + Continuation + Skill
6. **分层编排** — Metis(预分析)→Prometheus(规划)→Momus(审查)→Sisyphus(编排)→Hephaestus(执行)
7. **Team Mode 资格门控** — eligible/conditional/hard-reject 三级

---

## 三、我们的改进方案

### 采纳策略：取三家之长

| 来源 | 采纳什么 | 怎么用 |
|------|---------|--------|
| oh-my-claudecode | XML 结构化 Prompt 模板 | 每个 agent 统一结构 |
| oh-my-claudecode | disallowedTools 读写分离 | 审查类 agent 禁止写 |
| oh-my-claudecode | Why_This_Matters 动机先行 | 每个规则解释为什么 |
| oh-my-claudecode | Failure_Modes Good/Bad 对比 | 反模式教学 |
| oh-my-claudecode | Final_Response_Contract | 结构化交付 |
| oh-my-claudecode | 3-failure 断路器 | 失败升级机制 |
| agency-agents | 6 层角色定义 | Identity→Mission→Rules→Deliverables→Workflow→Metrics |
| agency-agents | 人格+记忆+经验 | 让 agent 有"直觉" |
| agency-agents | 真实代码交付物 | 完整的模板和示例代码 |
| agency-agents | 可量化成功指标 | 具体数字而非模糊描述 |
| agency-agents | 沟通风格例句 | 具体怎么说 |
| agency-agents | vibe 一句话 | frontmatter 增加 vibe |
| oh-my-openagent | 模型分级路由 | haiku/sonnet/opus 合理分配 |
| oh-my-openagent | 编排分层 | 规划→审查→执行的流程 |

### 最终 Agent 定义结构（融合版）

```yaml
---
name: xxx
description: >
  一句话描述（用于路由选择）
vibe: 一句话性格标签
tools: Read, Grep, Glob, Write, Edit, Bash
disallowedTools: Write, Edit          # 审查类才有
model: opus                           # 按复杂度选模型
level: 3                              # 2=执行 3=专家 4=战略
---
```

```xml
<Agent_Prompt>
  <Identity>           ← 你是谁 + 人格 + 记忆 + 经验
  <Core_Mission>       ← 3-5 个核心方向
  <Why_This_Matters>   ← 为什么这些规则重要
  <Critical_Rules>     ← 硬约束红线（6-8 条）
  <Workflow>           ← 分步骤工作流程
  <Deliverables>       ← 交付物模板 + 真实代码示例
  <Failure_Modes>      ← Good/Bad 对比反模式
  <Communication>      ← 沟通风格 + 具体例句
  <Success_Metrics>    ← 可量化指标
  <Collaboration>      ← 与其他 Agent 的协作规则
  <Final_Contract>     ← 最终交付契约
  <Circuit_Breaker>    ← 3 次失败后升级
</Agent_Prompt>
```

---

## 四、行数对比（改进前 vs 改进后目标）

| Agent | 改进前 | 改进后目标 | 参考基准 |
|-------|--------|-----------|---------|
| ui-designer | 40 行 | 200+ 行 | agency: 382 行 |
| frontend-dev | 35 行 | 200+ 行 | agency: 224 行 |
| backend-dev | 40 行 | 230+ 行 | agency: 236 行 |
| devops | 30 行 | 250+ 行 | agency: 375 行 |
| code-reviewer | 50 行 | 240+ 行 | omc: 242 行 |
| product-manager | 45 行 | 300+ 行 | agency: 469 行 |
