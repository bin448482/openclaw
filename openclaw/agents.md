# OpenClaw Session 清理指南

## Session 存储位置

Session 保存在 JSON 文件中，**不需要清理数据库**（SQLite 仅用于 QMD 索引）：

```
~/.openclaw/agents/<agentId>/sessions/sessions.json
~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl
```

## 清理命令

### 预览清理内容（dry-run）

```bash
openclaw sessions cleanup --dry-run
```

### 执行清理

```bash
openclaw sessions cleanup --enforce
```

### 修复缺失的 transcript 条目

```bash
openclaw sessions cleanup --fix-missing
```

### 针对特定 agent

```bash
openclaw sessions cleanup --agent <agent-name> --enforce
```

### 针对所有 agent

```bash
openclaw sessions cleanup --all-agents --enforce
```

## 自动清理配置

在 `openclaw.json` 中配置 `session.maintenance`：

| 配置 | 默认值 | 说明 |
|------|--------|------|
| `mode` | `"warn"` | `"warn"` 仅报告，`"enforce"` 自动清理 |
| `pruneAfter` | `30d` | 清理超过此时间的 stale 条目 |
| `maxEntries` | `500` | sessions.json 条目上限 |
| `rotateBytes` | `10mb` | sessions.json 超过此大小则轮转 |

## 其他相关命令

### 列出所有 session

```bash
# 列出所有 session
openclaw sessions

# 列出特定 agent 的 session
openclaw sessions --agent <agent-name>

# 只显示最近 2 小时活跃的 session
openclaw sessions --active 120
```

### 通过 RPC 删除 session

- **`sessions.delete`** - 删除指定 session 及其 transcript
- **`sessions.reset`** - 重置 session（创建新 sessionId，存档旧 transcript）

## Cron Session Reaper

OpenClaw 有自动清理 cron session 的 cron job：

- 默认保留期：**24 小时**
- 运行频率：每 5 分钟
- 可通过 `cron.sessionRetention` 配置，设为 `false` 可禁用
