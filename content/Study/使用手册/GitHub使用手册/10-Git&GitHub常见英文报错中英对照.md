---
# [[Git&GitHub常见英文报错中英对照]]
## 目录
- [[Git&GitHub常见英文报错中英对照#连接权限类报错|连接权限类报错]]
- [[Git&GitHub常见英文报错中英对照#提交推送Pull相关报错|提交推送Pull相关报错]]
- [[Git&GitHub常见英文报错中英对照#Quartz Actions部署报错|Quartz Actions部署报错]]
- [[Git&GitHub常见英文报错中英对照#基础解决通用思路|基础解决通用思路]]

> 用途：GitHub网页、Git终端、Actions运行日志英文报错快速查询

## 连接权限类报错
|英文报错原文|中文翻译|解决方案简述|
| ---- | ---- | ---- |
|Permission denied (publickey)|公钥拒绝访问|未配置SSH密钥，改用Token/配置SSH|
|fatal: Authentication failed|身份验证失败|账号密码已淘汰，使用Personal Access Token|
|Could not resolve host|无法解析主机|网络问题，检查代理与DNS|

## 提交推送Pull相关报错
|英文报错原文|中文翻译|解决方案简述|
| ---- | ---- | ---- |
|! [rejected] main -> main (non-fast-forward)|推送被拒绝，非快进更新|先执行git pull拉取远端代码合并|
|Your local changes would be overwritten by merge|本地修改将被合并覆盖|先commit或者stash暂存本地改动|
|Merge conflict|合并冲突|手动编辑冲突文件后重新提交|

## Quartz Actions部署报错（4.5.2高频）
|英文报错原文|中文翻译|解决方案简述|
| ---- | ---- | ---- |
|npx: command not found|找不到npx命令|检查node环境是否正常安装|
|Out of memory|内存溢出|扩容Runner，或者精简大量图片资源|
|No matching version found for quartz|石英版本不存在|锁定package.json quartz版本为4.5.2|
|publish_dir does not exist|publish_dir文件夹不存在|确认quartz build正常生成public目录

## 基础解决通用思路
1. 复制完整报错文本搜索关键词
2. 区分：本地Git终端报错 / GitHub网页报错 / Actions运行日志报错
3. Quartz部署类问题优先核对Node版本与quartz版本

[[跳转→[[03-GitHub Actions 工作流]]]]
[[←返回README.md]]