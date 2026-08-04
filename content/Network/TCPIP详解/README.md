
---

## 2. GitHub 首页说明 (`00-README.md`)

# TCP/IP详解 · 学习笔记

本仓库是对《TCP/IP详解》卷1和卷2的系统性整理，采用 Obsidian 风格的 Markdown 格式，支持知识图谱和双向链接。

## ✨ 特点

- 📖 完整覆盖卷1（协议）和卷2（实现）
- 🔗 内置 Obsidian 双链接，便于知识跳转
- 🌐 支持 Quartz 生成静态站点
- 📂 按章分文件，结构清晰

## 📁 目录结构
content/
├── index.md # Quartz 首页
├── TCPIP详解-卷1/ # 卷1：协议（30章 + 附录）
└── TCPIP详解-卷2/ # 卷2：实现（32章）

text

## 🔧 构建站点

```bash
git clone <this-repo>
cd <repo>
npx quartz build --serve
📚 内容来源
《TCP/IP详解 卷1：协议》- W. Richard Stevens

《TCP/IP详解 卷2：实现》- Gary R. Wright, W. Richard Stevens

📄 许可证
仅供个人学习使用，请勿用于商业用途。