# Windows + Caddy + acme.sh + ESA 部署方案

> 场景：Windows 服务器 + Caddy + acme.sh（Git Bash + DNS-01 验证）+ 阿里云 ESA，解决 80/443 端口封禁问题

---

## 目录

- [[#一、方案概述]]
- [[#二、环境准备]]
- [[#三、acme.sh 安装与证书申请]]
  - [[#3.1 安装 Git Bash]]
  - [[#3.2 安装 acme.sh]]
  - [[#3.3 配置阿里云 DNS 凭证]]
  - [[#3.4 申请泛域名证书]]
  - [[#3.5 安装证书到 Caddy 目录]]
- [[#四、Caddy 配置]]
  - [[#4.1 安装 Caddy]]
  - [[#4.2 Caddyfile 配置]]
  - [[#4.3 注册为 Windows 服务]]
- [[#五、阿里云 ESA 配置]]
  - [[#5.1 开通与添加站点]]
  - [[#5.2 DNS 解析配置]]
  - [[#5.3 回源规则配置]]
  - [[#5.4 SSL 证书配置]]
- [[#六、Windows 防火墙配置]]
- [[#七、自动续期配置]]
- [[#八、验证测试]]
- [[#九、常见问题排错]]
- [[#十、部署 Checklist]]

---

## 一、方案概述

### 适用场景
- Windows Server / Windows 10/11
- 80/443 端口被运营商封禁
- 使用阿里云 DNS 管理域名
- 需要泛域名证书（`*.example.com`）
- 配合阿里云 ESA 实现标准端口访问
- 偏好 Caddy 简洁配置

### 方案说明
Caddy 自带自动 HTTPS，但默认使用 HTTP-01 验证，需要 80 端口。
80 端口被封后，使用 acme.sh + DNS-01 验证申请证书，然后配置 Caddy 加载外部证书。

| 对比项 | Caddy 自动证书 | Caddy + acme.sh |
|--------|---------------|-----------------|
| 80 端口 | 需要 | 不需要 |
| 证书申请 | 自动 | acme.sh 管理 |
| 配置复杂度 | 极低 | 中等 |
| 泛域名支持 | 需 DNS 插件 | ✅ 支持 |
| 推荐度 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

### 整体架构

```
用户浏览器
  https://blog.example.com  (标准443端口)
       ↓
阿里云 ESA 边缘节点
  (加速 + HTTPS + WAF)
       ↓  ESA 回源 HTTPS 8443
  origin.example.com:8443
       ↓
Windows Caddy
  (监听 8443 端口，加载外部证书)
       ↓
Quartz 静态站点
  D:/quartz_build/
```

### 端口规划

| 端口 | 用途 | 说明 |
|------|------|------|
| 8443 | ESA 回源专用 | HTTPS，专供 ESA 回源 |
| 5443 | 用户直连备用 | HTTPS，用户直接访问备用 |

---

## 二、环境准备

### 系统要求
- Windows Server 2016+ / Windows 10+
- 公网 IP 地址
- 域名托管在阿里云 DNS
- 阿里云 AccessKey（DNS 管理权限）

### 目录规划

```
D:/
├── caddy/                  # Caddy 程序目录
│   ├── caddy.exe
│   ├── Caddyfile
│   └── ssl/
│       └── example.com/
├── quartz_build/           # Quartz 静态站点
├── Git/                    # Git Bash 安装目录
└── nssm/                   # NSSM 服务管理工具
```

### 准备阿里云 AccessKey

1. 登录阿里云控制台 → 访问控制 RAM
2. 创建用户，勾选「OpenAPI 访问」
3. 授权策略：`AliyunDNSFullAccess`
4. 保存 AccessKey ID 和 AccessKey Secret

---

## 三、acme.sh 安装与证书申请

### 3.1 安装 Git Bash

**下载地址：**
[https://git-scm.com/download/win](https://git-scm.com/download/win)

**安装步骤：**
1. 下载安装包，运行安装
2. 一路默认即可
3. 安装完成后，右键菜单会出现「Git Bash Here」

### 3.2 安装 acme.sh

**打开 Git Bash，执行：**
```bash
curl https://get.acme.sh | sh -s email=your-email@example.com
```

**使环境变量生效：**
```bash
source ~/.bashrc
```

**验证安装：**
```bash
acme.sh --version
```

### 3.3 配置阿里云 DNS 凭证

```bash
export Ali_Key="你的AccessKeyID"
export Ali_Secret="你的AccessKeySecret"
```

> 💡 凭证会自动保存到 `~/.acme.sh/account.conf`

### 3.4 申请泛域名证书

```bash
acme.sh --issue \
  --dns dns_ali \
  -d example.com \
  -d *.example.com
```

### 3.5 安装证书到 Caddy 目录

**创建证书目录：**
```cmd
mkdir D:\caddy\ssl\example.com
```

**在 Git Bash 中安装证书：**
```bash
acme.sh --install-cert -d example.com \
  --key-file       /d/caddy/ssl/example.com/privkey.pem \
  --fullchain-file /d/caddy/ssl/example.com/fullchain.pem \
  --reloadcmd      "D:/caddy/caddy.exe reload --config D:/caddy/Caddyfile"
```

**注意：**
- Git Bash 中 Windows 路径写法：`D:/` → `/d/`
- `--reloadcmd` 中的路径用 Windows 格式（正斜杠）

> ✅ 续期后自动复制新证书 + 自动重载 Caddy

---

## 四、Caddy 配置

### 4.1 安装 Caddy

**下载地址：**
[https://caddyserver.com/download](https://caddyserver.com/download)

**选择版本：**
- Platform: Windows
- Arch: amd64
- 点击「Download」

**安装步骤：**
1. 下载 zip 包
2. 解压到 `D:/caddy/`
3. 重命名 `caddy_windows_amd64.exe` 为 `caddy.exe`

**验证安装：**
打开 CMD，执行：
```cmd
cd /d D:/caddy
caddy version
```

### 4.2 Caddyfile 配置

**创建 Caddyfile：**
文件路径：`D:/caddy/Caddyfile`

```caddyfile
# ============================================
# Quartz 静态站点 - Windows Caddy + ESA 配置
# 端口：8443（ESA回源） + 5443（直连备用）
# 证书：acme.sh 外部证书
# ============================================

# ---------- 端口 8443：专供 ESA 回源 ----------
https://example.com:8443, https://www.example.com:8443, https://origin.example.com:8443 {
	# 加载外部证书（关闭 Caddy 自动 HTTPS）
	tls D:/caddy/ssl/example.com/fullchain.pem D:/caddy/ssl/example.com/privkey.pem

	# 站点根目录
	root * D:/quartz_build

	# 启用静态文件服务
	file_server

	# ---------- Quartz SPA 路由核心 ----------
	# 解决：页面刷新 404 问题
	try_files {path} {path}/ {path}.html /index.html

	# ---------- 静态资源缓存 ----------
	@static {
		path *.js *.css *.png *.jpg *.jpeg *.svg *.gif *.ico *.webp *.woff *.woff2 *.ttf *.eot
	}
	header @static Cache-Control "public, immutable"

	# ---------- 安全响应头 ----------
	header {
		X-Content-Type-Options nosniff
		X-Frame-Options SAMEORIGIN
		Referrer-Policy strict-origin-when-cross-origin
	}

	# ---------- 日志 ----------
	log {
		output file D:/caddy/logs/quartz-esa-access.log
		format json
	}
}

# ---------- 端口 5443：用户直连备用 ----------
https://example.com:5443, https://www.example.com:5443 {
	tls D:/caddy/ssl/example.com/fullchain.pem D:/caddy/ssl/example.com/privkey.pem

	root * D:/quartz_build
	file_server

	try_files {path} {path}/ {path}.html /index.html

	@static {
		path *.js *.css *.png *.jpg *.jpeg *.svg *.gif *.ico *.webp *.woff *.woff2 *.ttf *.eot
	}
	header @static Cache-Control "public, immutable"

	log {
		output file D:/caddy/logs/quartz-direct-access.log
		format json
	}
}
```

> ⚠️ **Windows 路径注意：**
> - Caddy 配置中路径用正斜杠 `D:/caddy/ssl/...`
> - 不要用单反斜杠

**创建日志目录：**
```cmd
mkdir D:\caddy\logs
```

**验证配置并启动：**
```cmd
cd /d D:/caddy

# 验证配置
caddy validate --config Caddyfile

# 前台运行（测试用）
caddy run --config Caddyfile

# 后台运行（生产用）
caddy start --config Caddyfile

# 重载配置
caddy reload --config Caddyfile

# 停止
caddy stop
```

### 4.3 注册为 Windows 服务

默认 Caddy 不是 Windows 服务，重启后需要手动启动。推荐用 NSSM 注册成服务。

**下载 NSSM：**
[https://nssm.cc/download](https://nssm.cc/download)

**解压到：** `D:/nssm/`

**注册服务（以管理员身份运行 CMD）：**
```cmd
cd /d D:/nssm/win64
nssm install Caddy
```

弹出图形界面：
- **Path**：`D:\caddy\caddy.exe`
- **Startup directory**：`D:\caddy`
- **Arguments**：`run --config D:\caddy\Caddyfile`
- 点击「Install service」

**服务管理命令：**
```cmd
# 启动
net start Caddy

# 停止
net stop Caddy

# 重启
net stop Caddy && net start Caddy

# 删除服务
nssm remove Caddy
```

---

## 五、阿里云 ESA 配置

### 5.1 开通与添加站点

1. 登录阿里云控制台 → 搜索「边缘安全加速 ESA」
2. 点击「立即开通」，选择合适套餐
3. 站点管理 → 添加站点
4. 站点域名：`example.com`
5. **区域选择：全球（不包含中国内地）** ⭐ 关键！无需备案
6. 接入方式：**CNAME**（推荐）
7. 完成添加

### 5.2 DNS 解析配置

在阿里云 DNS 添加记录：

| 记录类型 | 主机记录 | 记录值 | 说明 |
|----------|----------|--------|------|
| CNAME | `@` | ESA 提供的 CNAME 地址 | 主域名走 ESA |
| CNAME | `www` | ESA 提供的 CNAME 地址 | www 走 ESA |
| A | `origin` | 你的服务器公网 IP | 源站域名，直连 |

### 5.3 回源规则配置（核心！）

左侧菜单 → 规则 → 回源规则 → 新增规则

**配置参数：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| 规则名称 | `quartz-https-8443` | 自定义 |
| 传入请求类型 | 所有传入请求 | 全站加速 |
| 回源 HOST | `example.com` | 和 Caddy 域名一致 |
| 回源协议 | **HTTPS** | 源站用 HTTPS |
| 回源端口 | **8443** | Caddy 监听的端口 |
| 回源 SNI | `example.com` | HTTPS 回源必填 |

> 🎯 **核心逻辑：**
> - 用户访问 `https://example.com`（标准 443 端口）
> - ESA 回源到 `https://origin.example.com:8443`（自定义端口）
> - 完美绕过运营商 80/443 端口封禁

### 5.4 SSL 证书配置

**申请边缘证书：**
1. SSL/TLS → 边缘证书 → 申请免费证书
2. 证书颁发机构：Let's Encrypt
3. 证书域名：`example.com,*.example.com`
4. 等待签发

**SSL/TLS 加密模式：**
- 选择「**完整**」模式
- 源站有有效证书（acme.sh 签发）

---

## 六、Windows 防火墙配置

### 6.1 放行 Caddy 端口

**方式一：PowerShell 命令（推荐）**

以管理员身份运行 PowerShell：

```powershell
# 放行 8443 端口（ESA 回源）
New-NetFirewallRule `
  -DisplayName "Caddy ESA 8443" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 8443 `
  -Action Allow

# 放行 5443 端口（直连备用）
New-NetFirewallRule `
  -DisplayName "Caddy Direct 5443" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 5443 `
  -Action Allow
```

**方式二：图形界面**
1. 打开「高级安全 Windows Defender 防火墙」
2. 入站规则 → 新建规则
3. 规则类型：端口 → TCP
4. 特定本地端口：`8443,5443`
5. 操作：允许连接
6. 配置文件：全选
7. 名称：`Caddy HTTPS (8443,5443)`

---

## 七、自动续期配置

### 7.1 acme.sh 自动续期原理

acme.sh 安装后自动创建 Windows 计划任务，每天检查续期。

**续期逻辑：**
- 每天自动检查
- 证书剩余 < 30 天时自动续期
- 续期后自动执行 `--install-cert` 配置的 `--reloadcmd`

### 7.2 手动测试续期

在 Git Bash 中执行：
```bash
# 强制续期测试
acme.sh --renew -d example.com --force
```

**查看证书信息：**
```bash
acme.sh --info -d example.com
```

### 7.3 升级 acme.sh

```bash
# 升级到最新版
acme.sh --upgrade

# 开启自动升级
acme.sh --upgrade --auto-upgrade
```

---

## 八、验证测试

### 8.1 本地测试

```cmd
cd /d D:/caddy

# 验证配置
caddy validate --config Caddyfile

# 测试 HTTPS 访问
curl -k https://localhost:8443
curl -k https://localhost:5443
```

### 8.2 外网直连测试

用手机流量访问：
```
https://origin.example.com:5443
```

### 8.3 ESA 加速测试

访问主域名：
```
https://example.com
```

**检查项：**
- [ ] 正常打开首页
- [ ] 地址栏显示 🔒 锁图标
- [ ] URL 无端口号
- [ ] 内页跳转正常
- [ ] 刷新页面不 404
- [ ] 静态资源加载正常

---

## 九、常见问题排错

### Q1: Caddy 启动失败，证书文件找不到
**原因：** 路径写错或权限问题
**解决：**
- 检查证书路径是否用了正斜杠 `D:/caddy/ssl/...`
- 确认证书文件确实存在
- 查看日志：`D:/caddy/logs/`

### Q2: Caddy 还在尝试自动申请证书
**原因：** 没有正确配置外部证书
**解决：**
- 确保 `tls` 指令指定了证书文件路径
- 域名必须带端口号（如 `https://example.com:8443`）
- 这样 Caddy 不会自动申请证书

### Q3: 页面刷新 404
**原因：** try_files 缺少 `{path}.html`
**解决：**
```caddyfile
try_files {path} {path}/ {path}.html /index.html
```

### Q4: DNS 验证失败
**排查：**
- AccessKey 是否正确
- RAM 用户是否有 `AliyunDNSFullAccess` 权限
- 加 `--debug` 参数查看详细日志

### Q5: ESA 回源 502 错误
**排查步骤：**
1. 源站 Caddy 是否正常运行？
2. Windows 防火墙是否放行 8443 端口？
3. ESA 回源协议和端口是否匹配？
4. 回源 SNI 是否填写？
5. 查看 Caddy 日志

### Q6: 证书续期后 Caddy 没加载新证书
**原因：** 没有配置 `--reloadcmd`
**解决：**
```bash
# 重新安装证书，配置 reloadcmd
acme.sh --install-cert -d example.com \
  --key-file       /d/caddy/ssl/example.com/privkey.pem \
  --fullchain-file /d/caddy/ssl/example.com/fullchain.pem \
  --reloadcmd      "D:/caddy/caddy.exe reload --config D:/caddy/Caddyfile"
```

### Q7: Windows 服务启动失败
**排查：**
- 检查 NSSM 配置的路径和参数是否正确
- 查看 Windows 事件查看器 → 应用程序日志
- 手动运行 `caddy run` 看是否有报错

### Q8: Git Bash 路径写法混乱
**Git Bash 中的路径：**
- `D:/caddy/` → `/d/caddy/`

**Caddy 配置中的路径：**
- 用正斜杠 `D:/caddy/ssl/...`

**acme.sh --reloadcmd 中的路径：**
- 用 Windows 格式（正斜杠）`D:/caddy/caddy.exe reload --config D:/caddy/Caddyfile`

---

## 十、部署 Checklist

### 第一阶段：环境准备
- [ ] Windows 服务器可访问公网
- [ ] 域名托管在阿里云 DNS
- [ ] 创建 RAM 子用户，获取 AccessKey
- [ ] 安装 Git Bash
- [ ] 上传 Quartz 静态文件到 `D:/quartz_build/`

### 第二阶段：证书申请
- [ ] 安装 acme.sh（Git Bash 中）
- [ ] 配置阿里云 DNS 凭证
- [ ] 申请泛域名证书
- [ ] 安装证书到 `D:/caddy/ssl/example.com/`
- [ ] 配置 reloadcmd 自动重载 Caddy

### 第三阶段：Caddy 配置
- [ ] 下载 Caddy for Windows，解压到 `D:/caddy/`
- [ ] 编写 Caddyfile（8443 + 5443 双端口）
- [ ] 配置外部证书路径
- [ ] `caddy validate` 验证配置
- [ ] 启动 Caddy
- [ ] 用 NSSM 注册为 Windows 服务
- [ ] Windows 防火墙放行 8443、5443 端口

### 第四阶段：ESA 配置
- [ ] 开通 ESA 服务
- [ ] 添加站点（全球不含中国内地）
- [ ] 配置 DNS CNAME 记录
- [ ] 配置回源规则（HTTPS + 8443 + SNI）
- [ ] 申请边缘证书
- [ ] 设置 SSL 模式为「完整」

### 第五阶段：测试验证
- [ ] 本地测试 Caddy
- [ ] 外网直连测试
- [ ] ESA 加速测试
- [ ] 验证路由刷新不 404
- [ ] 验证证书有效
- [ ] 测试自动续期

---

## 相关笔记

- [Linux-Nginx-Certbot-ESA部署方案.md](Linux-Nginx-Certbot-ESA部署方案.md)
- [Linux-Nginx-acme.sh-ESA部署方案.md](Linux-Nginx-acme.sh-ESA部署方案.md)
- [Linux-Caddy-Certbot-ESA部署方案.md](Linux-Caddy-Certbot-ESA部署方案.md)
- [Linux-Caddy-acme.sh-ESA部署方案.md](Linux-Caddy-acme.sh-ESA部署方案.md)
- [Windows-Nginx-acme.sh-ESA部署方案.md](Windows-Nginx-acme.sh-ESA部署方案.md)
- [Quartz静态站点搭建与发布完整教程](Quartz静态站点搭建与发布完整教程.md)
