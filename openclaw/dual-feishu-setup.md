# OpenClaw 双飞书账号隔离方案 - ben & candy

## 概述

通过 OpenClaw 的多 Agent 机制，实现 ben 和 candy 两个飞书账号的完全隔离，同时共享技能和升级。

## 环境信息

| 项目 | 值 |
|------|-----|
| ben (main) | `cli_a93753481738dbd6` |
| candy | `【待配置】` |
| 共享技能 | `~/.openclaw/skills/` |
| ben 工作区 | `~/.openclaw/workspace/` |
| candy 工作区 | `/mnt/tool/1-works/openclaw/workspace-candy/` |

---

## 当前目录结构

```
~/.openclaw/
├── openclaw.json           # 主配置文件
├── skills/                 # 共享技能
├── agents/
│   ├── main/agent/         # ben 认证
│   ├── main/sessions/      # ben 对话历史
│   ├── candy/agent/        # candy 认证
│   └── candy/sessions/      # candy 对话历史
└── workspace/              # ben 工作区

/mnt/tool/1-works/openclaw/
└── workspace-candy/        # candy 工作区
    ├── AGENTS.md
    ├── SOUL.md
    └── USER.md
```

---

## 待完成的配置

### openclaw.json 增量配置

需要在现有配置基础上添加：

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "name": "ben",
        "workspace": "~/.openclaw/workspace",
        "agentDir": "~/.openclaw/agents/main/agent"
      },
      {
        "id": "candy",
        "name": "candy",
        "workspace": "/mnt/tool/1-works/openclaw/workspace-candy",
        "agentDir": "~/.openclaw/agents/candy/agent"
      }
    ]
  },

  "channels": {
    "feishu": {
      "enabled": true,
      "defaultAccount": "main",
      "accounts": {
        "main": {
          "appId": "cli_a93753481738dbd6",
          "appSecret": "3rauSZh6Ul3atCdwvaz0gfRcHFEiDL2C"
        },
        "candy": {
          "appId": "【待填入】",
          "appSecret": "【待填入】",
          "dmPolicy": "pairing"
        }
      }
    }
  },

  "bindings": [
    {
      "agentId": "main",
      "match": {
        "channel": "feishu",
        "accountId": "main"
      }
    },
    {
      "agentId": "candy",
      "match": {
        "channel": "feishu",
        "accountId": "candy"
      }
    }
  ]
}
```

---

## 执行步骤

### 已完成
- [x] 创建 candy 工作区目录 `/mnt/tool/1-works/openclaw/workspace-candy/`
- [x] 复制工作区模板文件 (AGENTS.md, SOUL.md, USER.md)
- [x] 创建 candy agent 目录 `~/.openclaw/agents/candy/`
- [x] 更新 openclaw.json（agents.list, channels.feishu, bindings）
- [x] 重启网关生效

### 待完成

1. **配置 candy 飞书应用**
   - 访问 [飞书开放平台](https://open.feishu.cn/app) 创建企业自建应用
   - 获取 App ID 和 App Secret
   - 配置权限、事件订阅（与 ben 相同）
   - 发布应用

2. **更新 openclaw.json 中的 candy 飞书凭证**
   - 将 `【待配置】` 替换为实际的 App ID 和 App Secret

3. **重启网关**
   ```bash
   openclaw gateway restart
   ```

4. **配对测试**
   ```bash
   openclaw pairing list feishu
   openclaw pairing approve feishu <CODE>
   ```

---

## 验证隔离效果

```bash
# 查看 agents 和 bindings
openclaw agents list --bindings

# 查看飞书频道状态
openclaw channels status
```

---

## 共享说明

- **共享**：技能 (`~/.openclaw/skills/`)、OpenClaw 本体
- **隔离**：对话历史、工作区文件、认证信息

---

_创建时间: 2026-03-21_