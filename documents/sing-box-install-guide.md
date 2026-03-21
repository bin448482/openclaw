# Sing-box 安装指南

## 简介

Sing-box 是一个通用代理平台，支持多种协议，包括 Shadowsocks、VMess、Trojan、VLESS、Hysteria 等。

## 系统要求

- Linux 系统（Ubuntu、Debian、CentOS 等）
- x86_64 架构
- root 或 sudo 权限

## 安装方法

### 方法一：使用官方脚本（推荐）

```bash
curl -fsSL https://sing-box.app/deb-install.sh | sudo bash
```

### 方法二：手动下载安装

#### 1. 下载最新版本

```bash
# 下载 sing-box deb 包
wget https://github.com/SagerNet/sing-box/releases/download/v1.10.5/sing-box_1.10.5_linux_amd64.deb
```

#### 2. 安装软件包

```bash
# 安装 deb 包
sudo dpkg -i sing-box_1.10.5_linux_amd64.deb

# 如果提示依赖问题，运行修复命令
sudo apt-get install -f
```

### 方法三：从源码编译

```bash
# 安装编译依赖
sudo apt-get update
sudo apt-get install -y git golang-go

# 克隆源码
git clone https://github.com/SagerNet/sing-box.git
cd sing-box

# 编译
make build

# 安装到系统
sudo cp sing-box /usr/local/bin/
```

### 方法四：使用 Docker

```bash
# 拉取镜像
docker pull ghcr.io/sagernet/sing-box

# 运行容器
docker run -d \
  --name sing-box \
  --restart always \
  -v /etc/sing-box:/etc/sing-box \
  -p 443:443 \
  ghcr.io/sagernet/sing-box
```

## 验证安装

```bash
# 检查版本
sing-box version

# 查看帮助
sing-box --help
```

## 配置

### 配置文件位置

- 系统配置：`/etc/sing-box/config.json`
- 用户配置：`~/.config/sing-box/config.json`

### 基本配置示例

```json
{
  "log": {
    "level": "info",
    "output": "/var/log/sing-box.log"
  },
  "inbounds": [
    {
      "type": "socks",
      "tag": "socks-in",
      "listen": "127.0.0.1",
      "port": 10808
    }
  ],
  "outbounds": [
    {
      "type": "vmess",
      "tag": "proxy",
      "server": "your-server.com",
      "server_port": 443,
      "uuid": "your-uuid",
      "security": "auto",
      "alter_id": 0
    }
  ]
}
```

## 服务管理

### 使用 systemd 管理

```bash
# 启动服务
sudo systemctl start sing-box

# 停止服务
sudo systemctl stop sing-box

# 重启服务
sudo systemctl restart sing-box

# 查看状态
sudo systemctl status sing-box

# 设置开机自启
sudo systemctl enable sing-box

# 查看日志
sudo journalctl -u sing-box -f
```

## 卸载

```bash
# 停止服务
sudo systemctl stop sing-box
sudo systemctl disable sing-box

# 卸载软件包
sudo apt-get remove sing-box
sudo apt-get autoremove

# 删除配置文件（可选）
sudo rm -rf /etc/sing-box
```

## 常见问题

### Q: 安装时提示依赖错误

```bash
sudo apt-get update
sudo apt-get install -f
sudo dpkg -i sing-box_*.deb
```

### Q: 服务启动失败

检查配置文件语法：
```bash
sing-box check -c /etc/sing-box/config.json
```

### Q: 端口被占用

```bash
sudo netstat -tlnp | grep 443
sudo kill -9 <PID>
```

## 参考链接

- 官方文档：https://sing-box.sagernet.org/
- GitHub：https://github.com/SagerNet/sing-box
- 发布页：https://github.com/SagerNet/sing-box/releases

---

*文档生成时间：2026年3月9日*
