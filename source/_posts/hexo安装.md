---
title: hexo安装
date: 2016-07-19 19:29:53
categories:
- 其他
tags:
- hexo
---


###下载hexo
+ 安装node
	- 查看安装的node版本 npm -v
+ 安装git
+ 安装Hexo
	- sudo npm install -g hexo-cli

###生成本地hexo目录
+ 创建hexo目录
	- cd hexo
+ 初始化hexo
	- hexo init

###启动hexo
+ 生成静态页面
	- hexo generate（hexo g也可以）
+ 启动本地服务
	- hexo server
	- 浏览器输入http://localhost:4000
	
------------------------------------

###github的repository与hexo关联
+ 创建个人github的repository
	- repository名称必须为${userName}.github.io
+ 建立与hexo的连接
	- cd hexo
	- vim _config.yml
	- 修改config.yml中最下方如下：
	
			deploy:
     			type: git
     			repo:https://github.com/${userName}/${userName}.github.io.git
     			branch:master
	- npm install hexo-deployer-git --save
	- hexo deploy
	- 浏览器http://${userName}.github.io/

### 部署
+ 每次布署前，按下列顺序执行
	- hexo clean
	- hexo generate
	- hexo deploy

------------------------------------

###更换头像、作者、主题
+ 修改头像
	- cd hexo/themes/yilia
	- vim _config.yml
	- 找到 #你的头像url avatar: 后接一个URL就行了，头像就修改成功了
+ 修改作者名字
	- cd hexo
	- vim _config.yml
	- 找到 author: 潘柏信，修改成你自己的名字就行了
	- 部署 hexo g
	- 提交 hexo d
+ 修改主题
	- cd hexo/themes
	- git clone ${gitThemeAddress}.git
	- cd hexo
	- vim _config.yml
	- 找到如下文字
	
				# Extensions
				## Plugins: http://hexo.io/plugins/
				## Themes: http://hexo.io/themes/
				theme:yilia
	- 改成theme: yilia，theme:后面接你自己的主题名字就行了,然后分别执行
	- 部署 hexo g
	- 提交 hexo d

------------------------------------

### 常用命令
+ hexo new"postName" #新建文章
+ hexo new page"pageName" #新建页面
+ hexo generate #生成静态页面至public目录
+ hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
+ hexo deploy #将.deploy目录部署到GitHub
+ hexo help # 查看帮助
+ hexo version #查看Hexo的版本

------------------------------------

### 常见错误
+ ERROR Deployer not found: git 或者 ERROR Deployer not found: github
	- 解决方法： npm install hexo-deployer-git --save
+ ERROR Process failed: layout/.DS_Store
	- 进入主题里面layout和_partial目录下，使用删除命令：
	- rm-rf.DS_Store
+ ERROR Plugin load failed: hexo-server
	- 原因：Besides,utilities are separated into a standalone module.hexo.util is not reachable anymore.
	- 解决方法，执行命令：sudo npm install hexo-server
+ 执行命令hexo server，提示：Usage: hexo ....
	- 原因：我认为是没有生成本地服务
	- 解决方法，执行命令：npm install hexo-server--save
			
			提示：hexo-server@0.1.2 node_modules/hexo-server
			....
			表示成功了
	- 这个时候再执行：hexo-server

			得到:
			INFOHexois running at http://0.0.0.0:4000/.PressCtrl+C to stop.
			这个时候再点击http://0.0.0.0:4000，正常情况下应该是最原始的画面，但是我看到的是：
			白板和Cannot GET / 几个字
			
			原因：
			由于2.6以后就更新了，我们需要手动配置些东西，我们需要输入下面三行命令：
			npm install hexo-renderer-ejs--save
			npm install hexo-renderer-stylus--save
			npm install hexo-renderer-marked--save
			
			这个时候再重新生成静态文件，命令：
			hexo generate（或hexo g）
			
			启动本地服务器：
			hexo server（或hexo s）
			再点击网址http://0.0.0.0:4000O
+ No layout: index.html
	- 原因：更换主题后，主题文件不正确。
