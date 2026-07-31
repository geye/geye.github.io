# Linux + Caddy + acme.sh + ESA 部署方案

> 场景：Linux 服务器 + Caddy + acme.sh（DNS-01 验证）+ 阿里云 ESA，解决 80/443 端口封禁问题

---

## 目录

- [[#一、方案概述]]
- [[#二、环境准备]]
- [[#三、acme.sh 安装与证书申请]]
  - [[#3.1 安装 acme.sh]]
  - [[#3.2 配置阿里云 DNS 凭证]]
  - [[#3.3 申请泛域名证书]]
  - [[#3.4 安装证书到 Caddy 目录]]
- [[#四、Caddy 配置]]
  - [[#4.1 安装 Caddy]]
  - [[#4.2 Caddyfile 配置]]
  - [[#4.3 关键配置说明]]
- [[#五、阿里云 ESA 配置]]
  - [[#5.1 开通与添加站点]]
  - [[#5.2 DNS 解析配置]]
  - [[#5.3 回源规则配置]]
  - [[#5.4 SSL 证书配置]]
- [[#六、自动续期配置]]
- [[#七、验证测试]]
- [[#八、常见问题排错]]
- [[#九、部署 Checklist]]

---

## 一、方案概述

### 适用场景
- Linux 服务器（Ubuntu/Debian/CentOS）
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
| DNS 插件 | 需单独编译 | 内置 100+ |
| 配置复杂度 | 极低 | 中等 |
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
Linux Caddy
  (监听 8443 端口，加载外部证书)
       ↓
Quartz 静态站点
  /var/www/quartz/
```

### 端口规划

| 端口 | 用途 | 说明 |
|------|------|------|
| 8443 | ESA 回源专用 | HTTPS，专供 ESA 回源 |
| 5443 | 用户直连备用 | HTTPS，用户直接访问备用 |

---

## 二、环境准备

### 系统要求
- Linux 服务器（Ubuntu 20.04+ / Debian 11+ / CentOS 7+）
- 公网 IP 地址
- 域名托管在阿里云 DNS
- 阿里云 AccessKey（DNS 管理权限）

### 目录规划

```
/var/www/quartz/          # Quartz 静态站点
/etc/caddy/               # Caddy 配置目录
├── Caddyfile             # 主配置文件
└── ssl/                  # 证书目录
    └── example.com/
~/.acme.sh/               # acme.sh 安装目录
```

### 准备阿里云 AccessKey

1. 登录阿里云控制台 → 访问控制 RAM
2. 创建用户，勾选「OpenAPI 访问」
3. 授权策略：`AliyunDNSFullAccess`
4. 保存 AccessKey ID 和 AccessKey Secret

---

## 三、acme.sh 安装与证书申请

### 3.1 安装 acme.sh

**一键安装：**
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

### 3.2 配置阿里云 DNS 凭证

```bash
export Ali_Key="你的AccessKeyID"
export Ali_Secret="你的AccessKeySecret"
```

> 💡 凭证会自动保存到 `~/.acme.sh/account.conf`

### 3.3 申请泛域名证书

```bash
acme.sh --issue \
  --dns dns_ali \
  -d example.com \
  -d *.example.com
```

**参数说明：**
| 参数 | 说明 |
|------|------|
| `--issue` | 申请证书 |
| `--dns dns_ali` | 使用阿里云 DNS 验证 |
| `-d` | 申请的域名，可多个 |

### 3.4 安装证书到 Caddy 目录

**创建证书目录：**
```bash
sudo mkdir -p /etc/caddy/ssl/example.com
sudo chown caddy:caddy /etc/caddy/ssl/example.com
```

**安装证书：**
```bash
acme.sh --install-cert -d example.com \
  --key-file       /etc/caddy/ssl/example.com/privkey.pem \
  --fullchain-file /etc/caddy/ssl/example.com/fullchain.pem \
  --reloadcmd      "systemctl reload caddy"
```

**参数说明：**
| 参数 | 说明 |
|------|------|
| `--install-cert` | 安装证书 |
| `--key-file` | 私钥输出路径 |
| `--fullchain-file` | 证书链输出路径 |
| `--reloadcmd` | 证书更新后执行的命令 |

> ✅ 续期后自动复制新证书 + 自动重载 Caddy

---

## 四、Caddy 配置

### 4.1 安装 Caddy

**Ubuntu/Debian：**
```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

**CentOS/RHEL：**
```bash
dnf install 'dnf-command(copr)'
dnf copr enable @caddy/caddy
dnf install caddy
```

**验证安装：**
```bash
caddy version
```

### 4.2 Caddyfile 配置

**编辑 Caddyfile：**
```bash
sudo nano /etc/caddy/Caddyfile
```

**完整配置：**
```caddyfile
# ============================================
# Quartz 静态站点 - Caddy + ESA 配置
# 端口：8443（ESA回源） + 5443（直连备用）
# 证书：acme.sh 外部证书
# ============================================

# ---------- 端口 8443：专供 ESA 回源 ----------
https://example.com:8443, https://www.example.com:8443, https://origin.example.com:8443 {
	# 加载外部证书（关闭 Caddy 自动 HTTPS）
	tls /etc/caddy/ssl/example.com/fullchain.pem /etc/caddy/ssl/example.com/privkey.pem

	# 站点根目录
	root * /var/www/quartz

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
		output file /var/log/caddy/quartz-esa-access.log
		format json
	}
}

# ---------- 端口 5443：用户直连备用 ----------
https://example.com:5443, https://www.example.com:5443 {
	tls /etc/caddy/ssl/example.com/fullchain.pem /etc/caddy/ssl/example.com/privkey.pem

	root * /var/www/quartz
	file_server

	try_files {path} {path}/ {path}.html /index.html

	@static {
		path *.js *.css *.png *.jpg *.jpeg *.svg *.gif *.ico *.webp *.woff *.woff2 *.ttf *.eot
	}
	header @static Cache-Control "public, immutable"

	log {
		output file /var/log/caddy/quartz-direct-access.log
		format json
	}
}
```

### 4.3 关键配置说明

**1. 加载外部证书**
```caddyfile
tls /path/to/fullchain.pem /path/to/privkey.pem
```
- 指定证书文件后，Caddy 不会自动申请证书
- 直接加载 acme.sh 安装的证书
- 适合 80 端口被封的场景

**2. try_files 详解（Quartz 必看）**
```caddyfile
try_files {path} {path}/ {path}.html /index.html
```
按顺序查找：
1. `{path}` → 精确匹配文件
2. `{path}/` → 匹配目录下的 index.html
3. `{path}.html` → 自动补 .html 后缀（Quartz 关键！）
4. `/index.html` → 都找不到，交给 SPA 路由

> 🎯 `{path}.html` 是解决 Quartz 路由问题的核心

**3. 多域名 + 端口写法**
```caddyfile
https://example.com:8443, https://www.example.com:8443 {
    # 配置
}
```
- 多个域名用逗号分隔
- 必须加 `https://` 前缀和端口号
- 共用同一个配置块

**验证配置并启动：**
```bash
# 验证配置
sudo caddy validate --config /etc/caddy/Caddyfile

# 重载配置
sudo systemctl reload caddy

# 查看状态
sudo systemctl status caddy
```

**创建日志目录：**
```bash
sudo mkdir -p /var/log/caddy
sudo chown caddy:caddy /var/log/caddy
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

## 六、自动续期配置

### 6.1 acme.sh 自动续期原理

acme.sh 安装后自动创建 cron 任务：
```bash
crontab -l
```

**续期逻辑：**
- 每天凌晨自动检查
- 证书剩余 < 30 天时自动续期
- 续期后自动执行 `--install-cert` 配置的 `--reloadcmd`

### 6.2 手动测试续期

```bash
# 强制续期测试
acme.sh --renew -d example.com --force
```

**查看证书信息：**
```bash
acme.sh --info -d example.com
```

### 6.3 升级 acme.sh

```bash
# 升级到最新版
acme.sh --upgrade

# 开启自动升级
acme.sh --upgrade --auto-upgrade
```

---

## 七、验证测试

### 7.1 本地测试

```bash
# 验证 Caddy 配置
sudo caddy validate --config /etc/caddy/Caddyfile

# 测试 HTTPS 访问
curl -k https://localhost:8443
curl -k https://localhost:5443
```

### 7.2 外网直连测试

用手机流量访问：
```
https://origin.example.com:5443
```

### 7.3 ESA 加速测试

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

### 7.4 证书验证

```bash
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null | openssl x509 -noout -dates
```

---

## 八、常见问题排错

### Q1: acme.sh 安装失败
**原因：** 网络问题或缺少依赖
**解决：**
```bash
# 安装依赖
sudo apt install curl socat -y

# 使用国内镜像
curl https://gitcode.net/acme-sh/acme.sh/-/raw/master/acme.sh | sh
```

### Q2: DNS 验证失败
**排查：**
- AccessKey 是否正确
- RAM 用户是否有 `AliyunDNSFullAccess` 权限
- 加 `--debug` 参数查看详细日志：
  ```bash
  acme.sh --issue --dns dns_ali -d example.com --debug
  ```

### Q3: Caddy 启动失败，证书权限问题
**原因：** caddy 用户没有读取证书的权限
**解决：**
```bash
# 修改证书目录权限
sudo chown -R caddy:caddy /etc/caddy/ssl

# 或设置全局可读
sudo chmod -R 644 /etc/caddy/ssl/example.com
```

### Q4: 页面刷新 404
**原因：** try_files 缺少 `{path}.html`
**解决：**
```caddyfile
try_files {path} {path}/ {path}.html /index.html
```

### Q5: ESA 回源 502 错误
**排查步骤：**
1. 源站 Caddy 是否正常运行？
2. 防火墙是否放行 8443 端口？
3. ESA 回源协议和端口是否匹配？
4. 回源 SNI 是否填写？
5. 查看 Caddy 日志：`journalctl -u caddy`

### Q6: 证书续期后 Caddy 没加载新证书
**原因：** 没有配置 `--reloadcmd`
**解决：**
```bash
# 重新安装证书，配置 reloadcmd
acme.sh --install-cert -d example.com \
  --key-file       /etc/caddy/ssl/example.com/privkey.pem \
  --fullchain-file /etc/caddy/ssl/example.com/fullchain.pem \
  --reloadcmd      "systemctl reload caddy"
```

---

## 九、部署 Checklist

### 第一阶段：环境准备
- [ ] Linux 服务器可访问公网
- [ ] 域名托管在阿里云 DNS
- [ ] 创建 RAM 子用户，获取 AccessKey
- [ ] 上传 Quartz 静态文件到 `/var/www/quartz/`

### 第二阶段：证书申请
- [ ] 安装 acme.sh
- [ ] 配置阿里云 DNS 凭证
- [ ] 申请泛域名证书
- [ ] 安装证书到 Caddy 目录
- [ ] 配置 reloadcmd 自动重载

### 第三阶段：Caddy 配置
- [ ] 安装 Caddy
- [ ] 编写 Caddyfile（8443 + 5443 双端口）
- [ ] 配置外部证书路径
- [ ] `caddy validate` 验证配置
- [ ] 启动/重载 Caddy
- [ ] 防火墙放行 8443、5443 端口

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
- [Linux-Caddy-acme.sh-ESA部署方案.md](Linux-Caddy-acme.sh-ESA部署方案.md)
- [Linux-Caddy-Certbot-ESA部署方案.md](Linux-Caddy-Certbot-ESA部署方案.md)
- [Windows-Nginx-acme.sh-ESA部署方案.md](Windows-Nginx-acme.sh-ESA部署方案.md)
- [Windows-Caddy-acme.sh-ESA部署方案.md](Windows-Caddy-acme.sh-ESA部署方案)
- [Quartz静态站点搭建与发布完整教程](Quartz静态站点搭建与发布完整教程.md)
