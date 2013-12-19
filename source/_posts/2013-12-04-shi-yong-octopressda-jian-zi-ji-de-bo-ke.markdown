---
layout: post
title: "使用octopress搭建自己的博客"
date: 2013-12-04 21:48:26 +0800
comments: true
categories: 
---

![octopress&github](/images/github-page-octopress-title.jpg)

感谢[破船之家](http://beyondvincent.com/)，分享了一篇关于使用octopress在Github上搭建博客文章，看了之后也尝试搭建了一个属于自己的博客。由于对于Ruby和git不是很了解，整个过程花费了不少时间。这篇文章主要记录MAC下搭建的过程。

##1. 安装Git

首先在[Github](https://github.com/)上创建一个账号，下载[最新版本的Git](http://git-scm.com/downloads)，并安装。

参见：[Set up Git](https://help.github.com/articles/set-up-git)


##2. 使用RVM安装Ruby1.9.3

使用ruby --version查看当前Ruby版本。

	$ ruby --version
	ruby 2.0.0p247 (2013-06-27 revision 41674)
	
这里有一点要注意，必须是Ruby1.9.3版本才行。我的mac自带的Ruby2.0.0版本，安装octopress总是提示出错，原因我还不清楚。

使用RVM(Ruby Version Manager)来管理Ruby安装。首先安装RVM:

	curl -L https://get.rvm.io | bash -s stable --ruby
	
然后使用RVM安装Ruby1.9.3版本：

	rvm install 1.9.3
	rvm use 1.9.3
	rvm rubygems latest
	
然后再次查看Ruby版本：

	ruby --version
	ruby 1.9.3p484 (2013-11-22 revision 43786)

参见：[Installing Ruby With RVM](http://octopress.org/docs/setup/rvm/)

##3. 安装octopress


	git clone git://github.com/imathis/octopress.git octopress
	cd octopress
	
然后安装依赖，如果没有使用rbenv，第二句可以忽略：

	gem install bundler
	rbenv rehash    # If you use rbenv, rehash to be able to run the bundle command
	bundle install

然后安装默认的主题：

	rake install
	
参见：[Octopress Setup](http://octopress.org/docs/setup/)

##4. 配置octopress

octopress已经尽量简化需要配置的内容，下面列出了可能需要配置的文件：

	_config.yml       # Main config (Jekyll's settings)
    Rakefile          # Configs for deployment
    config.rb         # Compass config
    config.ru         # Rack config
    
主要在_config.yml中有需要配置的内容。Rakefile中的配置几乎都与部署有关，如果不是用rsync的话，不需要修改它。我实际上就没有对它做修改。

_config.yml中url一定要修改，title、subtitle、author和支持的第三方服务可以选择修改。

	url:                # For rewriting urls for RSS, etc
    title:              # Used in the header and title tags
    subtitle:           # A description used in the header
    author:             # Your name, for RSS, Copyright, Metadata
    simple_search:      # Search engine for simple site search
    description:        # A default meta description for your site
    date_format:        # Format dates using Ruby's date strftime syntax
    subscribe_rss:      # Url for your blog's feed, defauts to /atom.xml
    subscribe_email:    # Url to subscribe by email (service required)
    category_feeds:     # Enable per category RSS feeds (defaults to false in 2.1)
    email:              # Email address for the RSS feed if you want it.

建议把twitter相关的信息删除，否则回加载很慢；/source/_includes/custom/head.html 把google的自定义字体去掉。
<p style="color:red">注意，分号后面必须有一个空格分隔，否则会出错。</p>

参见：[Configuring Octopress](http://octopress.org/docs/configuring/)

##5. 将博客部署到Github上

在Github上创建一个仓库，命名为：username.github.io或organization.github.io (username为你的用户名)

然后终端输入：

	$ rake setup_github_pages

会让你输入仓库路径，安装提示输入即可。

然后：

	rake generate
	rake deploy
	
这样就会生成你的博客，拷贝在_deploy/目录下。

这里要注意的是，如果rake deploy执行提示出错

	! [rejected]        master -> master (non-fast-forward)

由于master分支在_deploy/目录内，所以：

	cd octopress/_deploy
	git pull origin master
	cd ..
	rake deploy


将博客的source提交到gitub上：

	git add .
	git commit -m 'your message'
	git push origin source
	
参见：[Deploying to Github Pages](http://octopress.org/docs/deploying/github/)


##6. 开始写博客


	rake new_post["title"]

创建一个名称为title的博客，可以在source/_posts下找到按照Jekyll的命名规范新生成的文件，打开可以看到：

	---
	layout: post
	title: "title"
	date: 2011-07-03 5:59
	comments: true
	external-url:
	categories:
	---
	
生成并预览：

	rake generate   # Generates posts and pages into the public directory
	rake watch      # Watches source/ and sass/ for changes and regenerates
	rake preview    # Watches, and mounts a webserver at http://localhost:4000
	

可以本地打开http://localhost:4000 ，预览生成的博客。

最终，部署到Github上：

	$ git add .
	$ git commit -am "Some comment here." 
	$ git push origin source
	$ rake deploy
	
参见：[Blogging Basics](http://octopress.org/docs/blogging/)

##7. 小结

至此，使用octopress搭建的博客就已经完成了，之后可以选择新的主题,做一些个性化的配置。另外推荐一本学习git的好书：[Pro Git](http://www.ruanfei.com/doc/progit/)。