# OpenClaw 双飞书账号隔离方案

## 概述

通过 OpenClaw 的多 Agent 机制，实现两个飞书账号的完全隔离，同时共享技能和升级。

## 架构设计

```
┌─────────────────────────────────────────────────────────────────┐
│                      OpenClaw Gateway                           │
│                                                                 │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐     │
│  │  Agent-A     │    │  Agent-B     │    │   Skills     │     │
│  │ (张三账号)    │    │ (李四账号)    │    │  (共享)       │     │
│  │              │    │              │    │              │     │
│  │ workspace-A  │    │ workspace-B  │    │ browser_vis  │     │
│  │ sessions-A   │    │ sessions-B   │    │ cron         │     │
│  │ auth-A       │    │ auth-B       │    │ docx/pdf/... │     │
│  └──────────────┘    └──────────────┘    └──────────────┘     │
│         ▲                  ▲                                      │
│         │                  │                                      │
│  ┌──────────────────────────────────────┐                        │
│  │           Feishu Channel             │                        │
│  │  ┌─────────────┐  ┌─────────────┐   │                        │
│  │  │ account-A   │  │ account-B   │   │                        │
│  │  │ cli_xxx1    │  │ cli_xxx2    │   │                        │
│  │  └─────────────┘  └─────────────┘   │                        │
│  └──────────────────────────────────────┘                        │
└─────────────────────────────────────────────────────────────────┘
                    │                    │
                    ▼                    ▼
             ┌──────────┐         ┌──────────┐
             │ 飞书账号A │         │ 飞书账号B │
             │  (张三)   │         │  (李四)   │
             └──────────┘         └──────────┘
```

## 隔离内容

| 内容 | 隔离方式 |
|------|----------|
| 对话历史 | 每个 agent 独立的 `sessions/` 目录 |
| 认证信息 | 每个 agent 独立的 `auth-profiles.json` |
| 工作区文件 | 每个 agent 独立的 `workspace/` 目录 |
| 技能配置 | 共享 `~/.openclaw/skills/` 目录 |
| 系统升级 | 共享 OpenClaw 本体，通过 npm 更新 |

## 配置步骤

### 1. 准备工作

确保已安装的共享技能：

```bash
~/.openclaw/skills/          # 共享技能目录
├── browser_visible/
├── cron/
├── docx/
├── file_reader/
├── news/
├── pdf/
├── pptx/
├── qbittorrent/
└── xlsx/
```

### 2. 创建第二个飞书应用

1. 访问 [飞书开放平台](https://open.feishu.cn/app)
2. 创建第二个企业自建应用
3. 获取 `App ID` 和 `App Secret`
4. 配置权限、事件订阅（与第一个应用相同）

### 3. 创建第二个 Agent 工作区

```bash
# 创建 agent-B 的工作区
mkdir -p ~/.openclaw/workspace-lisi
mkdir -p ~/.openclaw/agents/lisi/agent
mkdir -p ~/.openclaw/agents/lisi/sessions

# 复制基础配置
cp ~/.openclaw/workspace/AGENTS.md ~/.openclaw/workspace-lisi/
cp ~/.openclaw/workspace/SOUL.md ~/.openclaw/workspace-lisi/
cp ~/.openclaw/workspace/USER.md ~/.openclaw/workspace-lisi/
```

### 4. 更新配置文件

编辑 `~/.openclaw/openclaw.json`：

```json5
{
  // 代理配置
  "agents": {
    "defaults": {
      "model": {
        "primary": "minimax-portal/MiniMax-M2.7"
      },
      "models": {
        "minimax-portal/MiniMax-M2.7": { "alias": "minimax-m2.7" },
        "minimax-portal/MiniMax-M2.7-highspeed": { "alias": "minimax-m2.7-highspeed" },
        "minimax-portal/MiniMax-M2.5": { "alias": "minimax-m2.5" },
        "minimax-portal/MiniMax-M2.5-highspeed": { "alias": "minimax-m2.5-highspeed" },
        "minimax-portal/MiniMax-M2.5-Lightning": { "alias": "minimax-m2.5-lightning" }
      },
      "compaction": { "mode": "safeguard" }
    },
    "list": [
      {
        "id": "zhangsan",
        "name": "张三",
        "workspace": "~/.openclaw/workspace-zhangsan",
        "agentDir": "~/.openclaw/agents/zhangsan/agent"
      },
      {
        "id": "lisi",
        "name": "李四",
        "workspace": "~/.openclaw/workspace-lisi",
        "agentDir": "~/.openclaw/agents/lisi/agent"
      }
    ]
  },

  // 飞书频道配置 - 多账号
  "channels": {
    "feishu": {
      "enabled": true,
      "defaultAccount": "zhangsan",
      "accounts": {
        "zhangsan": {
          "appId": "cli_a93753481738dbd6",        // 替换为实际 App ID
          "appSecret": "your-app-secret-1",         // 替换为实际 App Secret
          "botName": "张三助手",
          "dmPolicy": "pairing"
        },
        "lisi": {
          "appId": "cli_xxxxxxxxxxxxxx",           // 替换为第二个应用 App ID
          "appSecret": "your-app-secret-2",         // 替换为第二个应用 App Secret
          "botName": "李四助手",
          "dmPolicy": "pairing"
        }
      }
    }
  },

  // 绑定规则 - 路由配置
  "bindings": [
    {
      "agentId": "zhangsan",
      "match": {
        "channel": "feishu",
        "accountId": "zhangsan"
      }
    },
    {
      "agentId": "lisi",
      "match": {
        "channel": "feishu",
        "accountId": "lisi"
      }
    }
  ],

  // 网关配置
  "gateway": {
    "mode": "local",
    "auth": {
      "mode": "token",
      "token": "b2549bb08afb9f62dff1f28a4a837dbbe6d1c1c95b44f5b3"
    }
  },

  // 插件配置
  "plugins": {
    "entries": {
      "minimax": { "enabled": true },
      "feishu": { "enabled": true }
    }
  }
}
```

### 5. 使用 OpenClaw 工具创建 Agent（推荐）

```bash
# 创建 agent
openclaw agents add zhangsan
openclaw agents add lisi

# 查看 agents
openclaw agents list

# 查看 bindings
openclaw agents list --bindings
```

### 6. 重启网关

```bash
openclaw gateway restart
```

### 7. 配置第二个飞书应用

在飞书开放平台完成第二个应用的配置：

1. **权限配置**：批量导入权限（与第一个应用相同）
2. **事件订阅**：启用长连接，添加 `im.message.receive_v1` 事件
3. **发布应用**：创建版本并提交发布

### 8. 配对测试

```bash
# 查看配对请求
openclaw pairing list feishu

# 批准配对
openclaw pairing approve feishu <CODE>
```

## 目录结构

配置完成后，目录结构如下：

```
~/.openclaw/
├── openclaw.json           # 主配置文件
├── skills/                 # 共享技能（所有 agent 共用）
│   ├── browser_visible/
│   ├── cron/
│   ├── docx/
│   ├── file_reader/
│   ├── news/
│   ├── pdf/
│   ├── pptx/
│   ├── qbittorrent/
│   └── xlsx/
├── agents/                 # Agent 状态目录
│   ├── zhangsan/
│   │   ├── agent/          # auth-profiles.json 等
│   │   └── sessions/       # 对话历史（隔离）
│   └── lisi/
│       ├── agent/
│       └── sessions/
├── workspace-zhangsan/     # Agent A 工作区（隔离）
│   ├── AGENTS.md
│   ├── SOUL.md
│   └── USER.md
└── workspace-lisi/         # Agent B 工作区（隔离）
    ├── AGENTS.md
    ├── SOUL.md
    └── USER.md
```

## 验证隔离效果

### 1. 检查 Agent 列表

```bash
openclaw agents list --bindings
```

预期输出：
```
┌───────────┬─────────────┬──────────────────┐
│ Agent ID  │ Name        │ Bindings         │
├───────────┼─────────────┼──────────────────┤
│ zhangsan  │ 张三        │ feishu/zhangsan  │
│ lisi      │ 李四        │ feishu/lisi      │
└───────────┴─────────────┴──────────────────┘
```

### 2. 检查飞书频道状态

```bash
openclaw channels status
```

### 3. 测试对话隔离

1. 用飞书账号A 给 Bot-A 发消息
2. 用飞书账号B 给 Bot-B 发消息
3. 确认两边对话历史独立

## 技能共享说明

### 安装新技能

新技能安装到共享目录，所有 agent 均可使用：

```bash
# 安装到共享目录
cp -r /path/to/new-skill ~/.openclaw/skills/

# 重启网关
openclaw gateway restart
```

### 目录优先级

Agent 技能加载顺序：
1. `~/.openclaw/agents/<agentId>/workspace/skills/`（per-agent）
2. `~/.openclaw/skills/`（共享）
3. `~/.openclaw/workspace/skills/`（默认工作区）

## 系统升级

所有 agent 共享同一个 OpenClaw 安装，升级后所有 agent 同时生效：

```bash
# 升级 OpenClaw
npm install -g openclaw@latest

# 重启网关
openclaw gateway restart

# 检查版本
openclaw --version
```

## 常见问题

### Q: 如何为不同 agent 设置不同模型？

在 `agents.list[].model` 中单独设置：

```json5
{
  "agents": {
    "list": [
      {
        "id": "zhangsan",
        "model": "minimax-portal/MiniMax-M2.7"
      },
      {
        "id": "lisi",
        "model": "minimax-portal/MiniMax-M2.5"
      }
    ]
  }
}
```

### Q: 如何限制某个 agent 的工具权限？

使用 `tools.allow` 和 `tools.deny`：

```json5
{
  "agents": {
    "list": [
      {
        "id": "lisi",
        "tools": {
          "allow": ["read", "sessions_list", "sessions_history"],
          "deny": ["browser", "exec", "write"]
        }
      }
    ]
  }
}
```

### Q: 如何迁移现有 agent 的数据？

复制对应的目录：

```bash
# 迁移 agent 数据
cp -r ~/.openclaw/agents/old-agent ~/.openclaw/agents/new-agent
cp -r ~/.openclaw/workspace-old ~/.openclaw/workspace-new
```

## 配置参考

- [OpenClaw 多 Agent 文档](https://docs.openclaw.ai/concepts/multi-agent)
- [飞书频道配置](https://docs.openclaw.ai/channels/feishu)
- [完整配置参考](https://docs.openclaw.ai/gateway/configuration-reference)
