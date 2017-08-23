---
title: hexo搭建github博客
date: 2017-08-23 14:51:27
tags: [hexo, github]
categories: 前端
---
{% blockquote 什么是Hexo https://hexo.io/zh-cn/docs/index.html Hexo官方文档 %}
Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。
{% endblockquote %}

## 安装Hexo

### 安装环境

- Node.js
- Git
node.js建议用nvm进行管理，可以根据不同需要切换所需的node版本。
如果已经安装好上述环境，就可以进行hexo安装啦。
```bash
$ npm install -g hexo-cli
```

## 建站

安装Hexo完成后，执行以下命令初始化工程
```bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```
### 站点配置文件

配置文件`_config.yml`中可以配置网站信息
- 更改*site*下的内容配置title、author等基本信息。
- 更改*deploy*下内容当执行命令`hexo deploy`时实现自动部署到github仓库中
```
deploy:
  type: git
  repository: https://github.com/some/some.github.io.git
  branch: master
```
### 安装依赖

package.json中应该有默认的依赖，直接执行命令下载
```
$ npm install
```
### 创建新博文

```bash
hexo new [layout] "postName" #新建博文
```
scaffolds文件夹中为新建文章的模板，默认有draft、page、post三种，新建博文时更改参数layout为模板类型即可。

### 运行服务器

启动服务器，可以在本地4000端口预览网站效果
```
$ hexo server
```
### 生成静态文件

```
$ hexo generate
```
生成html、css、js等静态资源到public文件夹，deploy时将静态资源部署到github中

### 部署到Git

```
$ hexo deploy
```
配置文件设置好git值后即可命令行一键部署！


## 使用主题


本站点使用[NexT](http://theme-next.iissnan.com/theme-settings.html)主题，详细的配置信息戳链接进入官方文档即可看到。

### 主题下载

```
$ cd <hexo目录>
$ git clone https://github.com/iissnan/hexo-theme-next themes/next

```
### 主题应用

修改站点配置文件`_config.yml`
```
theme: next
```


## 搭建GitHub博客
{% blockquote 阮一峰 %}
GitHub Pages功能，允许用户自定义项目首页，用来替代默认的源码列表。所以，github Pages可以被认为是用户编写的、托管在github上的静态网页。
{% endblockquote %}

用[GitHub Pages](https://pages.github.com/)搭建博客只需要以下几步：
- 创建一个仓库，以`username.github.io`命名（username必须与你GitHub的用户名一致）
- 将Hexo站点配置文件的git仓库地址改为username.github.io仓库地址
- 写好博文，`hexo generate`生成静态资源，`hexo deploy`部署到git仓库
- 访问 `https://username.github.io/`

你的专属博客就展现在你面前啦！


参考文档：
[Hexo](https://hexo.io/)
[NexT](http://theme-next.iissnan.com/theme-settings.html)
[Jekyll迁移到Hexo搭建个人博客](https://www.ezlippi.com/blog/2016/02/jekyll-to-hexo.html)
