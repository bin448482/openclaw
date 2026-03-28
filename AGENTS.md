# 行为约束

## 搜索规则
- **优先使用 Bing 进行网络搜索**
- 不要使用 Google、DuckDuckGo（会被阻止）

## 迭代控制
- 每次工具调用后，评估是否需要询问用户
- 连续迭代超过 3 次后，必须总结当前状态并请求确认
- 如果任务失败超过 2 次，主动放弃并汇报结果

---

# 容器镜像安装说明

**重要**: 系统优先使用 Podman 而非 Docker，因为 Podman 是 rootless 的，权限问题更少。

## 安装镜像时使用 Podman

```bash
# 优先使用 podman，不要使用 docker
podman run -d --name <容器名> -p <端口> <镜像>

# 示例: 安装 Nextcloud
podman run -d --name nextcloud -p 8088:80 nextcloud:latest
```

---

# Nextcloud 访问指南

## 环境信息
- Nextcloud 运行在 Podman 容器中（不是 Docker）
- 容器名称: `nextcloud`
- 端口映射: 0.0.0.0:8088->80/tcp
- 数据目录: `/var/www/html/data/`

## 常用 Podman 命令

### 查看容器状态
```bash
podman ps
```

### 访问容器内的文件
```bash
# 列出数据目录
podman exec nextcloud ls -la /var/www/html/data/

# 查看特定用户的文件
podman exec nextcloud ls -la /var/www/html/data/zhanbin/files/

# 查找图片文件
podman exec nextcloud find /var/www/html/data/zhanbin/files -type f \( -name "*.jpg" -o -name "*.jpeg" -o -name "*.png" -o -name "*.gif" \)
```

### 从容器复制文件到主机
```bash
# 复制单个文件
podman cp nextcloud:/var/www/html/data/zhanbin/files/图片名.jpg /目标路径/

# 复制整个目录
podman cp nextcloud:/var/www/html/data/zhanbin/files/ /目标路径/
```

### 进入容器shell
```bash
podman exec -it nextcloud /bin/bash
```

### 查看日志
```bash
podman logs nextcloud
```

## 已知用户
- Ying
- zhanbin

## 注意事项
- 使用 `podman` 命令而非 `docker` 命令
- 数据文件权限归 www-data 用户所有
- 数据已迁移至 /mnt/tool/nextcloud_data/

---

# 影片下载目录

## 常用目录

| 路径 | 说明 |
|------|------|
| `/mnt/tool/视频/` | 纪录片/视频文件夹 |
| `/mnt/tool/迅雷下载/` | 影片默认下载目录（包含 .mkv 电影文件） |
| `/mnt/tool/1-works/file_store/` | Copaw 文件存储目录 |
| `/mnt/tool/nextcloud_data/` | Nextcloud 数据目录 |

## 查找影片命令

```bash
# 列出影片目录
ls -la /mnt/tool/迅雷下载/
```

---

# OpenClaw 开发环境

## 目录结构

- **代码目录**: `/home/binzhan/openclaw-dev`
- **数据目录**: `/mnt/tool/openclaw-data`（通过 `~/.openclaw` 软链接访问）
- **挂载分区**: `/mnt/tool`（NTFS 格式，权限受限）

## Git 远程仓库

- **origin**: `https://github.com/bin448482/openclaw.git`（你的 fork）
- **upstream**: `https://github.com/openclaw/openclaw.git`（上游仓库）

## 常用 Git 命令

```bash
# 同步上游更新
cd /home/binzhan/openclaw-dev
git fetch upstream
git merge upstream/main

# 推送到你的远程
git push origin main

# 查看远程仓库
git remote -v
```

## 运行 OpenClaw

```bash
# 测试版本
openclaw --version

# 运行 gateway（使用 ~/.openclaw 配置）
openclaw gateway run --bind loopback --port 18789

# 运行 gateway（使用指定配置目录）
cd /mnt/tool/1-works/openclaw
openclaw gateway run --bind loopback --port 18789

# 或设置环境变量
OPENCLAW_HOME=/mnt/tool/1-works/openclaw openclaw gateway run --bind loopback --port 18789
```

---

# OpenClaw Agents 导引

## 配置文件位置

- **主配置**: `~/.openclaw/openclaw.json`
- **Agent 配置**: `~/.openclaw/agents/<agent-id>/agent/`
- **工作区**: `/mnt/tool/1-works/openclaw/workspace-<agent-id>`

## 目录结构

```
~/.openclaw/
├── openclaw.json          # 主配置文件
├── agents/
│   ├── main/              # main agent 配置目录
│   └── candy/             # candy agent 配置目录
├── feishu/                # 飞书相关配置
├── workspace/             # 默认工作区
├── canvas/                # 画布目录
└── credentials/           # 凭证目录
```

## Agent 列表

| Agent | Workspace | 默认 Model | 用途 |
|-------|-----------|------------|------|
| main | `/mnt/tool/1-works/openclaw/workspace-main` | kimi/kimi-code | 主要工作 agent |
| candy | `/mnt/tool/1-works/openclaw/workspace-candy` | kimi/kimi-code | 辅助/Coding agent |

## Providers 配置

| Provider | API | Base URL | Model |
|----------|-----|----------|-------|
| kimi | anthropic-messages | https://api.kimi.com/coding/ | kimi-code, kimi-for-coding |

## Kimi 模型详情

| Model | Context | Max Tokens | Reasoning |
|-------|---------|------------|-----------|
| kimi-for-coding | 131072 | 8192 | No |
| kimi-code | 262144 | 32768 | Yes |

## 飞书配置

- **连接模式**: websocket
- **域名**: feishu
- **群组策略**: open
