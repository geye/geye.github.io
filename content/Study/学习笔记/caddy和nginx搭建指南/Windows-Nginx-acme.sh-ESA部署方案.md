# Windows + Nginx + acme.sh + ESA 部署方案

> 场景：Windows 服务器 + Nginx + acme.sh（Git Bash + DNS-01 验证）+ 阿里云 ESA，解决 80/443 端口封禁问题

---

## 目录

- [[#一、方案概述]]
- [[#二、环境准备]]
- [[#三、acme.sh 安装与证书申请]]
  - [[#3.1 安装 Git Bash]]
  - [[#3.2 安装 acme.sh]]
  - [[#3.3 配置阿里云 DNS 凭证]]
  - [[#3.4 申请泛域名证书]]
  - [[#3.5 安装证书到 Nginx 目录]]
- [[#四、Nginx 配置]]
  - [[#4.1 安装 Nginx]]
  - [[#4.2 站点配置文件]]
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
- 偏好 acme.sh 跨平台统一脚本

### Windows 下 acme.sh 运行方式
acme.sh 是 Shell 脚本，Windows 下需要通过 Git Bash 运行。

| 运行环境 | 说明 | 推荐度 |
|----------|------|--------|
| Git Bash | 轻量，安装简单 | ⭐⭐⭐⭐⭐ |
| WSL2 | 完整 Linux 环境 | ⭐⭐⭐⭐ |
| Cygwin | 较老，不推荐 | ⭐⭐ |

> 💡 推荐 Git Bash，轻量且够用

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
Windows Nginx
  (监听 8443 端口)
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
├── nginx/                  # Nginx 程序目录
│   ├── nginx.exe
│   ├── conf/
│   │   ├── nginx.conf
│   │   └── conf.d/
│   │       └── quartz.conf
│   └── logs/
├── ssl/                    # 证书目录
│   └── example.com/
│       ├── fullchain.pem
│       └── privkey.pem
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

**验证安装：**
打开 Git Bash，输入：
```bash
git --version
bash --version
```

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

**安装位置：**
```
C:/Users/你的用户名/.acme.sh/
```

### 3.3 配置阿里云 DNS 凭证

在 Git Bash 中执行：
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

**参数说明：**
| 参数 | 说明 |
|------|------|
| `--issue` | 申请证书 |
| `--dns dns_ali` | 使用阿里云 DNS 验证 |
| `-d` | 申请的域名，可多个 |

**证书默认位置：**
```
C:/Users/你的用户名/.acme.sh/example.com/
```

> ⚠️ 不要直接使用这个目录的文件！用 `--install-cert` 安装到指定目录

### 3.5 安装证书到 Nginx 目录

**创建证书目录（Windows 资源管理器或 CMD）：**
```cmd
mkdir D:\ssl\example.com
```

**在 Git Bash 中安装证书：**
```bash
acme.sh --install-cert -d example.com \
  --key-file       /d/ssl/example.com/privkey.pem \
  --fullchain-file /d/ssl/example.com/fullchain.pem \
  --reloadcmd      "D:/nginx/nginx.exe -s reload"
```

**注意：**
- Git Bash 中 Windows 路径写法：`D:/` → `/d/`
- `--reloadcmd` 中的路径用 Windows 格式（正斜杠）

**参数说明：**
| 参数 | 说明 |
|------|------|
| `--install-cert` | 安装证书 |
| `--key-file` | 私钥输出路径 |
| `--fullchain-file` | 证书链输出路径 |
| `--reloadcmd` | 证书更新后执行的命令 |

> ✅ 续期后自动复制新证书 + 自动重载 Nginx

---

## 四、Nginx 配置

### 4.1 安装 Nginx

**下载地址：**
[https://nginx.org/en/download.html](https://nginx.org/en/download.html)

**选择版本：**
- Stable version（稳定版）
- nginx/Windows-x64 版本

**安装步骤：**
1. 下载 zip 包
2. 解压到 `D:/nginx/`

**快速启动测试：**
打开 CMD，执行：
```cmd
cd /d D:/nginx
start nginx
tasklist | findstr nginx
```

**常用命令：**
```cmd
cd /d D:/nginx

nginx -t          # 检查配置
start nginx       # 启动
nginx -s stop     # 停止
nginx -s quit     # 优雅退出
nginx -s reload   # 重载配置（推荐）
```

### 4.2 站点配置文件

**第一步：修改主配置 `nginx.conf`**

文件路径：`D:/nginx/conf/nginx.conf`

```nginx
worker_processes  auto;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    sendfile        on;
    keepalive_timeout  65;

    # Gzip 压缩
    gzip  on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types
        text/plain text/css text/xml text/javascript
        application/json application/javascript application/xml
        image/svg+xml;

    # 引入站点配置
    include conf.d/*.conf;
}
```

**第二步：创建站点配置目录和文件**

创建目录：`D:/nginx/conf/conf.d/`

创建文件：`D:/nginx/conf/conf.d/quartz.conf`

写入内容：
```nginx
# ============================================
# Quartz 静态站点 - Windows Nginx + ESA 配置
# 端口：8443（ESA回源） + 5443（直连备用）
# ============================================

# ---------- 端口 8443：专供 ESA 回源 ----------
server {
    listen 8443 ssl;
    server_name example.com www.example.com origin.example.com;

    # SSL 证书（acme.sh 安装）
    ssl_certificate     D:/ssl/example.com/fullchain.pem;
    ssl_certificate_key D:/ssl/example.com/privkey.pem;

    # SSL 安全配置
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL_ESA:10m;
    ssl_session_timeout 10m;

    # 站点根目录（全局生效）
    root D:/quartz_build;
    index index.html;

    # 日志
    access_log  D:/nginx/logs/quartz-esa-access.log  main;
    error_log   D:/nginx/logs/quartz-esa-error.log   warn;

    # ---------- Quartz SPA 路由核心 ----------
    # 解决：页面刷新 404 问题
    location / {
        try_files $uri $uri/ $uri.html /index.html;
    }

    # ---------- 静态资源缓存 ----------
    location ~* \.(js|css|png|jpg|jpeg|svg|gif|ico|webp|woff|woff2|ttf|eot)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # ---------- 安全响应头 ----------
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    # ---------- 禁止访问隐藏文件 ----------
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }
}

# ---------- 端口 5443：用户直连备用 ----------
server {
    listen 5443 ssl;
    server_name example.com www.example.com;

    ssl_certificate     D:/ssl/example.com/fullchain.pem;
    ssl_certificate_key D:/ssl/example.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_session_cache shared:SSL_DIRECT:5m;

    root D:/quartz_build;
    index index.html;

    access_log  D:/nginx/logs/quartz-direct-access.log  main;
    error_log   D:/nginx/logs/quartz-direct-error.log   warn;

    location / {
        try_files $uri $uri/ $uri.html /index.html;
    }

    location ~* \.(js|css|png|jpg|jpeg|svg|gif|ico|webp|woff|woff2|ttf|eot)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
}
```

> ⚠️ **Windows 路径注意：**
> - ✅ 推荐：`D:/ssl/example.com/fullchain.pem`（正斜杠）
> - ❌ 避免：`D:\ssl\example.com\fullchain.pem`（单反斜杠，转义问题）

**第三步：验证配置并启动**

```cmd
cd /d D:/nginx

# 检查配置
nginx -t

# 启动
start nginx

# 或重载
nginx -s reload
```

### 4.3 注册为 Windows 服务

默认 Nginx 不是 Windows 服务，重启后需要手动启动。推荐用 NSSM 注册成服务。

**下载 NSSM：**
[https://nssm.cc/download](https://nssm.cc/download)

**解压到：** `D:/nssm/`

**注册服务（以管理员身份运行 CMD）：**
```cmd
cd /d D:/nssm/win64
nssm install Nginx
```

弹出图形界面：
- **Path**：`D:\nginx\nginx.exe`
- **Startup directory**：`D:\nginx`
- 点击「Install service」

**服务管理命令：**
```cmd
# 启动
net start Nginx

# 停止
net stop Nginx

# 重启
net stop Nginx && net start Nginx

# 删除服务
nssm remove Nginx
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
| 回源 HOST | `example.com` | 和 Nginx server_name 一致 |
| 回源协议 | **HTTPS** | 源站用 HTTPS |
| 回源端口 | **8443** | Nginx 监听的端口 |
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

### 6.1 放行 Nginx 端口

**方式一：PowerShell 命令（推荐）**

以管理员身份运行 PowerShell：

```powershell
# 放行 8443 端口（ESA 回源）
New-NetFirewallRule `
  -DisplayName "Nginx ESA 8443" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 8443 `
  -Action Allow

# 放行 5443 端口（直连备用）
New-NetFirewallRule `
  -DisplayName "Nginx Direct 5443" `
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
7. 名称：`Nginx HTTPS (8443,5443)`

### 6.2 进阶：只允许 ESA 回源 IP（可选）

更安全的做法：8443 端口只允许 ESA 回源 IP 访问。

```powershell
# 删除之前的通用规则
Remove-NetFirewallRule -DisplayName "Nginx ESA 8443"

# 新建只允许 ESA IP 的规则（替换为实际 IP 段）
New-NetFirewallRule `
  -DisplayName "Nginx ESA 8443 (ESA Only)" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 8443 `
  -RemoteAddress 100.100.100.0/24 `
  -Action Allow
```

---

## 七、自动续期配置

### 7.1 acme.sh 自动续期原理

acme.sh 安装后自动创建 Windows 计划任务：

**查看计划任务：**
```powershell
# 打开任务计划程序
taskschd.msc
# 找到：acme.sh 相关任务
```

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
cd /d D:/nginx

# 测试配置
nginx -t

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

### Q1: Git Bash 中 acme.sh 命令找不到
**原因：** 环境变量未生效
**解决：**
```bash
source ~/.bashrc
# 或重启 Git Bash
```

### Q2: DNS 验证失败
**排查：**
- AccessKey 是否正确
- RAM 用户是否有 `AliyunDNSFullAccess` 权限
- 加 `--debug` 参数查看详细日志：
  ```bash
  acme.sh --issue --dns dns_ali -d example.com --debug
  ```

### Q3: Nginx 启动失败，提示找不到证书
**原因：** 路径写错了
**解决：**
- 检查路径是否用了正斜杠 `D:/ssl/...`
- 确认证书文件确实存在
- 执行 `nginx -t` 看具体错误

### Q4: 页面刷新 404
**原因：** try_files 缺少 `$uri.html`
**解决：**
```nginx
location / {
    try_files $uri $uri/ $uri.html /index.html;
}
```

### Q5: ESA 回源 502 错误
**排查步骤：**
1. 源站 Nginx 是否运行？`tasklist | findstr nginx`
2. Windows 防火墙是否放行 8443 端口？
3. ESA 回源协议和端口是否匹配？
4. 回源 SNI 是否填写？
5. 查看 Nginx 错误日志

### Q6: 证书续期后 Nginx 没加载新证书
**原因：** 没有配置 `--reloadcmd`
**解决：**
```bash
# 重新安装证书，配置 reloadcmd
acme.sh --install-cert -d example.com \
  --key-file       /d/ssl/example.com/privkey.pem \
  --fullchain-file /d/ssl/example.com/fullchain.pem \
  --reloadcmd      "D:/nginx/nginx.exe -s reload"
```

### Q7: Windows 路径写法混乱
**Git Bash 中的路径：**
- Windows 路径 `D:/ssl/` → Git Bash 中 `/d/ssl/`

**Nginx 配置中的路径：**
- 用正斜杠 `D:/ssl/example.com/fullchain.pem`

**acme.sh --reloadcmd 中的路径：**
- 用 Windows 格式（正斜杠）`D:/nginx/nginx.exe -s reload`

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
- [ ] 安装证书到 `D:/ssl/example.com/`
- [ ] 配置 reloadcmd 自动重载 Nginx

### 第三阶段：Nginx 配置
- [ ] 下载 Nginx for Windows，解压到 `D:/nginx/`
- [ ] 修改主配置 `nginx.conf`
- [ ] 编写站点配置 `conf.d/quartz.conf`
- [ ] `nginx -t` 验证配置
- [ ] 启动 Nginx
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
- [ ] 本地测试 Nginx
- [ ] 外网直连测试
- [ ] ESA 加速测试
- [ ] 验证路由刷新不 404
- [ ] 验证证书有效
- [ ] 测试自动续期

---

## 相关笔记

- [Linux-Nginx-Certbot-ESA部署方案](Linux-Nginx-Certbot-ESA部署方案.md)
- [Linux-Nginx-acme.sh-ESA部署方案](Linux-Nginx-acme.sh-ESA部署方案.md)
- [Linux-Caddy-Certbot-ESA部署方案](Linux-Caddy-Certbot-ESA部署方案.md)
- [Linux-Caddy-acme.sh-ESA部署方案](Linux-Caddy-acme.sh-ESA部署方案.md)
- [Windows-Caddy-acme.sh-ESA部署方案](Windows-Caddy-acme.sh-ESA部署方案.md)
- [Quartz静态站点搭建与发布完整教程](Quartz静态站点搭建与发布完整教程.md)
