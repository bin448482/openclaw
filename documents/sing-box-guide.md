# Sing-Box 代理使用指南

## 服务状态

### 启动代理
```bash
sudo sing-box run -D ~/.config/sing-box
```

### 关闭代理
```bash
pkill sing-box
```

### 检查运行状态
```bash
ps aux | grep sing-box
```

---

## 代理端口

- **HTTP/SOCKS5 代理**: `127.0.0.1:7890`
- **Web 控制面板**: `http://127.0.0.1:9090/ui`

---

## 切换节点

### 方式一：Web 面板
访问 http://127.0.0.1:9090/ui
- 左侧选择「代理」
- 点击要使用的节点即可切换
- 选择「DIRECT」可关闭代理（直连）

### 方式二：命令行
```bash
# 关闭代理（直连）
curl -X PUT http://127.0.0.1:9090 -d '{"name": "DIRECT"}'

# 开启代理（自动选择）
curl -X PUT http://127.0.0.1:9090 -d '{"name": "♻️ 自动选择"}'

# 选择特定节点（如香港节点）
curl -X PUT http://127.0.0.1:9090 -d '{"name": "🇭🇰 香港 01α"}'
```

---

## 客户端配置

### 浏览器
安装 SwitchyOmega 插件，代理地址填：
- HTTP: `127.0.0.1:7890`
- SOCKS5: `127.0.0.1:7890`

### 命令行代理
```bash
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890

# 测试
curl -x http://127.0.0.1:7890 https://google.com
```

---

## 常用节点标签

| 标签 | 说明 |
|------|------|
| ♻️ 自动选择 | 自动选择最优节点 |
| 🚀 节点选择 | 手动选择节点 |
| DIRECT | 直连（关闭代理） |
| 🇭🇰 香港 XX | 香港节点 |
| 🇹🇼 台湾 XX | 台湾节点 |
| 🇯🇵 日本 XX | 日本节点 |
| 🇺🇸 美国 XX | 美国节点 |
| 🇸🇬 狮城 XX | 新加坡节点 |

---

## 故障排查

### 代理无法连接
1. 检查 sing-box 是否运行：`ps aux | grep sing-box`
2. 检查端口是否监听：`ss -tlnp | grep 7890`
3. 查看日志：`journalctl -u sing-box`

### 节点无法使用
切换到其他节点试试，或重启服务：
```bash
pkill sing-box
sudo sing-box run -D ~/.config/sing-box
```
