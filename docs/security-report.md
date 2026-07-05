# Claude Code 阿里云 ECS 部署安全报告

> 编制日期：2026-07-04 | 适用环境：阿里云 ECS（Linux）+ Claude Code（含 Agent Team 模式）

---

## 1. Claude Code 安全模型分析

### 1.1 系统访问权限

Claude Code 作为 CLI 工具运行在服务器上，具有广泛的系统访问能力：

| 访问类型 | 能力范围 | 风险等级 |
|---------|---------|---------|
| **文件系统** | 可读写当前工作目录及子目录中的所有文件 | **高** |
| **Shell 执行** | 可通过 Bash 工具执行任意 Shell 命令（默认需用户确认） | **极高** |
| **网络访问** | 需要外网访问 `api.anthropic.com` | **中** |
| **环境变量** | 可读取运行进程的环境变量（含 API Key） | **高** |

**关键认知**：Claude Code 本质上是一个拥有完整 Shell 访问权限的 AI Agent。默认权限模型是「请求-确认」模式，但在 headless/YOLO 模式下保护层会被移除。

### 1.2 权限系统

```json
// .claude/settings.json
{
  "permissions": {
    "allow": [
      "Read", "Edit", "Write",
      "Bash(npm run *)", "Bash(npm test)"
    ],
    "deny": [
      "Bash(rm -rf *)", "Bash(sudo *)",
      "Bash(curl * | bash)", "Bash(chmod 777 *)"
    ]
  }
}
```

**⚠️ 绝对禁止**在服务器上使用 `claude --dangerously-skip-permissions`（YOLO 模式）。

### 1.3 网络端口

Claude Code 自身**不监听任何入站端口**——它只是出站连接到 Anthropic API。因此除 SSH 外不需要为 Claude Code 开放任何入站端口。

---

## 2. 已知漏洞（必须修复）

### CVE-2026-21852 — API Key 窃取（高危）

- **攻击方式**：恶意 Git 仓库可在信任对话框显示**之前**窃取 API Key
- **修复版本**：≥ **2.0.65**

### CVE-2025-59536 — MCP 配置远程代码执行（CVSS 8.7）

- **攻击方式**：恶意 MCP Server 配置 / 项目 hooks 执行任意命令
- **修复版本**：≥ **1.0.87**

### 必须执行

```bash
# 检查版本
claude --version

# 如果低于 2.0.65，立即更新
npm update -g @anthropic-ai/claude-code
```

---

## 3. 阿里云 ECS 攻击面分析

```
┌──────────────────────────────────────────────┐
│                    攻击面总览                  │
├──────────────────────────────────────────────┤
│  ① SSH 暴力破解 ──────► 获取服务器控制权      │
│  ② 恶意 Git 仓库 ────► CVE-2026-21852 API窃取│
│  ③ MCP Server 供应链 ─► 远程代码执行          │
│  ④ 安全组配置不当 ────► 未授权访问            │
│  ⑤ 元数据服务 SSRF ──► 阿里云凭证窃取         │
│  ⑥ Agent 过度权限 ───► 误操作/数据泄露        │
│  ⑦ .env 文件泄露 ────► API Key 暴露           │
└──────────────────────────────────────────────┘
```

### 端口风险矩阵

| 端口 | 服务 | 风险等级 | 建议 |
|------|------|---------|------|
| 22 (SSH) | SSH 登录 | **高** | 改非标准端口 + IP 白名单 |
| 3306 | MySQL | **极高** | 绝对禁止公网暴露 |
| 6379 | Redis | **极高** | 绝对禁止公网暴露 |
| MCP 端口 | MCP 工具 | **高** | 仅内网或完全禁止 |

---

## 4. 安全加固方案

### 4.1 阿里云安全组（最小端口开放）

| 方向 | 端口 | 授权对象 | 说明 |
|------|------|---------|------|
| 入站 | 2222（自定义 SSH） | 公司固定 IP/32 | SSH 仅允许指定 IP |
| 入站 | ALL | 0.0.0.0/0 | **默认拒绝所有入站** |
| 出站 | 443 | 0.0.0.0/0 | HTTPS（Anthropic API + npm） |
| 出站 | 80 | 0.0.0.0/0 | HTTP（包管理器下载） |

### 4.2 SSH 加固

```bash
# 1. 创建非 root 用户
sudo useradd -m -s /bin/bash claude-dev
sudo usermod -aG sudo claude-dev

# 2. 配置 SSH 密钥认证
ssh-keygen -t ed25519 -C "claude-dev-ecs"
ssh-copy-id -i ~/.ssh/claude_dev_ecs.pub claude-dev@your-ecs-ip -p 22

# 3. 加固 sshd_config
sudo tee -a /etc/ssh/sshd_config << 'EOF'
Port 2222
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
LoginGraceTime 30
AllowUsers claude-dev
X11Forwarding no
AllowTcpForwarding no
EOF
sudo systemctl restart sshd

# 4. 安装 Fail2Ban
sudo apt-get install -y fail2ban
sudo tee /etc/fail2ban/jail.local << 'EOF'
[sshd]
enabled = true
port = 2222
maxretry = 3
bantime = 3600
EOF
sudo systemctl enable --now fail2ban
```

### 4.3 Claude Code 权限限制

```json
// /opt/claude-workspace/.claude/settings.json
{
  "permissions": {
    "allow": [
      "Read", "Edit", "Write",
      "Bash(npm run *)", "Bash(npm test)", "Bash(git *)"
    ],
    "deny": [
      "Bash(rm -rf *)", "Bash(sudo *)",
      "Bash(curl * | bash)", "Bash(wget * | bash)",
      "Bash(chmod 777 *)", "Bash(env)", "Bash(printenv)",
      "Bash(* /etc/*)", "Bash(* /root/*)",
      "Bash(* /home/*/.*ssh*)",
      "Bash(curl http://100.100.100.200/*)"
    ]
  }
}
```

### 4.4 API Key 保护

```bash
# 方案 A：启动脚本注入（推荐）
cat > /opt/claude-workspace/start-claude.sh << 'EOF'
#!/bin/bash
export ANTHROPIC_API_KEY="$(cat /run/secrets/anthropic_key 2>/dev/null)"
cd /opt/claude-workspace
exec claude "$@"
EOF
chmod 700 /opt/claude-workspace/start-claude.sh

# 方案 B：使用阿里云 KMS 动态获取（企业级）
```

### 4.5 元数据服务加固

```bash
# 在阿里云控制台启用元数据加固模式
# 或 CLI：
aliyun ecs ModifyInstanceMetadataOptions \
  --InstanceId i-xxxx \
  --HttpEndpoint enabled \
  --HttpTokens required \
  --HttpPutResponseHopLimit 1
```

---

## 5. Agent Team 特殊风险

| 风险 | 说明 | 缓解措施 |
|------|------|---------|
| 同时编辑同一文件 | 后写入覆盖先写入 | 使用 Git Worktree 隔离 |
| 构建冲突 | 一个编译时另一个改源文件 | 任务划分确保文件不重叠 |
| Git 状态混乱 | 多 Agent 同时 git 操作 | 指定一个 Agent 负责 Git |
| npm install 冲突 | 多 Agent 同时安装依赖 | 预安装依赖后再启动 |

### Git Worktree 隔离（强烈推荐）

```bash
# 为每个 Agent 创建独立 worktree
git worktree add ../project-frontend feature/frontend-task
git worktree add ../project-backend  feature/backend-task

# 在各 worktree 中分别启动 Claude Code
cd ../project-frontend && claude
cd ../project-backend  && claude
```

---

## 6. Docker 容器化方案（推荐）

### Dockerfile

```dockerfile
FROM node:20-slim
RUN apt-get update && apt-get install -y git openssh-client ca-certificates && rm -rf /var/lib/apt/lists/*
RUN useradd -m -s /bin/bash claude-user
RUN npm install -g @anthropic-ai/claude-code
USER claude-user
WORKDIR /workspace
ENTRYPOINT ["claude"]
```

### docker-compose.yml

```yaml
version: '3.8'
services:
  claude-code:
    build: .
    container_name: claude-dev
    user: "1000:1000"
    volumes:
      - ./projects:/workspace:rw
    networks:
      - claude-net
    security_opt:
      - no-new-privileges:true
    deploy:
      resources:
        limits: { cpus: '2.0', memory: 4G }
    secrets:
      - anthropic_api_key

secrets:
  anthropic_api_key:
    file: ./secrets/anthropic_key.txt

networks:
  claude-net:
    driver: bridge
```

---

## 7. 安全检查清单

部署前必须完成：

- [ ] Claude Code 版本 ≥ 2.0.65
- [ ] SSH 已改非标准端口
- [ ] SSH 已禁用 root 登录
- [ ] SSH 已禁用密码认证（仅密钥）
- [ ] Fail2Ban 已安装运行
- [ ] 阿里云安全组已配置最小端口
- [ ] ECS 元数据服务已启用加固模式
- [ ] API Key 未硬编码
- [ ] Claude Code settings.json 已配置 deny 规则
- [ ] 项目目录独立于系统目录
- [ ] 已创建非 root 专用用户
- [ ] `.env` 文件已加入排除列表
- [ ] 已审计项目中的 `.claude/` 目录

## 8. 最终建议

| 部署方式 | 安全性 | 推荐场景 |
|---------|--------|---------|
| 直接在 ECS 裸机运行 | ⭐⭐ | 仅限个人临时使用 |
| **Docker 容器隔离** | **⭐⭐⭐⭐** | **✅ 推荐：日常开发** |
| Docker + 网络隔离 | ⭐⭐⭐⭐⭐ | ✅ 推荐：Agent Team |
| ACK/ACS 安全沙箱 | ⭐⭐⭐⭐⭐ | 企业级生产部署 |

**核心结论**：Claude Code 自身不需要开放任何入站端口，但它的 Shell 执行能力意味着一旦服务器被入侵，攻击者可能通过 Claude Code 获得更大的破坏面。**Docker 容器化隔离是性价比最高的安全方案**。
