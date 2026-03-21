# CoPaw 工作目录配置记录

## 概述

CoPaw 默认使用 `~/.copaw` 作为工作目录。为将工作目录迁移到 `/mnt/tool/1-works`，进行了以下配置。

---

## 配置步骤

### 1. 创建 systemd 用户服务

路径: `~/.config/systemd/user/copaw.service`

```ini
[Unit]
Description=CoPaw AI Assistant
After=network.target

[Service]
Type=simple
ExecStart=/home/binzhan/.copaw/bin/copaw app --host 127.0.0.1 --port 8088
WorkingDirectory=/home/binzhan/.copaw
Restart=always
RestartSec=10
Environment=COPAW_HOME=/home/binzhan/.copaw
Environment=COPAW_WORKING_DIR=/mnt/tool/1-works

[Install]
WantedBy=default.target
```

启用服务:
```bash
systemctl --user daemon-reload
systemctl --user enable copaw.service
systemctl --user start copaw.service
```

### 2. 复制配置文件

将原有配置复制到新工作目录:

```bash
cp ~/.copaw/config.json /mnt/tool/1-works/config.json
cp -r ~/.copaw/providers /mnt/tool/1-works/
```

### 3. 添加 LLM Provider

使用 Aliyun Coding Plan (内置 provider, 已包含 MiniMax):

```bash
export COPAW_WORKING_DIR=/mnt/tool/1-works
export COPAW_HOME=/home/binzhan/.copaw
export MINIMAX_API_KEY="your-api-key"

# 列出可用模型
copaw models list

# 或添加自定义 provider
copaw models add-provider minimax --name MiniMax --base-url "https://api.minimax.chat/v1"
copaw models add-model minimax --model-id "MiniMax-M2.5" --model-name "MiniMax-M2.5"

# 设置默认模型
copaw models set-llm
```

### 4. Shell 环境变量

在 `~/.bashrc` 中添加:

```bash
export COPAW_HOME=/home/binzhan/.copaw
export COPAW_WORKING_DIR=/mnt/tool/1-works
```

---

## 当前配置状态

| 项目 | 值 |
|------|-----|
| 工作目录 | `/mnt/tool/1-works` |
| LLM Provider | Aliyun Coding Plan (minimax) |
| 模型 | MiniMax-M2.5 |
| 飞书 | 已配置 |
| 端口 | 127.0.0.1:8088 |
| 开机自启 | 已启用 |

---

## 常用命令

```bash
# 查看状态
systemctl --user status copaw.service

# 重启服务
systemctl --user restart copaw.service

# 查看日志
journalctl --user -u copaw.service -f

# 查看模型列表
export COPAW_WORKING_DIR=/mnt/tool/1-works
~/.copaw/bin/copaw models list
```

---

## 配置文件位置

- CoPaw 配置: `/mnt/tool/1-works/config.json`
- CoPaw 日志: `~/.copaw/copaw.log`
- Providers: `/mnt/tool/1-works/providers/`
- Sessions: `~/.copaw/sessions/`
