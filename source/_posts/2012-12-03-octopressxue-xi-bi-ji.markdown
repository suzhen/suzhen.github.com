---
layout: post
title: "octopress学习笔记"
date: 2012-12-03 01:14
comments: true
categories: octopress 
---

octopress学习笔记(一)

一些常用的命令和知识点

克隆octopress.git到本地
	git clone git://github.com/imathis/octopress.git octopress
进入octopress项目
	cd octopress
此时需要我们来确定是否将ruby的版本切换到rvm中的1.9.3，输入"y",确定。
	bundler install

octopress的大部分操作都是会通过本身的rake任务来完成的。
可以通过查看Rakefile里的内容来查看，可以通过rake list来查看一些rake命令。

octopress会把blog的主题放在一个.themes的目录下。
我们的第一步就是要加载这个主题
	rake install 
默认加载的主题是classic,加载的过程是把.themes的内容分别拷贝到./source和./sass两个目
录里。sass里放的这个theme是css,而source里放的是theme的html文件和脚本、图片等。

此时我们就可以预览一下这个新blog了。
	rake preview 
这个命令会在目录下出现在一个public的子目录，里面就是程序生成的静态页面。
在浏览器里输入0.0.0.0:4000，就可以看到样子了。是不是很简单。

但是页面里显示得都是些原始的信息，需要我们去设置成自己的信息。就在目录下有一个
_config.yml中可以配置。这个文件里有三部分，一是基本配置，配置
url,title,subtitle,author等，二是配置Jekyll,这部分是设置一些静态文件的目录，路由规则
等。第三部分是一些第三方的功能的设置，比如twitter,googleplus,disqus等。配置好了这些
，再重新rake preview 一下就可以看到包含自己信息的blog了。

我们开始写博客，也是用rake来创建。
	rake new_post[博客标题]
这时候会在source目录下的_post目录下产生一个年月日和文章名称组成的****-**-**-
**.markdown文件，我们编辑这个文件就行了。如果想删除这篇博客文章，通过删除这个文件就
可以了。

编辑好了文章，就需要将文章转化成相应的静态文件，放在public目录下了，这样才能访问到。
	rake generate

我们刷新一下0.0.0.0:4000就可以看到新文章了。


最后是我们要将博客发布到github上。就需要设置一下github的地址。
	rake setup_github_pages
此时，输入***.github.com之后，会在目录下创建一个_deploy的目录，并且初始化为一个git仓
库。
当我们已经把markdown转化为public目录下的静态文件后，就可以通过deploy命令把public下的
内容放到_deploy目录下，加入版本库，并上传到关联的github的master分支上。
	rake deploy
一个命令所有事搞定。

我们对博客的修改其实是对source和sass的修改，那么修改后的内容也应该用git管理起来，只
需要
	git add .
	git commit -m "修改内容"
就可以进行博客的管理。在.gitignore中系统已经自动的把_deploy和public这目录排除了。当
然这个也是需要推送到github上source分支的。
	git push origin source

如果我们需要在另一台机器上也可以写博客，那么就需要把***.github.com仓库克隆到本地。
        git clone git@github.com:suzhen/suzhen.github.com.git octopress
查看一下分支
        git branch
会发现有master和source两个分支，通过git checkout source 切换到代码分支就可以开发博客
了。但是我们会发现_deploy这个文件并不存在。rake deploy命令并不能执行。所以我们还需要
将deploy从github上克隆下来。在octopress目录里再git clone 
git@github.com:suzhen/suzhen.github.com.git _deploy 就可以了。这样我们就可以继续使用
rake deploy命令了。
















