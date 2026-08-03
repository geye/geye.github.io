# 07-Dependabot依赖自动更新
[[README.md|返回总目录]]
[[03-GitHub Actions工作流]]

对应截图工作流：Dependabot Updates（运行6次）
## 作用
Dependabot：GitHub官方工具，自动扫描项目package、maven等依赖文件
检测第三方库新版本，**自动创建Pull Request升级依赖版本**

## 配置文件
路径：`.github/dependabot.yml`
可以设置：
- 多久检查一次更新（每日/每周/每月）
- 哪些目录需要扫描
- 更新后自动触发Actions流水线编译验证

## 工作流程
1. Dependabot定时运行 → 发现依赖新版本
2. 自动创建分支，修改版本号
3. 自动提交PR
4. 触发 Dependabot Updates 这条Actions流水线编译测试
5. 人工确认无报错后合并PR，完成依赖升级

## 优势
避免第三方库版本老旧带来安全漏洞，不用手动逐个升级依赖。