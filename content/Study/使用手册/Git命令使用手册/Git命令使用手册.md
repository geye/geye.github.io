# Git 命令手册

> 适用于日常开发的 Git 速查手册，按场景分类整理。
> 标签：#Git #版本控制 #开发工具

---

## 📑 目录

- [[#一、基础概念|一、基础概念]]
- [[#二、仓库初始化与配置|二、仓库初始化与配置]]
- [[#三、文件状态与暂存|三、文件状态与暂存]]
- [[#四、提交操作|四、提交操作]]
- [[#五、分支管理|五、分支管理]]
- [[#六、远程仓库|六、远程仓库]]
- [[#七、拉取与推送|七、拉取与推送]]
- [[#八、撤销与回滚|八、撤销与回滚]]
- [[#九、标签管理|九、标签管理]]
- [[#十、日志与查看|十、日志与查看]]
- [[#十一、储藏与暂存|十一、储藏与暂存]]
- [[#十二、合并与变基|十二、合并与变基]]
- [[#十三、常见问题解决|十三、常见问题解决]]
- [[#十四、工作流示例|十四、工作流示例]]

---

## 一、基础概念

### Git 三区模型
1. **工作区（Working Directory）**：你编辑的文件
2. **暂存区（Staging Area / Index）**：`git add` 后的文件
3. **仓库区（Repository）**：`git commit` 后的历史版本

### 文件状态
- `Untracked`：未跟踪，新文件
- `Modified`：已修改
- `Staged`：已暂存
- `Committed`：已提交

---

## 二、仓库初始化与配置

### 初始化仓库
```bash
# 在当前目录初始化新仓库
git init

# 克隆远程仓库到本地
git clone <仓库地址>

# 克隆并指定本地目录名
git clone <仓库地址> <目录名>

# 克隆指定分支
git clone -b <分支名> <仓库地址>
```

### 配置用户信息
```bash
# 全局配置（所有仓库）
git config --global user.name "你的名字"
git config --global user.email "你的邮箱@example.com"

# 当前仓库配置（优先级高于全局）
git config user.name "你的名字"
git config user.email "你的邮箱@example.com"

# 查看配置
git config --list
git config user.name
```

### 常用配置
```bash
# 设置默认分支名为 main
git config --global init.defaultBranch main

# 设置默认编辑器为 vim
git config --global core.editor "vim"

# 开启颜色显示
git config --global color.ui auto

# 设置换行符处理（Windows推荐）
git config --global core.autocrlf true

# 设置换行符处理（Mac/Linux推荐）
git config --global core.autocrlf input
```

---

## 三、文件状态与暂存

### 查看状态
```bash
# 查看当前文件状态
git status

# 简洁状态
git status -s
```

### 添加文件到暂存区
```bash
# 添加单个文件
git add <文件名>

# 添加所有修改和新增文件
git add .

# 添加所有修改和删除（不含新增）
git add -u

# 交互式添加
git add -i

# 按补丁添加（选择部分修改）
git add -p
```

### 取消暂存
```bash
# 取消暂存某个文件（保留修改）
git reset HEAD <文件名>

# 取消所有暂存
git reset HEAD .
```

---

## 四、提交操作

### 提交代码
```bash
# 提交暂存区内容
git commit -m "提交说明"

# 跳过暂存，直接提交所有已跟踪文件的修改
git commit -am "提交说明"

# 修改上一次提交（未推送到远程时使用）
git commit --amend -m "新的提交说明"

# 修改上一次提交但保留原提交信息
git commit --amend --no-edit
```

### 提交规范建议
```
feat: 新功能
fix: 修复bug
docs: 文档更新
style: 格式调整（不影响代码运行）
refactor: 重构（既不新增功能也不修bug）
perf: 性能优化
test: 测试相关
chore: 构建/工具相关
```

示例：
```bash
git commit -m "feat: 添加用户登录功能"
git commit -m "fix: 修复首页加载慢的问题"
```

---

## 五、分支管理

### 查看分支
```bash
# 查看本地分支
git branch

# 查看所有分支（本地+远程）
git branch -a

# 查看远程分支
git branch -r

# 查看分支及最后提交
git branch -v

# 查看已合并到当前分支的分支
git branch --merged

# 查看未合并到当前分支的分支
git branch --no-merged
```

### 创建与切换分支
```bash
# 创建新分支（不切换）
git branch <分支名>

# 切换到指定分支
git checkout <分支名>

# 创建并切换到新分支
git checkout -b <分支名>

# 基于指定分支创建新分支
git checkout -b <新分支名> <基础分支>

# Git 2.23+ 新命令（更清晰）
git switch <分支名>
git switch -c <新分支名>
```

### 删除分支
```bash
# 删除已合并的分支
git branch -d <分支名>

# 强制删除分支（未合并也删）
git branch -D <分支名>

# 删除远程分支
git push origin --delete <分支名>
```

### 重命名分支
```bash
# 重命名当前分支
git branch -m <新分支名>

# 重命名指定分支
git branch -m <旧分支名> <新分支名>
```

---

## 六、远程仓库

### 查看远程仓库
```bash
# 查看远程仓库名称
git remote

# 查看远程仓库详细地址
git remote -v

# 查看远程仓库信息
git remote show origin
```

### 添加与删除远程
```bash
# 添加远程仓库
git remote add <别名> <地址>

# 修改远程仓库地址
git remote set-url origin <新地址>

# 删除远程仓库
git remote remove <别名>

# 重命名远程仓库
git remote rename <旧名> <新名>
```

### 同步远程分支信息
```bash
# 获取远程所有分支信息（不合并）
git fetch origin

# 获取所有远程仓库信息
git fetch --all

# 删除远程已不存在的本地跟踪分支
git fetch -p
# 或
git remote prune origin
```

---

## 七、拉取与推送

### 拉取代码
```bash
# 拉取并合并当前分支对应的远程分支
git pull

# 等价于 git fetch + git merge
git pull origin <分支名>

# 使用 rebase 方式拉取（更干净的提交历史）
git pull --rebase

# 拉取所有远程分支
git pull --all
```

### 推送代码
```bash
# 推送到远程仓库
git push origin <分支名>

# 设置上游分支并推送（首次推送新分支）
git push -u origin <分支名>
# 或
git push --set-upstream origin <分支名>

# 强制推送（⚠️ 危险，会覆盖远程历史）
git push -f

# 安全强制推送（只有远程没有新提交时才强制）
git push --force-with-lease

# 推送所有分支
git push --all

# 推送标签
git push --tags
```

---

## 八、撤销与回滚

### 工作区撤销
```bash
# 丢弃工作区修改（未 add 的文件）
git checkout -- <文件名>

# 丢弃所有工作区修改
git checkout -- .

# Git 2.23+ 新命令
git restore <文件名>
git restore .
```

### 暂存区撤销
```bash
# 从暂存区撤回（保留工作区修改）
git reset HEAD <文件名>

# Git 2.23+ 新命令
git restore --staged <文件名>
```

### 提交回滚
```bash
# 回滚到上一次提交（保留修改到暂存区）
git reset --soft HEAD~1

# 回滚到上一次提交（保留修改到工作区，默认）
git reset --mixed HEAD~1
# 或简写
git reset HEAD~1

# 回滚到上一次提交（⚠️ 丢弃所有修改）
git reset --hard HEAD~1

# 回滚到指定提交
git reset --hard <commit-hash>

# 创建新提交来撤销某次提交（安全，不会改写历史）
git revert <commit-hash>
```

### 常用 HEAD 表示
```
HEAD      当前最新提交
HEAD~1    上一次提交
HEAD~2    上两次提交
HEAD^     上一次提交（与~1类似）
HEAD@{0}  当前状态
HEAD@{1}  上一个状态
```

---

## 九、标签管理

### 查看标签
```bash
# 列出所有标签
git tag

# 搜索标签
git tag -l "v1.*"

# 查看标签详情
git show <标签名>
```

### 创建标签
```bash
# 轻量标签
git tag <标签名>

# 附注标签（推荐，包含完整信息）
git tag -a <标签名> -m "标签说明"

# 给指定提交打标签
git tag -a <标签名> <commit-hash> -m "标签说明"
```

### 推送与删除标签
```bash
# 推送单个标签到远程
git push origin <标签名>

# 推送所有标签
git push --tags

# 删除本地标签
git tag -d <标签名>

# 删除远程标签
git push origin --delete <标签名>
# 或
git push origin :refs/tags/<标签名>
```

---

## 十、日志与查看

### 查看提交日志
```bash
# 完整日志
git log

# 简洁日志（一行一条）
git log --oneline

# 图形化分支展示
git log --graph --oneline --all

# 显示最近 N 条
git log -n 5

# 按作者筛选
git log --author="用户名"

# 按时间筛选
git log --since="2024-01-01" --until="2024-12-31"

# 搜索提交信息
git log --grep="关键词"

# 查看某个文件的修改历史
git log -- <文件名>

# 查看每次提交的具体改动
git log -p
```

### 查看差异
```bash
# 查看工作区与暂存区差异
git diff

# 查看暂存区与最新提交差异
git diff --cached
# 或
git diff --staged

# 查看工作区与最新提交差异
git diff HEAD

# 比较两个分支差异
git diff <分支1> <分支2>

# 比较两个提交差异
git diff <commit1> <commit2>

# 查看某个文件的差异
git diff -- <文件名>

# 统计差异行数
git diff --stat
```

### 查看指定提交
```bash
# 查看某次提交详情
git show <commit-hash>

# 查看某次提交的某个文件
git show <commit-hash>:<文件名>
```

---

## 十一、储藏与暂存

### 储藏修改
```bash
# 储藏当前修改（工作区和暂存区）
git stash

# 储藏并添加说明
git stash save "说明文字"

# 储藏包含未跟踪文件
git stash -u

# 储藏所有文件（包括忽略的）
git stash -a
```

### 查看与恢复储藏
```bash
# 查看储藏列表
git stash list

# 恢复最近的储藏（不删除储藏记录）
git stash apply

# 恢复指定储藏
git stash apply stash@{0}

# 恢复并删除储藏记录
git stash pop

# 删除最近的储藏
git stash drop

# 删除指定储藏
git stash drop stash@{0}

# 清空所有储藏
git stash clear
```

### 从储藏创建分支
```bash
# 基于储藏创建新分支
git stash branch <新分支名> stash@{0}
```

---

## 十二、合并与变基

### 合并分支
```bash
# 将指定分支合并到当前分支
git merge <分支名>

# 不自动提交（方便检查合并结果）
git merge --no-commit <分支名>

# 快进合并（如果可能）
git merge --ff-only <分支名>

# 禁止快进，生成合并提交
git merge --no-ff <分支名>

# 中止合并（冲突时）
git merge --abort
```

### 变基（Rebase）
```bash
# 将当前分支变基到目标分支
git rebase <目标分支>

# 交互式变基（可修改提交历史）
git rebase -i HEAD~3

# 继续变基（解决冲突后）
git rebase --continue

# 跳过当前提交
git rebase --skip

# 中止变基
git rebase --abort
```

> ⚠️ **注意**：不要对已经推送到远程的提交进行 rebase，会改写历史导致他人同步困难。

### 解决冲突
```bash
# 查看冲突文件
git status

# 标记冲突已解决
git add <文件名>

# 继续合并/变基
git commit  # merge 场景
git rebase --continue  # rebase 场景
```

---

## 十三、常见问题解决

### 1. GitHub 连接超时（443端口）
**症状**：`Failed to connect to github.com port 443: Connection timed out`

**方案A：使用镜像代理**
```bash
# 修改远程地址为镜像
git remote set-url origin https://mirror.ghproxy.com/https://github.com/用户名/仓库名.git
```

**方案B：配置代理**
```bash
# HTTP 代理
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890

# SOCKS5 代理
git config --global http.proxy socks5://127.0.0.1:7890

# 取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```

**方案C：切换 SSH**
```bash
# 生成 SSH Key
ssh-keygen -t rsa -b 4096 -C "你的邮箱"

# 查看公钥并添加到 GitHub
cat ~/.ssh/id_rsa.pub

# 切换远程地址为 SSH
git remote set-url origin git@github.com:用户名/仓库名.git

# 测试连接
ssh -T git@github.com
```

### 2. 强制覆盖本地（丢弃所有本地修改）
```bash
git fetch origin
git reset --hard origin/main  # 或 origin/master
git clean -fd
```

### 3. 撤销已经 push 的提交
```bash
# 方式1：创建新的撤销提交（推荐，安全）
git revert <commit-hash>
git push

# 方式2：强制回滚（⚠️ 危险，会丢失历史，仅个人分支使用）
git reset --hard HEAD~1
git push -f
```

### 4. 不小心 commit 了大文件/敏感文件
```bash
# 从历史中彻底删除文件（⚠️ 会改写所有历史）
git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch 文件名' \
  --prune-empty --tag-name-filter cat -- --all

# 强制推送
git push --force
```

### 5. 合并冲突太多想放弃
```bash
# 中止合并
git merge --abort

# 中止变基
git rebase --abort
```

### 6. 恢复误删的分支/提交
```bash
# 查看操作历史
git reflog

# 找到对应的 commit-hash，恢复
git checkout <commit-hash>
# 或基于它创建新分支
git checkout -b <新分支名> <commit-hash>
```

### 7. 换行符问题（Windows/Linux 混用）
```bash
# 查看当前设置
git config core.autocrlf

# Windows 推荐：提交时转 LF，检出时转 CRLF
git config --global core.autocrlf true

# Mac/Linux 推荐：提交时转 LF，检出保持不变
git config --global core.autocrlf input

# 关闭自动转换
git config --global core.autocrlf false
```

---

## 十四、工作流示例

### 日常开发流程
```bash
# 1. 拉取最新代码
git pull

# 2. 创建开发分支
git checkout -b feature/新功能

# 3. 编写代码...

# 4. 查看修改
git status
git diff

# 5. 提交代码
git add .
git commit -m "feat: 完成新功能开发"

# 6. 推送到远程
git push -u origin feature/新功能
```

### 多人协作流程
```bash
# 1. Fork 项目后克隆到本地
git clone <你的fork地址>

# 2. 添加原仓库为上游
git remote add upstream <原仓库地址>

# 3. 同步上游最新代码
git fetch upstream
git checkout main
git merge upstream/main

# 4. 创建功能分支开发
git checkout -b feature/xxx

# 5. 开发完成后提交 PR
```

### 紧急修复（Hotfix）流程
```bash
# 1. 从主分支创建修复分支
git checkout main
git checkout -b hotfix/修复问题

# 2. 修复并提交
git add .
git commit -m "fix: 紧急修复xxx问题"

# 3. 合并回主分支
git checkout main
git merge hotfix/修复问题

# 4. 打标签发布
git tag -a v1.0.1 -m "发布v1.0.1修复版"
git push --tags

# 5. 删除修复分支
git branch -d hotfix/修复问题
```

---

## 📎 相关链接

- [[#五、分支管理|分支管理详解]]
- [[#八、撤销与回滚|撤销操作大全]]
- [[#十三、常见问题解决|常见问题排查]]
- [[#十四、工作流示例|工作流参考]]

---

> 💡 **提示**：在 Obsidian 中使用时，可将本文件放在知识库中，通过 `[[Git命令手册]]` 双链接引用。各章节标题也支持通过 `[[Git命令手册#章节名]]` 进行锚点跳转。
