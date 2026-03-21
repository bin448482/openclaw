# OpenClaw 飞书远程管理部署计划

## 项目概述

在家庭内网服务器上部署 OpenClaw，通过飞书机器人实现远程管理，支持下载资源、分析网页信息等功能。

## 环境信息

- **主机**: 家庭内网服务器（无公网 IP）
- **操作系统**: Linux
- **用户**: binzhan
- **网络**: 出向连接正常，无需内网穿透
- **连接方式**: WebSocket 长连接（飞书主动连接 OpenClaw）

## 目标功能

1. ✅ 通过飞书发送消息控制服务器
2. ✅ 执行 shell 命令（下载工具如 aria2/wget）
3. ✅ 访问本地文件系统（管理下载的文件）
4. ✅ 网页分析和信息提取

---

## 执行步骤

### Phase 1: 环境准备

#### 1.1 检查 Node.js 版本
```bash
node --version
```

**要求**: Node 24 (推荐) 或 Node 22 LTS (22.16+)

**如果不符合**:
```bash
# 使用 nvm 安装 Node 24
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
nvm install 24
nvm use 24
```

> **注意**: 安装后需要重新加载 shell 配置或使用 `export` 命令使 nvm 生效。建议将 nvm 初始化命令添加到 `~/.bashrc`。

#### 1.2 安装 OpenClaw
```bash
npm install -g openclaw@latest
```

验证安装:
```bash
openclaw --version
```

---

### Phase 2: 飞书应用配置

#### 2.1 创建飞书应用

1. 访问 [飞书开放平台](https://open.feishu.cn/app)
2. 登录后点击 **创建企业应用**
3. 填写应用名称（如 "HomeServerBot"）
4. 选择应用图标

#### 2.2 获取凭证

在 **凭证与基础信息** 页面获取：
- **App ID** (格式: `cli_xxx`)
- **App Secret**

**保存好这些信息，后续配置需要**

#### 2.3 配置权限

在 **权限管理** 页面，点击 **批量导入**，粘贴：

```json
{
  "scopes": {
    "tenant": [
      "im:message",
      "im:message:send_as_bot",
      "im:message.p2p_msg:readonly",
      "im:message.group_at_msg:readonly",
      "im:message:readonly",
      "im:chat.access_event.bot_p2p_chat:read",
      "im:chat.members:bot_access",
      "im:resource"
    ]
  }
}
```

> 注：已精简权限，只保留消息相关必要权限

#### 2.4 启用机器人能力

1. 进入 **应用能力** > **机器人**
2. 启用机器人能力
3. 设置机器人名称（如 "服务器助手"）

#### 2.5 配置事件订阅

1. 进入 **事件与回调** > **事件订阅**
2. 选择 **使用长连接接收事件** (WebSocket)
3. 添加事件：`im.message.receive_v1`

> ⚠️ **重要**: 配置事件订阅前确保 OpenClaw gateway 已启动

#### 2.6 发布应用

1. 进入 **版本管理与发布**
2. 创建版本（填写版本号、更新说明）
3. 提交审核并发布
4. 等待管理员批准（企业应用通常自动通过）

---

### Phase 3: OpenClaw 配置

#### 3.1 初始化配置

运行配置向导：
```bash
openclaw onboard
```

或手动添加飞书频道：
```bash
openclaw channels add
# 选择 Feishu，输入 App ID 和 App Secret
```

#### 3.2 配置 exec 工具权限

编辑配置文件 `~/.openclaw/openclaw.json`：

```json5
{
  channels: {
    feishu: {
      enabled: true,
      dmPolicy: "pairing",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "你的App Secret",
          botName: "服务器助手"
        }
      }
    }
  },
  tools: {
    exec: {
      host: "gateway",
      security: "allowlist",
      ask: "on-miss",
      pathPrepend: ["/home/binzhan/bin", "/usr/local/bin"]
    }
  },
  agents: {
    list: [
      {
        id: "main",
        model: "gpt-4o",  // 或其他你拥有的模型
        provider: "openai" // 或其他提供商
      }
    ]
  }
}
```

#### 3.3 配置 AI 模型提供商

需要设置 API Key。可以通过环境变量或配置文件：

```bash
# 方式1: 环境变量
export OPENAI_API_KEY="your-api-key"

# 方式2: 使用 openclaw config
openclaw config set providers.openai.apiKey "your-api-key"
```

支持的提供商：OpenAI、Anthropic、Moonshot、Qwen 等

---

### Phase 4: 启动与测试

#### 4.1 启动 Gateway

```bash
# 前台运行（测试时使用）
openclaw gateway

# 或安装为服务并启动
openclaw gateway install
openclaw gateway start
```

#### 4.2 检查状态

```bash
# 查看 gateway 状态
openclaw gateway status

# 查看日志
openclaw logs --follow
```

#### 4.3 飞书配对

1. 在飞书中找到你的机器人，发送任意消息
2. 机器人会回复配对码
3. 在服务器上执行：
```bash
openclaw pairing approve feishu <配对码>
```

#### 4.4 功能测试

在飞书中测试以下功能：

**测试 1: 基本对话**
```
你好，请介绍一下你自己
```

**测试 2: 执行命令**
```
请执行命令：ls -la
```

**测试 3: 下载文件**
```
请使用 wget 下载这个文件：https://example.com/file.zip 到 ~/Downloads/
```

**测试 4: 网页分析**
```
请访问 https://news.ycombinator.com 并总结今天的头条新闻
```

---

### Phase 5: 生产环境配置

#### 5.1 配置 systemd 服务（可选）

创建服务文件 `/etc/systemd/system/openclaw.service`：

```ini
[Unit]
Description=OpenClaw Gateway
After=network.target

[Service]
Type=simple
User=binzhan
Environment="OPENAI_API_KEY=your-api-key"
ExecStart=/usr/local/bin/openclaw gateway
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

启用服务：
```bash
sudo systemctl daemon-reload
sudo systemctl enable openclaw
sudo systemctl start openclaw
```

#### 5.2 配置防火墙（如果需要）

OpenClaw 使用出向 WebSocket 连接，通常不需要开放入向端口。但如果配置了 webhook 模式：

```bash
# 开放 webhook 端口（默认 3000）
sudo ufw allow 3000/tcp
```

#### 5.3 日志轮转

配置 logrotate 防止日志文件过大：

```bash
sudo tee /etc/logrotate.d/openclaw << 'EOF'
/home/binzhan/.openclaw/logs/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 644 binzhan binzhan
}
EOF
```

---

## 安全配置建议

### 访问控制

```json5
{
  channels: {
    feishu: {
      dmPolicy: "allowlist",  // 只允许特定用户
      allowFrom: ["ou_xxx"],  // 你的飞书 Open ID
      groupPolicy: "disabled" // 禁用群聊功能（可选）
    }
  }
}
```

### Exec 工具安全

```json5
{
  tools: {
    exec: {
      host: "gateway",
      security: "allowlist",  // 只允许特定命令
      ask: "always"          // 每次执行前询问确认
    }
  }
}
```

配置命令白名单 `~/.openclaw/exec-approvals.json`：

```json
{
  "allowedCommands": [
    "/usr/bin/wget",
    "/usr/bin/curl",
    "/usr/bin/aria2c",
    "/usr/bin/ls",
    "/usr/bin/cat",
    "/usr/bin/tail"
  ]
}
```

---

## 故障排查

### 常见问题

**1. Gateway 无法启动**
```bash
# 检查日志
openclaw logs --follow

# 检查配置
openclaw doctor
```

**2. 飞书收不到消息**
- 确认应用已发布并通过审核
- 确认事件订阅配置正确
- 检查 `im.message.receive_v1` 事件是否添加
- 查看日志确认 WebSocket 连接状态

**3. 命令执行失败**
- 检查 exec 工具配置
- 确认命令在白名单中
- 检查文件权限

**4. 配对失败**
```bash
# 查看待处理的配对请求
openclaw pairing list feishu

# 手动批准
openclaw pairing approve feishu <CODE>
```

---

## 进阶功能

### 多代理配置

可以为不同任务配置不同的 AI 代理：

```json5
{
  agents: {
    list: [
      {
        id: "main",
        model: "gpt-4o",
        provider: "openai"
      },
      {
        id: "download-agent",
        model: "gpt-4o-mini",
        provider: "openai",
        systemPrompt: "你是一个专门负责下载任务的助手..."
      }
    ]
  },
  bindings: [
    {
      agentId: "download-agent",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_xxx" }
      }
    }
  ]
}
```

### 自动化任务

使用 cron 功能设置定时任务：

```bash
openclaw cron add "0 2 * * *" "清理下载目录" "exec rm -rf ~/Downloads/temp/*"
```

---

## 相关资源

- [OpenClaw 官方文档](https://docs.openclaw.ai)
- [飞书开放平台](https://open.feishu.cn)
- [OpenClaw GitHub](https://github.com/openclaw)

---

## 执行检查清单

- [ ] Node.js 版本检查/安装
- [ ] OpenClaw 安装
- [ ] 飞书应用创建
- [ ] 飞书权限配置
- [ ] 飞书事件订阅配置
- [ ] 飞书应用发布
- [ ] OpenClaw 飞书频道配置
- [ ] AI 模型 API Key 配置
- [ ] Exec 工具权限配置
- [ ] Gateway 启动
- [ ] 飞书配对
- [ ] 基本功能测试
- [ ] 下载功能测试
- [ ] 网页分析功能测试
- [ ] 生产环境配置（systemd 等）

---

**计划生成时间**: 2026-03-14
**执行优先级**: 高
**预计完成时间**: 2-3 小时
