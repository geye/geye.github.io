# Linux + Nginx + Certbot + ESA 部署方案

> 场景：Linux 服务器 + Nginx + Certbot（DNS-01 验证）+ 阿里云 ESA，解决 80/443 端口封禁问题

---

## 目录

- [[#一、方案概述]]
- [[#二、环境准备]]
- [[#三、Certbot 安装与证书申请]]
  - [[#3.1 安装 Certbot]]
  - [[#3.2 安装阿里云 DNS 插件]]
  - [[#3.3 配置阿里云凭证]]
  - [[#3.4 申请泛域名证书]]
- [[#四、Nginx 配置]]
  - [[#4.1 安装 Nginx]]
  - [[#4.2 站点配置文件]]
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
Linux Nginx
  (监听 8443 端口)
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
/etc/nginx/conf.d/        # Nginx 站点配置
/etc/letsencrypt/live/    # Certbot 证书目录
```

### 准备阿里云 AccessKey

1. 登录阿里云控制台 → 访问控制 RAM
2. 创建用户，勾选「OpenAPI 访问」
3. 授权策略：`AliyunDNSFullAccess`
4. 保存 AccessKey ID 和 AccessKey Secret

> 💡 遵循最小权限原则，只授予 DNS 管理权限

---

## 三、Certbot 安装与证书申请

### 3.1 安装 Certbot

**Ubuntu/Debian：**
```bash
sudo apt update
sudo apt install certbot python3-pip -y
```

**CentOS/RHEL：**
```bash
sudo yum install epel-release -y
sudo yum install certbot python3-pip -y
```

### 3.2 安装阿里云 DNS 插件

```bash
sudo pip3 install certbot-dns-aliyun
```

> ⚠️ 注意：插件名可能是 `certbot-dns-aliyun` 或 `certbot-dns-ali`，根据实际情况选择

### 3.3 配置阿里云凭证

创建凭证文件：
```bash
sudo mkdir -p /etc/letsencrypt
sudo nano /etc/letsencrypt/aliyun.ini
```

写入内容：
```ini
dns_aliyun_access_key = 你的AccessKeyID
dns_aliyun_secret_key = 你的AccessKeySecret
```

设置权限（重要！）：
```bash
sudo chmod 600 /etc/letsencrypt/aliyun.ini
```

### 3.4 申请泛域名证书

```bash
sudo certbot certonly \
  --dns-aliyun \
  --dns-aliyun-credentials /etc/letsencrypt/aliyun.ini \
  -d example.com \
  -d *.example.com \
  --agree-tos \
  --email your-email@example.com
```

**参数说明：**
| 参数 | 说明 |
|------|------|
| `certonly` | 只申请证书，不自动配置 Web 服务器 |
| `--dns-aliyun` | 使用阿里云 DNS 验证 |
| `--dns-aliyun-credentials` | 阿里云凭证文件路径 |
| `-d` | 申请的域名，可多个 |
| `--agree-tos` | 同意服务条款 |
| `--email` | 邮箱，用于证书过期提醒 |

**证书输出位置：**
```
/etc/letsencrypt/live/example.com/
├── fullchain.pem    # 证书链（Nginx 用）
├── privkey.pem      # 私钥（Nginx 用）
├── cert.pem         # 单证书
└── chain.pem        # 中间证书
```

---

## 四、Nginx 配置

### 4.1 安装 Nginx

**Ubuntu/Debian：**
```bash
sudo apt install nginx -y
```

**CentOS/RHEL：**
```bash
sudo yum install nginx -y
sudo systemctl enable nginx
```

### 4.2 站点配置文件

创建配置文件：
```bash
sudo nano /etc/nginx/conf.d/quartz-esa.conf
```

写入内容：
```nginx
# ============================================
# Quartz 静态站点 - Nginx + ESA 配置
# 端口：8443（ESA回源） + 5443（直连备用）
# ============================================

# ---------- 端口 8443：专供 ESA 回源 ----------
server {
    listen 8443 ssl;
    server_name example.com www.example.com origin.example.com;

    # SSL 证书（Certbot 生成）
    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # SSL 安全配置
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL_ESA:10m;
    ssl_session_timeout 10m;

    # 站点根目录（全局生效）
    root /var/www/quartz;
    index index.html;

    # 日志
    access_log  /var/log/nginx/quartz-esa-access.log;
    error_log   /var/log/nginx/quartz-esa-error.log warn;

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

    # ---------- 真实客户端 IP ----------
    # 从 ESA 透传的 X-Forwarded-For 获取
    # set_real_ip_from  100.100.100.0/24;  # 替换为 ESA 回源 IP 段
    # real_ip_header    X-Forwarded-For;
    # real_ip_recursive on;

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

    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_session_cache shared:SSL_DIRECT:5m;

    root /var/www/quartz;
    index index.html;

    access_log  /var/log/nginx/quartz-direct-access.log;
    error_log   /var/log/nginx/quartz-direct-error.log warn;

    location / {
        try_files $uri $uri/ $uri.html /index.html;
    }

    location ~* \.(js|css|png|jpg|jpeg|svg|gif|ico|webp|woff|woff2|ttf|eot)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
}
```

### 4.3 关键配置说明

**1. try_files 详解（Quartz 必看）**
```nginx
try_files $uri $uri/ $uri.html /index.html;
```
按顺序查找：
1. `$uri` → 精确匹配文件
2. `$uri/` → 匹配目录下的 index.html
3. `$uri.html` → 自动补 .html 后缀（Quartz 关键！）
4. `/index.html` → 都找不到，交给 SPA 路由

> 🎯 `$uri.html` 是解决 Quartz 路由问题的核心，很多教程漏掉导致刷新 404

**2. root 位置**
- ✅ 推荐：放在 `server {}` 块，全局生效
- ❌ 不推荐：只放在 `location /`，新增其他 location 会 404

**3. 双端口设计**
- 8443：专供 ESA 回源，日志独立
- 5443：用户直连备用
- 分开管理，便于排查问题

**验证配置并启动：**
```bash
# 检查配置语法
sudo nginx -t

# 重载配置
sudo systemctl reload nginx

# 或重启
sudo systemctl restart nginx
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

> ⚠️ 没有备案必须选「全球（不包含中国内地）」，否则添加失败

### 5.2 DNS 解析配置

在阿里云 DNS（或其他域名服务商）添加记录：

| 记录类型 | 主机记录 | 记录值 | 说明 |
|----------|----------|--------|------|
| CNAME | `@` | ESA 提供的 CNAME 地址 | 主域名走 ESA |
| CNAME | `www` | ESA 提供的 CNAME 地址 | www 走 ESA |
| A | `origin` | 你的服务器公网 IP | 源站域名，直连 |

> 💡 `origin.example.com` 作为源站域名，直接解析到服务器，ESA 回源用

**验证解析生效：**
```bash
dig example.com
# 或
nslookup example.com
```

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
4. 等待签发（通常几分钟）

**SSL/TLS 加密模式：**
- 选择「**完整**」模式
- 因为源站有有效证书（Certbot 签发）

| 模式 | 用户→ESA | ESA→源站 | 适用场景 |
|------|----------|----------|----------|
| 灵活 | HTTPS | HTTP | 源站只有 HTTP |
| **完整** | HTTPS | HTTPS | 源站有 HTTPS 证书 |
| 完整（严格） | HTTPS | HTTPS（校验） | 源站有有效正式证书 |

---

## 六、自动续期配置

### 6.1 Certbot 自动续期

Certbot 安装后会自动创建定时任务：

**检查续期定时器：**
```bash
# Ubuntu/Debian
systemctl list-timers | grep certbot

# 或查看 cron
sudo crontab -l | grep certbot
```

**手动测试续期：**
```bash
sudo certbot renew --dry-run
```

### 6.2 续期后自动重载 Nginx

证书续期后需要重载 Nginx 才能生效。

**创建续期钩子脚本：**
```bash
sudo nano /etc/letsencrypt/renewal-hooks/post/reload-nginx.sh
```

写入内容：
```bash
#!/bin/bash
systemctl reload nginx
echo "$(date) Nginx reloaded after cert renewal" >> /var/log/letsencrypt/renew.log
```

设置执行权限：
```bash
sudo chmod +x /etc/letsencrypt/renewal-hooks/post/reload-nginx.sh
```

> ✅ 这样每次证书自动续期后，会自动重载 Nginx

---

## 七、验证测试

### 7.1 本地测试

```bash
# 测试 Nginx 配置
sudo nginx -t

# 测试 HTTPS 访问
curl -k https://localhost:8443
curl -k https://localhost:5443
```

### 7.2 外网直连测试

用手机流量或其他网络访问：
```
https://origin.example.com:5443
```
> 应该能正常打开网站

### 7.3 ESA 加速测试

访问主域名（不带端口）：
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

1. 点击浏览器 🔒 图标
2. 查看证书信息
3. 确认颁发者、有效期
4. 确认域名匹配

---

## 八、常见问题排错

### Q1: Certbot 插件安装失败
**原因：** Python 版本或 pip 源问题
**解决：**
```bash
# 升级 pip
sudo pip3 install --upgrade pip

# 使用国内源
sudo pip3 install certbot-dns-aliyun -i https://pypi.tuna.tsinghua.edu.cn/simple
```

### Q2: DNS 验证失败
**排查：**
- AccessKey 是否正确
- RAM 用户是否有 `AliyunDNSFullAccess` 权限
- 域名是否在阿里云 DNS 管理
- 查看 Certbot 详细日志：`/var/log/letsencrypt/letsencrypt.log`

### Q3: Nginx 启动失败，SSL 报错
**排查：**
```bash
sudo nginx -t
```
- 检查证书路径是否正确
- 检查证书文件权限（Nginx 进程需可读）
- 检查证书和私钥是否匹配

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
1. 源站 Nginx 是否正常运行？
2. 防火墙是否放行 8443 端口？
3. ESA 回源协议和端口是否匹配？
4. 回源 SNI 是否填写？
5. 查看 Nginx 错误日志

### Q6: 证书续期后 Nginx 没加载新证书
**原因：** 没有配置续期钩子
**解决：**
```bash
# 手动重载
sudo systemctl reload nginx

# 配置自动重载钩子（见第六章）
```

---

## 九、部署 Checklist

### 第一阶段：环境准备
- [ ] Linux 服务器可访问公网
- [ ] 域名托管在阿里云 DNS
- [ ] 创建 RAM 子用户，获取 AccessKey
- [ ] 上传 Quartz 静态文件到 `/var/www/quartz/`

### 第二阶段：证书申请
- [ ] 安装 Certbot
- [ ] 安装阿里云 DNS 插件
- [ ] 配置凭证文件
- [ ] 申请泛域名证书
- [ ] 验证证书文件存在

### 第三阶段：Nginx 配置
- [ ] 安装 Nginx
- [ ] 编写站点配置文件（8443 + 5443）
- [ ] `nginx -t` 验证配置
- [ ] 启动/重载 Nginx
- [ ] 防火墙放行 8443、5443 端口

### 第四阶段：ESA 配置
- [ ] 开通 ESA 服务
- [ ] 添加站点（全球不含中国内地）
- [ ] 配置 DNS CNAME 记录
- [ ] 配置回源规则（HTTPS + 8443 + SNI）
- [ ] 申请边缘证书
- [ ] 设置 SSL 模式为「完整」

### 第五阶段：自动续期
- [ ] 配置续期后重载 Nginx 钩子
- [ ] 测试续期流程

### 第六阶段：测试验证
- [ ] 本地测试 Nginx
- [ ] 外网直连测试
- [ ] ESA 加速测试
- [ ] 验证路由刷新不 404
- [ ] 验证证书有效

---

## 相关笔记

- [Linux-Nginx-acme.sh-ESA部署方案.md](Linux-Nginx-acme.sh-ESA部署方案.md)
- [Linux-Caddy-Certbot-ESA部署方案.md](Linux-Caddy-Certbot-ESA部署方案.md)
- [Linux-Caddy-acme.sh-ESA部署方案.md](Linux-Caddy-acme.sh-ESA部署方案.md)
- [Windows-Nginx-acme.sh-ESA部署方案.md](Windows-Nginx-acme.sh-ESA部署方案.md)
- [Windows-Caddy-acme.sh-ESA部署方案.md](Windows-Caddy-acme.sh-ESA部署方案.md)
- [Quartz静态站点搭建与发布完整教程](Quartz静态站点搭建与发布完整教程.md)