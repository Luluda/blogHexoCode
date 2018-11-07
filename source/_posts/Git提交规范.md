---
title: Git提交规范
date: 2018-11-07 20:14:54
tags: [git]
categories: git
---
Git每次提交代码，都要写Commit message（提交说明），一般来说commit message 应该清晰明了说明本次提交的目的。提交说明有多种规范，这里介绍Angular规范，并使用validate-commit-msg加ghooks在提交时强制校验message格式。格式与工程配置方式如下：

## 格式
```text
<type>(<scope>): <subject>
```
`type`(必需)、`scope`（可选）、`subject`（必需）
### **type**
type用于说明commit类别，一般使用下面7个标识（完整的为validate-commit-msg中配置）
```text
feat：新功能（feature）
fix：修补bug
docs：文档（documentation）
style： 格式（不影响代码运行的变动）
refactor：重构（即不是新增功能，也不是修改bug的代码变动）
test：增加测试
chore：构建过程或辅助工具的变动
```
### **scope**
用于说明 commit 影响的范围，例如组件、页面模块等
### **subject**
commit 目的的简短描述，不超过50个字符
```doc
动词开头，若英文以第一人称现在时。例如：feat(库存): 增库存 或 feat(inventory): add inventory
结尾不加句号
```
## **validate-commit-msg**+**ghooks**校验
[validate-commit-msg](https://github.com/conventional-changelog-archived-repos/validate-commit-msg)用于检查commit message是否符合格式。
[ghooks](https://www.npmjs.com/package/ghooks)为git 操作钩子，可以在git操作前做一些处理，这里用**commit-msg**钩子校验git commit是否合法。
安装配置：
### 安装validate-commit-msg
```
npm install --save-dev validate-commit-msg
```
### 配置validate-commit-msg
有两种配置方式：新建.vcmrc或在package.json中配置
 - 新建.vcmrc：在根目录下新建.vcmrc，默认配置如下，具体配置见文档
```json
{
  "types": ["feat", "fix", "docs", "style", "refactor", "perf", "test", "build", "ci", "chore", "revert"],
  "scope": {
    "required": false,
    "allowed": ["*"],
    "validate": false,
    "multiple": false
  },
  "warnOnFail": false,
  "maxSubjectLength": 100,
  "subjectPattern": ".+",
  "subjectPatternErrorMsg": "subject does not match subject pattern!",
  "helpMessage": "",
  "autoFix": false
}
```
 - 配置package.json: 文件中新增
```json
{
  "config": {
    "validate-commit-msg": {
      /* your config here */
    }
  }
}
```
 
### 安装ghooks
```
npm install ghooks --save-dev
```
### 配置ghooks
package.json新增：
```json
"config": {
    "ghooks": {
      "commit-msg": "validate-commit-msg"
    }
  }
```
配置好后，每次commit 之前就会进行自动格式校验，不符合规范直接报错
![](/images/commitVerify.jpg)
## 为什么要规范提交？
### 提供更多的历史信息，方便快速浏览。
```
git log <last tag> HEAD --pretty=format:%s
```
命令可以快速列出上次发布后的变动，规范commit可以清楚地知道每次代码修改
规范提交：
![](/images/commitLog.png)
### 可以过滤某些commit（比如文档改动），便于快速查找信息。
commit规范，可以用下面的命令仅仅显示本次发布新增加的功能
```
git log <last release> HEAD --grep feature
```
### 可以直接从commit生成Change log
脚本自动生成change log 见[Commit message 和 Change log 编写指南](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)

参考文档：
[Commit message 和 Change log 编写指南](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)
[validate-commit-msg](https://github.com/conventional-changelog-archived-repos/validate-commit-msg)
[ghooks](https://www.npmjs.com/package/ghooks)