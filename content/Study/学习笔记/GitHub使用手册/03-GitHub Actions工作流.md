# [[03-GitHub Actions 工作流]]
> 锚点目录 | 适配Quartz 4.5.2
## 目录
- [[03-GitHub Actions 工作流#基础概念介绍|基础概念介绍]]
- [[03-GitHub Actions 工作流#关键名词中英翻译|关键名词中英翻译]]
- [[03-GitHub Actions 工作流#Quartz4.5.2 完整yml配置范例|Quartz4.5.2 完整yml配置范例]]
- [[03-GitHub Actions 工作流#部署常见注意事项|部署常见注意事项]]

## 基础概念介绍
GitHub Actions 是 GitHub 内置自动化工具，可以实现：代码自动构建、自动测试、自动部署。
对于 Quartz 知识库：实现**提交笔记后自动编译、自动发布静态博客网站**。

> 英文界面提示：Actions = 自动化工作流

## 关键名词中英翻译
|英文原文|中文翻译|说明|
| ---- | ---- | ---- |
|Workflow|工作流|整套自动化任务|
|Job|任务|工作流中一组步骤|
|Step|步骤|每一条执行指令|
|Runner|运行器|执行脚本的虚拟环境|
|.yml / .yaml|配置文件|工作流规则文件，存放路径：`.github/workflows/`

## Quartz4.5.2 完整yml配置范例
> 文件路径：`.github/workflows/deploy.yml`
> ⚠️ 该配置**仅限 Quartz 4.5.2 使用，Quartz5 无法兼容**
```yaml
name: Deploy Quartz Site
# 触发条件：main分支推送文件时运行
on:
  push:
    branches: [main]

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build Quartz 4.5.2 site
        run: npx quartz build

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public

**使用操作步骤**
1.在仓库根目录新建文件夹 .github/workflows/
2.新建文件 deploy.yml，复制上方代码
3.提交推送代码，Actions 将自动执行构建部署

**部署常见注意事项**
1.仓库设置 → Pages：部署分支选择 gh-pages，文件夹选择 (root)
2.必须确认 Node 版本 ≥18，推荐 20
3.如果报错，查阅 [[Git&GitHub 常见英文报错中英对照]]

[[←返回 README.md]]

