# 05-Pull Request(PR)详解
[[README.md|返回总目录]]
[[02-仓库(Repository)操作]]

## 名词翻译
Pull Request（简称 PR）中文标准称呼：**合并请求**
开源流程核心：Fork → 修改代码 → Pull Request

### 完整Fork贡献流程
1. Fork：复刻目标仓库到自己账号（页面右上角Fork按钮）
2. 克隆你账号内的仓库到本地，新建分支修改代码
3. 推送分支到自己仓库
4. 点击 Compare & pull request 发起PR
5. 上游仓库管理员代码评审 Review
6. 通过后 Merge pull request：合并代码到主干

## PR页面功能翻译
- base repository：目标仓库（要合并进的仓库）
- head repository：你的仓库、你的分支
- Create pull request：创建合并请求
- Review changes：代码评审
- Add review comment：单行代码添加评论
- Approve：批准合并
- Request changes：要求修改代码
- Merge pull request：合并（三种模式）
  1. Create a merge commit：保留全部提交记录（最常用）
  2. Squash and merge：压缩所有提交为一条记录
  3. Rebase and merge：变基合并

## 冲突
Merge conflict：代码冲突，两个人修改同一处代码，需要手动解决冲突。