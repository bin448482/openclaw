# CasaOS 使用说明

## 概述

CasaOS 是一个基于 Docker 的开源家庭云系统，提供简洁的 Web 界面来管理应用、文件和系统。

## 系统信息

- **版本**: v0.4.15
- **Web 访问地址**: http://localhost:8088
- **配置文件**: /etc/casaos/casaos.conf
- **数据目录**: /var/lib/casaos

## 常用命令

### 服务管理

```bash
# 查看服务状态
systemctl status casaos

# 启动 CasaOS
sudo systemctl start casaos

# 停止 CasaOS
sudo systemctl stop casaos

# 重启 CasaOS
sudo systemctl restart casaos

# 查看运行时日志
journalctl -u casaos -f
```

### CLI 操作

```bash
# 查看版本
casaos -v

# 指定配置文件启动
casaos -c /path/to/config.conf

# 指定数据库路径
casaos -db /path/to/db
```

## 应用管理

### 应用商店

- 内置应用商店提供丰富的 Docker 应用
- 支持一键安装/卸载
- 已安装应用位于: `/var/lib/casaos/appstore/`

### 常用应用

- **文件管理**: 内置文件管理器
- **Terminal**: Web 终端
- **Docker 管理**: 查看和管理 Docker 容器

## 文件存储

- 默认存储路径: `/var/lib/casaos/www`
- 日志路径: `/var/log/casaos/`
- 应用数据: `/var/lib/casaos/db`

## Samba 网络共享

通过 Samba (SMB) 协议可以在其他设备上访问硬盘文件。

### 配置共享

编辑 Samba 配置文件：

```bash
sudo nano /etc/samba/smb.conf
```

在文件末尾添加共享配置：

```ini
[tool]
   path = /mnt/tool
   browsable = yes
   writable = yes
   guest ok = yes
   create mask = 0777
   directory mask = 0777
   force user = binzhan
   force group = binzhan
```

重启 SMB 服务：

```bash
sudo systemctl restart smbd
```

### 访问共享

- **Windows**: `\\<IP>\tool`
- **Mac**: `smb://<IP>/tool`
- **Linux**: `smb://<IP>/tool`

查看本机 IP：
```bash
hostname -I
```

### 查看 SMB 状态

```bash
systemctl status smbd
```

## 访问方式

1. **本地访问**: http://localhost:8088
2. **局域网访问**: http://<your-ip>:8088

## API 接口

CasaOS 提供 REST API，可通过以下地址访问:

- API 基础地址: http://localhost:8088/v1
- 系统信息: http://localhost:8088/v1/meta/info
- 应用列表: http://localhost:8088/v1/appstore/list

## 注意事项

1. 首次使用建议修改默认配置
2. 定期备份数据目录 `/var/lib/casaos/`
3. 确保防火墙允许 8088 端口访问
