# 02-仓库(Repository)操作
[[README.md|返回总目录]]
[[08-Git设置与本地协同]] ← 本地Git与仓库同步配套阅读

## 1. New repository 新建仓库页面翻译
点击右上角 + → New repository
- Repository name：仓库名称（必填，英文小写推荐）
- Description：项目简介（选填）
- Public：公开仓库，所有人可见代码，免费
- Private：私有仓库，仅授权人员查看
- Add a README file：自动创建README.md（仓库首页文档，强烈勾选）
- Add .gitignore：Git忽略文件模板（选择开发语言，自动过滤不需要上传的文件）
- Choose a license：开源协议（开源项目必须选择）

点击 Create repository 创建仓库

## 2. 仓库基础操作
### Clone（克隆）
把线上仓库完整下载到本地电脑，三种方式：
1. HTTPS：最简单，每次推送代码需要输入账号密码/Token
2. SSH：配置密钥后免密码推送，推荐使用，参考 [[08-Git设置与本地协同]]

### Branch 分支操作
1. 仓库首页分支下拉框：main（主干分支）
> 旧仓库默认叫 master，新仓库统一 main
- Create branch：新建分支，用于开发新功能，不直接改动主干
- Compare：对比两个分支代码差异

### Release（版本发布）
路径：仓库页面 → Releases → Create a new release
- Tag version：版本号，例 v1.0.0
- Target：选择对应的分支（一般main）
- Release title：版本标题
- Describe this release：更新日志
- Attach binaries：上传程序压缩包

## 3. 仓库设置常用选项（Settings）
- General：仓库基础信息，修改名称、简介、开启Wiki、Discussions
- Branches：分支保护规则（禁止直接向main推送代码，强制走PR）
- Pages：GitHub Pages静态网站配置 [[06-GitHub Pages静态站点]]
- Actions：工作流权限、缓存配置 [[03-GitHub Actions工作流]]
- Secrets and variables：密钥环境变量（Actions存放密码、密钥核心位置）
- Danger Zone：危险区域！包含仓库改名、转移、删除仓库
> ⚠️ Delete this repository = 删除仓库，操作不可逆，谨慎点击！