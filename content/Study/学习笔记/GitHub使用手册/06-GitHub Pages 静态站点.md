# 06-GitHub Pages静态站点
[[README.md|返回总目录]]
[[03-GitHub Actions工作流]]

## 作用
GitHub Pages：免费静态网页托管，常用于 Quartz Obsidian知识库、文档网站、博客
对应流水线：Deploy Quartz site to GitHub Pages

## 设置入口
仓库 → Settings → Pages
### 页面翻译
Source：部署来源
1. Deploy from a branch
   从指定分支部署（早期方案，直接读取分支html文件）
2. GitHub Actions
   通过Actions流水线自动构建部署（**Quartz推荐方案**，你当前项目使用方案）

### Quartz站点部署流程
1. Actions工作流拉取Obsidian笔记
2. 构建完整静态HTML站点（Build步骤）
3. 自动推送产物至pages分支
4. GitHub Pages读取分支对外提供网页访问

### 域名
默认域名格式：用户名.github.io/仓库名
支持绑定自有域名 Custom domain

⚠️ 注意：私有仓库开启Pages有功能限制；公开仓库无流量限制。