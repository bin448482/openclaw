# OpenClaw 配置飞书指南

OpenClaw 原生支持飞书（Feishu）频道，可以通过飞书与 AI 助手对话。

## 方式一：向导配置（推荐）

```bash
openclaw onboard
# 或
openclaw channels add
```

选择 **Feishu**，输入 App ID 和 App Secret 向导会引导完成配置。

## 方式二：手动配置

编辑 `~/.openclaw/openclaw.json`：

```json
{
  "channels": {
    "feishu": {
      "enabled": true,
      "dmPolicy": "pairing",
      "accounts": {
        "main": {
          "appId": "cli_xxx",
          "appSecret": "你的App Secret",
          "botName": "My AI assistant"
        }
      }
    }
  }
}
```

## 飞书应用创建步骤

### 1. 创建应用

访问 [飞书开放平台](https://open.feishu.cn/app) 或 [Lark 国际版](https://open.larksuite.com/app)，登录后创建企业应用。

### 2. 获取凭证

在应用的 **凭证与基础信息** 中获取：
- **App ID**（格式：`cli_xxx`）
- **App Secret**

### 3. 配置权限

在 **权限** 页面点击 **批量导入**，粘贴以下权限：

```json
{
  "scopes": {
    "tenant": [
      "aily:file:read",
      "aily:file:write",
      "application:application.app_message_stats.overview:readonly",
      "application:application:self_manage",
      "application:bot.menu:write",
      "cardkit:card:read",
      "cardkit:card:write",
      "contact:user.employee_id:readonly",
      "corehr:file:download",
      "event:ip_list",
      "im:chat.access_event.bot_p2p_chat:read",
      "im:chat.members:bot_access",
      "im:message",
      "im:message.group_at_msg:readonly",
      "im:message.p2p_msg:readonly",
      "im:message:readonly",
      "im:message:send_as_bot",
      "im:resource"
    ],
    "user": ["aily:file:read", "aily:file:write", "im:chat.access_event.bot_p2p_chat:read"]
  }
}
```

### 4. 启用机器人能力

在 **应用能力** > **机器人** 中：
1. 启用机器人能力
2. 设置机器人名称

### 5. 配置事件订阅

1. 进入 **事件订阅**
2. 选择 **使用长连接接收事件**（WebSocket）
3. 添加事件：`im.message.receive_v1`

> 注意：配置事件订阅前需确保 gateway 正在运行

### 6. 发布应用

1. 在 **版本管理与发布** 创建版本
2. 提交审核并发布
3. 等待管理员批准（企业应用通常自动通过）

## 启动与测试

```bash
# 启动 gateway
openclaw gateway

# 在飞书中给机器人发送消息

# 审批配对
openclaw pairing approve feishu <CODE>
```

审批后即可正常对话。

## 常用命令

| 命令 | 说明 |
|------|------|
| `openclaw gateway status` | 查看 gateway 状态 |
| `openclaw gateway restart` | 重启 gateway |
| `openclaw logs --follow` | 查看日志 |
| `openclaw pairing list feishu` | 查看配对请求 |
| `openclaw pairing approve feishu <CODE>` | 审批配对 |

## 访问控制

### 私信配置

- 默认 `dmPolicy: "pairing"`，未知用户需配对
- 可设置为 `"allowlist"`（仅允许列表用户）或 `"open"`（允许所有人）

### 群聊配置

```json
{
  "channels": {
    "feishu": {
      "groupPolicy": "open",           // open/allowlist/disabled
      "groupAllowFrom": ["oc_xxx"],    // 允许的群组
      "groups": {
        "oc_xxx": {
          "requireMention": true        // 是否需要@机器人
        }
      }
    }
  }
}
```

## 高级配置

### Lark 国际版

如使用国际版飞书（Lark），需设置 domain：

```json
{
  "channels": {
    "feishu": {
      "domain": "lark"
    }
  }
}
```

### 多账户

```json
{
  "channels": {
    "feishu": {
      "defaultAccount": "main",
      "accounts": {
        "main": {
          "appId": "cli_xxx",
          "appSecret": "xxx"
        },
        "backup": {
          "appId": "cli_yyy",
          "appSecret": "yyy",
          "enabled": false
        }
      }
    }
  }
}
```

### 消息限额

- `textChunkLimit`：文本分块大小（默认 2000）
- `mediaMaxMb`：媒体文件大小限制（默认 30MB）

## 相关链接

- [OpenClaw 官方文档](https://docs.openclaw.ai)
- [飞书频道配置文档](https://docs.openclaw.ai/channels/feishu)
- [飞书开放平台](https://open.feishu.cn)
