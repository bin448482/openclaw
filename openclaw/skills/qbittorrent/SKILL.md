---
name: qbittorrent
description: "提供 qBittorrent Web UI 的访问信息、配置文件路径、下载目录等。当用户询问 qBittorrent 相关问题或需要管理下载任务时使用。"
metadata:
  {
    "copaw":
      {
        "emoji": "📥",
        "requires": {}
      }
  }
---

# qBittorrent 参考

## 访问信息

| 项目 | 值 |
|------|-----|
| Web UI 地址 | http://192.168.71.63:8080 |
| 用户名 | admin |
| 密码 | monkey1216 |

## 配置文件

- **配置文件路径**: `~/.config/qBittorrent/qBittorrent.conf`
- **数据配置**: `~/.config/qBittorrent/qBittorrent-data.conf`

## 常用路径

- **下载目录**: `/mnt/tool/迅雷下载`
- **BT 监听端口**: 27209

## 常用操作

### 启动 qBittorrent

```bash
qbittorrent --no-splash --webui-port=8080
```

### 查看运行状态

```bash
ps aux | grep qbit
ss -tlnp | grep 8080
```

### 关闭 qBittorrent

```bash
pkill qbittorrent
```

## 文档

详细配置说明见：`/mnt/tool/1-works/docs/qbittorrent-webui.md`
