# Feishu 群组配置指南

本文档说明如何在 Feishu 群里配置机器人的响应行为（是否需要 @mention）。

---

## 获取群 ID

机器人 ID 格式为 `oc_xxx`。

### 方法一：看日志（推荐）

1. 把机器人拉进群
2. 在群里 @ 机器人发一条消息
3. 查看日志：

```bash
openclaw logs --follow
```

日志中会打印 `chat_id`，格式为 `oc_xxx`。

### 方法二：Feishu API 调试

使用 Feishu Open Platform 的 API 调试工具列出群组信息。

---

## 配置响应行为

编辑 `~/.openclaw/openclaw.json`，在 `channels.feishu.groups` 下配置。

### 基础配置结构

```json
{
  "channels": {
    "feishu": {
      "groups": {
        "<群ID>": { "requireMention": true或false }
      }
    }
  }
}
```

### 示例一：所有群都不用 @mention 就能响应

```json
{
  "channels": {
    "feishu": {
      "groups": {
        "*": { "requireMention": false }
      }
    }
  }
}
```

### 示例二：所有群都不用 @mention，但某个群需要 @mention

```json
{
  "channels": {
    "feishu": {
      "groups": {
        "*": { "requireMention": false },
        "oc_群A的ID": { "requireMention": true }
      }
    }
  }
}
```

### 示例三：默认要 @mention，只对特定群关闭

```json
{
  "channels": {
    "feishu": {
      "groups": {
        "*": { "requireMention": true },
        "oc_群B的ID": { "requireMention": false }
      }
    }
  }
}
```

---

## 优先级规则

具体群 ID 的配置会覆盖 `"*"` 的默认配置。

| 配置 | 效果 |
|------|------|
| `"*": { requireMention: false }` | 所有群都不用 @mention |
| `"oc_xxx": { requireMention: true }` | 具体某个群需要 @mention |

---

## 生效

修改配置后需要重启 gateway：

```bash
openclaw gateway restart
```

重启后查看状态确认生效：

```bash
openclaw gateway status
```
