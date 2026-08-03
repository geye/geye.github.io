# 08-Git设置与本地协同
[[README.md|返回总目录]]
[[02-仓库(Repository)操作]]

## 1. 两种连接方式
### HTTPS
地址格式：https://github.com/用户名/仓库名.git
新版GitHub不再支持账号密码登录，必须使用【Personal access token】代替密码

### SSH（推荐长期使用）
地址格式：git@github.com:用户名/仓库名.git
需要本地生成SSH密钥，公钥添加到GitHub账号设置，一次配置永久免密推送

## 2. 基础Git常用命令
```bash
# 克隆仓库
git clone 仓库地址
# 查看当前分支
git branch
# 新建并切换分支
git checkout -b 分支名
# 添加所有变更文件
git add .
# 提交变更
git commit -m "填写更新说明"
# 推送到线上仓库
git push
# 拉取线上最新代码
git pull

## 3. Token 获取路径（网页菜单翻译）
右上角头像 → Settings → Developer settings → Personal access tokens → Tokens (classic)
Generate new token：生成令牌，勾选 repo 权限，复制保存，只显示一次！

## 4. SSH 密钥配置位置
头像 → Settings → SSH and GPG keys → New SSH key
标题随意填写，粘贴本地生成的公钥文本保存。