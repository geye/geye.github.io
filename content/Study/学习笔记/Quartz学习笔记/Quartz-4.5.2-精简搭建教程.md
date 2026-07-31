---
title: Quartz 4.5.2 精简搭建教程
description: Quartz 4.5.2 从零搭建完整教程，含全新安装、复制项目、常见问题排查
tags:
  - Quartz
  - 教程
  - 部署
date: 2026-07-20
---

# Quartz 4.5.2 精简搭建教程

> 版本：v4.5.2 | 更新：2026.07

---

## 📋 前置准备

### 环境要求
- **Node.js**：≥ v20（推荐 v22），终端执行 `node -v` 检查
- **Git**：已安装并配置，终端执行 `git --version` 检查

### 国内网络优化（可选）
```bash
npm config set registry https://registry.npmmirror.com
```

---

## 🚀 方案一：全新创建项目

### 1. 克隆 v4 分支
```bash
git clone -b v4 https://github.com/jackyzha0/quartz.git my-quartz-site
cd my-quartz-site
```

### 2. 安装依赖
```bash
npm install
```

### 3. 初始化项目
```bash
npx quartz create
```
> 提示：全新项目直接按回车确认即可

### 4. 本地预览
```bash
npx quartz build --serve
```
> 浏览器打开 http://localhost:8080

---

## 📋 方案二：复制现有项目

### 1. 复制并清理
复制项目文件夹后，删除以下内容：
- `.git` 文件夹（旧仓库关联）
- `.quartz` 文件夹（缓存，必须删）
- `node_modules` 文件夹（依赖，建议删）
- `public` 文件夹（构建产物，可选删）
- `content/` 下的旧笔记（替换为你的内容）

### 2. 重新安装依赖
```bash
cd 新项目文件夹
npm install
```

### 3. 本地预览
```bash
npx quartz build --serve
```

---

## ⚠️ 关键避坑（必看）

### 1. 注释 CustomOgImages 插件
国内网络必报错：`Failed to emit from plugin CustomOgImages: fetch failed`

**解决**：打开 `quartz.config.ts`，找到 plugins 配置，注释掉：
```typescript
// Plugin.CustomOgImages(),  // 国内网络必须注释
```

### 2. Node 版本问题
报错：`ERR_INVALID_ARG_TYPE` 或 `Unsupported engine`

**解决**：升级 Node.js 到 v22+

### 3. baseUrl 配置
配置错误会导致样式错乱、图片 404、链接失效

| 部署方式 | baseUrl 配置 |
|----------|-------------|
| 自定义域名 | `baseUrl: "example.com"` |
| GitHub 默认 | `baseUrl: "用户名.github.io/仓库名"` |

### 4. 不要手动创建 .quartz
`.quartz` 是缓存目录，由 `npx quartz create` 自动生成，手动创建会报错。

### 5. git clone仓库后切回自己仓库地址再上传
```bash
git remote set-url origin https://github.com/你的用户名/quartz.git
```

### 6. 重点注意仓库分支是main还是master

```bash
git branch -M main    #如果你是master，请修改为 git branch -M main 
git push -u origin main   #如果你是master，请修改为git push -u origin main
```
---

## 🔧 常用配置

### 修改网站标题
编辑 `quartz.config.ts`：
```typescript
pageTitle: "你的网站标题"
```

### 修改首页
编辑 `content/index.md`，替换为你自己的内容。

### 忽略文件
```typescript
ignorePatterns: [".obsidian", ".trash", "*.draft.md"]
```

---

## 📤 GitHub Pages 部署

### 1. 创建 GitHub 仓库
新建空仓库，不要勾选初始化 README。

### 2. 推送代码
```bash
git init
git add .
git commit -m "Initial commit"
git remote set-url origin https://github.com/你的用户名/quartz.git
git remote add origin https://github.com/用户名/仓库名.git
git branch -M main    #如果你是master，请修改为 git branch -M main 
git push -u origin main   #如果你是master，请修改为git push -u origin main
```

### 3. 配置自动部署
创建 `.github/workflows/deploy.yml`：
```yaml
name: Deploy Quartz site to GitHub Pages
on:
  push:
    branches: [main]    # 如果你的主分支叫 master，请改为 master
permissions:
  contents: read
  pages: write
  id-token: write
jobs:
  build:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: "npm" }
      - run: npm ci
      - run: npx quartz build
      - uses: actions/upload-pages-artifact@v3
        with: { path: public }
  deploy:
    needs: build
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

### 4. 开启 Pages
仓库 Settings → Pages → Source 选 **GitHub Actions**

---

## 🌐 自定义域名配置

### DNS 解析配置

**根域名（如 example.com）**：添加 4 条 A 记录
| 主机记录 | 类型 | 记录值 |
|---------|------|--------|
| @ | A | 185.199.108.153 |
| @ | A | 185.199.109.153 |
| @ | A | 185.199.110.153 |
| @ | A | 185.199.111.153 |

**子域名（如 notes.example.com）**：添加 1 条 CNAME
| 主机记录 | 类型 | 记录值 |
|---------|------|--------|
| notes | CNAME | 用户名.github.io |

### GitHub 设置
仓库 Settings → Pages → Custom domain 填写域名 → 保存

### 重要：CNAME 文件
GitHub 会自动生成 `CNAME` 文件，**必须提交到 main 或者 maser 分支**，根据第一次提交相同，否则每次部署会丢失域名。

### 开启 HTTPS
等待 DNS 生效后（10分钟~24小时），勾选 **Enforce HTTPS**。

---

## 🔄 日常更新流程

### 标准三步
```bash
git add .
git commit -m "更新说明"
git push
```

推送后 GitHub Actions 自动构建，1~3 分钟后线上更新。

### 本地预览（推荐）
```bash
npx quartz build --serve
```
修改后自动热重载，确认没问题再推送。

---

## ❓ 常见问题

### Q1: 构建报错 fetch failed
**原因**：CustomOgImages 插件请求外网失败  
**解决**：`quartz.config.ts` 中注释 `Plugin.CustomOgImages()`

### Q2: npm error Unsupported engine
**原因**：Node 版本太低  
**解决**：升级 Node.js 到 v22+

### Q3: 页面样式错乱、图片 404
**原因**：baseUrl 配置错误  
**解决**：检查 baseUrl 是否与实际访问地址一致

### Q4: 域名访问 404
**原因**：CNAME 文件未提交  
**解决**：确认仓库根目录有 CNAME 文件且内容正确

### Q5: Enforce HTTPS 灰色不可点
**原因**：DNS 未生效或证书未签发  
**解决**：等待 1 小时后重试

### Q6: 复制项目后构建失败
**原因**：.quartz 缓存或 node_modules 不兼容  
**解决**：删除 .quartz 和 node_modules，重新 npm install

### Q7: 修改文件夹名有影响吗？
**答**：不影响 Git 和部署，只影响本地路径。改名后重新在 Obsidian/编辑器中打开即可。


### Q8: git push时报如下错误：
	$ git push -u origin master
	error: src refspec master does not match any
	error: failed to push some refs to 'https://github.com/jackyzha0/quartz.git'

**原因**：当前还在jackyzha0的仓库地址  
**解决**：切回自己的仓库地址再执行推送
```bash
git remote set-url origin https://github.com/你的用户名/quartz.git
```

---

## 📝 新建独立仓库完整流程

从旧项目复制出全新独立项目：

1. 复制项目文件夹，重命名
2. 删除 `.git` 文件夹（关键！）
3. 删除 `.quartz` 和 `node_modules`
4. 修改 `quartz.config.ts` 中的 pageTitle 和 baseUrl
5. 替换 `content/` 内容
6. `npm install`
7. `npx quartz build --serve` 本地测试
8. GitHub 新建空仓库
9. `git init` → `git add .` → `git commit -m "Initial"`
10. `git remote add origin 仓库地址` → `git branch -M main 或master` → `git push -u origin main 或 master`
11. 配置 Pages 和 Actions

---

> 💡 **提示**：遇到问题先看常见问题，90% 的坑都在里面。
>
> 官方文档（v4）：https://four.quartz.jzhao.xyz/
