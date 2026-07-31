# 📚 Quartz 4.5.2 知识库

> 基于 Obsidian + Quartz 4.5.2 搭建的个人数字花园 / 知识库

---

## ✨ 项目特点

- 📝 **Obsidian 原生语法** - 完美支持双链、标签、Callout 等
- 🔍 **全文搜索** - 快速找到需要的内容
- 🔗 **双向链接** - 自动生成反向链接，发现知识关联
- 🕸️ **知识图谱** - 可视化展示笔记之间的关系
- 📱 **响应式设计** - 完美适配电脑、平板、手机
- 🚀 **自动部署** - 推送代码后 GitHub Actions 自动构建部署
- 🆓 **完全免费** - 基于 GitHub Pages，零成本运行

---

## 🚀 快速开始

### 环境要求

- Node.js >= 22
- Git
- GitHub 账号

### 本地运行

```bash
# 克隆项目
git clone https://github.com/mocoss/quartz.git
cd 你的仓库名

# 安装依赖
npm install

# 初始化（首次运行）
npx quartz create

# 本地预览
npx quartz build --serve
```

浏览器打开 `http://localhost:8080` 查看效果。

### 部署到 GitHub Pages

1. 推送代码到 GitHub
2. 仓库 Settings → Pages → Source 选择 **GitHub Actions**
3. 等待 Actions 自动构建完成
4. 访问 `https://你的用户名.github.io/你的仓库名/`

---

## 📁 项目结构

```
.
├── content/              # Markdown 笔记内容
│   ├── index.md          # 首页
│   ├── 00-Inbox/         # 收件箱
│   ├── 01-Daily/         # 日记
│   ├── 02-Projects/      # 项目笔记
│   ├── 03-Areas/         # 领域知识
│   ├── 04-Resources/     # 参考资料
│   ├── 05-Notes/         # 永久笔记
│   ├── MOC/              # 知识地图
│   └── Attachments/      # 图片/附件
├── quartz/               # Quartz 核心源码
├── quartz.config.ts      # 全局配置
├── quartz.layout.ts      # 页面布局配置
├── package.json          # 依赖配置
└── .github/workflows/
    └── deploy.yml        # 自动部署配置
```

---

## 🛠️ 技术栈

| 技术 | 用途 |
|------|------|
| Quartz 4.5.2 | 静态网站生成器 |
| Obsidian | 笔记编辑工具 |
| Markdown | 内容格式 |
| GitHub Pages | 网站托管 |
| GitHub Actions | 自动构建部署 |
| Git | 版本控制 |

---

## 📖 相关文档

- [Quartz 官方文档（v4）](https://four.quartz.jzhao.xyz/)
- [Obsidian 官网](https://obsidian.md/)
- [GitHub Pages 文档](https://docs.github.com/en/pages)

---

## 📝 更新笔记

- **2026-07** - 初始化项目，基于 Quartz 4.5.2

---

## 📄 许可证

MIT License

---

> 💡 **提示**：本项目由 Quartz 4.5.2 驱动，使用 Obsidian 编写内容。
