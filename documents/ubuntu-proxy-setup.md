# Ubuntu 代理工具安装指南

## 方案 1：Clash Verge Rev（推荐）

Clash Verge Rev 是一个现代化的 Clash GUI 客户端，支持多种代理协议和订阅链接。

### 安装步骤

```bash
# 下载最新版
wget https://github.com/clash-verge-rev/clash-verge-rev/releases/download/v1.5.10/clash-verge_1.5.10_amd64.deb

# 安装 deb 包
sudo dpkg -i clash-verge_1.5.10_amd64.deb

# 修复依赖（如有需要）
sudo apt-get install -f
```

### 使用说明

1. 安装完成后，在应用菜单中搜索 "Clash Verge" 启动
2. 导入订阅链接：点击左侧 "订阅" → "导入"
3. 粘贴你的订阅链接，点击确定
4. 选择节点后点击 "系统代理" 开启代理

### 支持的代理协议

- Trojan
- Shadowsocks
- VMess
- VLESS
- Hysteria
- TUIC

---

## 其他备选方案

### 方案 2：Hiddify（界面现代）

```bash
wget https://github.com/hiddify/hiddify-next/releases/latest/download/Hiddify-Linux-x64.deb
sudo dpkg -i Hiddify-Linux-x64.deb
```

特点：
- 界面美观，支持中文
- 一键导入订阅
- 支持多种协议

### 方案 3：v2rayA（Web 界面）

```bash
curl -Ls https://mirrors.v2raya.com/go.sh | sudo bash
sudo systemctl start v2raya
```

然后浏览器访问：`http://localhost:2017`

### 方案 4：Snap 安装 Clash

```bash
sudo snap install clash
```

---

## 订阅配置格式

根据你的代理服务商提供的不同配置格式：

| 配置类型 | 支持工具 |
|---------|---------|
| Trojan-Qt5 / Shadowrocket / Spectre | Clash Verge, Hiddify |
| Clash / ClashX | Clash Verge |
| Surge Mac/iOS | Hiddify |
| QuantumultX | Hiddify |
| GUI-Config | Clash Verge |

---

## 常见问题

### 1. 无法连接 GitHub 下载

如果无法直接访问 GitHub，可以使用代理或镜像：

```bash
# 使用 ghproxy 镜像
wget https://ghproxy.com/https://github.com/clash-verge-rev/clash-verge-rev/releases/download/v1.5.10/clash-verge_1.5.10_amd64.deb
```

### 2. 依赖问题

```bash
sudo apt-get update
sudo apt-get install -f
sudo apt-get install libgtk-3-0 libwebkit2gtk-4.0-37
```

### 3. 启动失败

尝试从命令行启动查看错误：

```bash
clash-verge
```

---

---

## 功能说明

### TUN 模式

**TUN 模式**（Tunnel 模式）是一种全局代理模式，创建一个虚拟网卡来拦截**所有网络流量**（TCP + UDP）。

与普通系统代理的区别：

| 模式 | 代理范围 | 适用场景 |
|-----|---------|---------|
| **系统代理** | 仅 HTTP/HTTPS（浏览器等） | 日常上网、浏览网页 |
| **TUN 模式** | 所有流量（包括 UDP） | 游戏、视频通话、某些 APP |

**什么时候需要开 TUN？**
- 玩游戏需要代理
- 某些软件不走系统代理
- 需要代理 UDP 流量（视频通话、语音聊天）
- 想要真正的"全局代理"

**注意**：TUN 模式需要管理员/root 权限，可能会与 VPN 软件冲突。

---

### 系统代理守卫

**系统代理守卫**是一种保护功能，防止系统代理设置被其他程序修改或意外清除。

**主要作用**：
1. **锁定代理设置** - 阻止其他程序（浏览器扩展、其他代理工具）修改系统代理
2. **自动恢复** - 如果代理被意外清空，自动恢复正确配置
3. **防泄漏** - 确保所有流量都经过代理，不会绕过代理直连

**开启建议**：
- **开启**：需要确保代理不被篡改，或使用多个代理工具
- **关闭**：需要频繁手动切换代理，或与其他 VPN 软件共用

---

### 局域网连接

**局域网连接**允许局域网内其他设备通过你的电脑上网。

#### 开启局域网代理

**方法 1：GUI 界面（推荐）**
1. 打开 Clash Verge
2. 设置 → 基础设置 → **勾选"允许局域网连接"**
3. 重启 Clash

**方法 2：修改配置文件**
```bash
# 找到配置文件
~/.local/share/io.github.clash-verge-rev.clash-verge-rev/clash-verge.yaml

# 修改配置
sed -i 's/allow-lan: false/allow-lan: true/' ~/.local/share/io.github.clash-verge-rev.clash-verge-rev/clash-verge.yaml

# 重启 Clash Verge
```

#### 查看本机局域网 IP

```bash
ip addr show | grep "inet " | grep -v "127.0.0.1" | head -1
```

#### 其他设备连接配置

**连接信息**：

| 项目 | 值 |
|-----|---|
| 服务器地址 | `192.168.x.x`（你的电脑 IP） |
| HTTP 代理端口 | `7899` |
| SOCKS5 代理端口 | `7898` |
| 混合端口（推荐） | `7897` |

**手机 WiFi 代理设置**：
```
设置 → WLAN → 长按已连接 WiFi → 修改网络
代理类型：手动
代理主机名：192.168.x.x（你的电脑 IP）
代理端口：7897
```

**Windows/Mac 系统代理**：
```
系统设置 → 网络 → 代理
HTTP 代理：192.168.x.x:7899
HTTPS 代理：192.168.x.x:7899
SOCKS5 代理：192.168.x.x:7898
```

#### 防火墙设置

如果连接失败，确保防火墙允许端口：

```bash
sudo ufw allow 7897/tcp
sudo ufw allow 7898/tcp
sudo ufw allow 7899/tcp
```

**注意事项**：
- 两台设备必须在**同一局域网**（同一路由器/WiFi）
- 你的电脑必须保持开机和代理软件运行
- 代理端口可以在 Clash Verge → 设置 → 端口中查看

---

*文档更新于：2026-03-15*
