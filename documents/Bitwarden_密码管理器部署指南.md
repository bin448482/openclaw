# Bitwarden 自托管密码管理器部署指南

## 简介

Bitwarden 是一款开源的密码管理器，支持自托管，数据存储在自己的服务器上，安全可控。

## 方案对比

| 方案 | 类型 | 费用 | 特点 |
|-----|------|-----|------|
| Bitwarden | 开源/自托管 | 免费 | 可在 NAS 上运行，数据自主掌控 |
| KeePass | 开源/离线 | 免费 | 本地数据库，适合离线使用 |
| 1Password | 商业 | 付费 | 体验好，跨平台 |
| Chrome/Edge | 浏览器内置 | 免费 | 简单，但安全性一般 |

## 推荐方案：Bitwarden

### 优势
- 开源免费
- 浏览器/手机/电脑全平台支持
- 数据存在自己 NAS 上，安全可控
- 安装简单（Docker）

## 安装步骤

### 前提条件
- 已安装 Docker
- 硬盘挂载在 /mnt/tool

### 方法一：Docker 安装（推荐）

```bash
# 1. 创建存储目录
mkdir -p /mnt/tool/bitwarden

# 2. 运行 Bitwarden
docker run -d \
  --name bitwarden \
  -v /mnt/tool/bitwarden:/data \
  -p 8080:80 \
  bitwardenrs/server:latest
```

### 方法二：Docker Compose 安装

```yaml
# docker-compose.yml
version: '3'

services:
  bitwarden:
    image: bitwardenrs/server:latest
    container_name: bitwarden
    restart: always
    ports:
      - "8080:80"
    volumes:
      - /mnt/tool/bitwarden:/data
    environment:
      - SIGNUPS_ALLOWED=true
```

```bash
# 启动
docker-compose up -d
```

### 方法三：进阶：使用 Vaultwarden（更轻量）

```bash
# Vaultwarden 是 Bitwarden 的轻量替代品
docker run -d \
  --name vaultwarden \
  -v /mnt/tool/vaultwarden:/data \
  -p 8080:80 \
  vaultwarden/server:latest
```

## 访问方式

- 浏览器访问：`http://你的IP:8080`
- 首次注册账号后，建议关闭注册：`SIGNUPS_ALLOWED=false`

## 配置 HTTPS（可选）

建议使用 Nginx 反向代理 + Let's Encrypt 配置 HTTPS。

## 数据备份

Bitwarden 数据存储在 `/mnt/tool/bitwarden` 目录，定期备份该目录即可。

## 客户端配置

- Windows/Mac/Linux: 下载官方客户端
- iOS/Android: 应用商店搜索 "Bitwarden"
- 浏览器: 安装 Chrome/Firefox 扩展

服务器地址填写: `http://你的IP:8080`

## 卸载

```bash
# 停止容器
docker stop bitwarden

# 删除容器
docker rm bitwarden

# 数据保留（可选择性删除）
# rm -rf /mnt/tool/bitwarden
```
