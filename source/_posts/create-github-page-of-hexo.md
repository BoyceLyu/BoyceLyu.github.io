title: "Hexo 创建GitHub Pages Blog"
date: 2015-05-14 21:16:55
tags: 关于本blog
description: 本篇教程介绍怎么在github创建自己免费的静态blog.
---
本篇教程介绍怎么在github创建自己免费的静态blog.

相关:nodejs,github
	
#创建Gighub Pages
创建github pages,详情请看这里[Gighub Pages](https://pages.github.com/),这里是官方教程不需要英文功底,照着图做就可以了.这里强调一下创建的repository名字必须是 `username.github.io` ,问为啥?恩~~就是这么强制规定的.
	
官方推荐是用[Jekyll](http://jekyllrb.com/)生成静态页面,由于最早查询相关资料时看到了hexo所以就这么配置上了,以后有机会可以试试Jekyll.好,按照官方文档配置出来之后在浏览器输入`username.github.io`就可以看到你的主页了.是不很开心~~

#使用hexo创建静态页

[hexo](http://hexo.io)是台湾的一个作者开发,基于nodejs的静态blog生成器.官方介绍,Hexo是一个简单、快速、强大的博客发布工具，支持Markdown格式。
官网里详细介绍了怎么配置hexo,本篇bolg就是基于官方文档配置完成.博主目前使用版本:hexo 3.0 ,以后版本不一定试用,如遇到问题请查看[官方文档](http://hexo.io/zh-cn/docs/index.html).
##运行环境

在安装前请确定系统中安装了以下程序:
*	[nodejs](https://nodejs.org/)
*	[git](http://git-scm.com/)

##安装hexo
git命令行:

{% codeblock %}
	$ npm install -g hexo-cli
{% endcodeblock %}

##建站
`<folder>` 为你要创建文件夹名称
{% codeblock %}
	$ hexo init <folder>
	$ cd <folder>
	$ npm install
{% endcodeblock %}

完成之后可以看见目录下的文件
{% codeblock %}
	.
	├── _config.yml
	├── package.json
	├── scaffolds
	├── scripts
	├── source
	|   ├── _drafts
	|   └── _posts
	└── themes
{% endcodeblock %}

###scaffolds
模板文件夹.当创建文章时,Hexo会根据scaffold来创建文件夹.

###scripts
脚本 文件夹。脚本是扩展 Hexo 最简易的方式，在此文件夹内的 JavaScript 文件会被自动执行。

###source
资源文件夹是存放用户资源的地方。除 _posts 文件夹之外，开头命名为 _ (下划线)的文件 / 文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 public 文件夹，而其他文件会被拷贝过去。

###themes
主题 文件夹。Hexo 会根据主题来生成静态页面。

##使用

###创建一篇新文章

{% codeblock %}
	$ hexo new [layout] <title>
	或
	$ hexo n [layout] <title>
{% endcodeblock %}

可以指定 `[layout]` 页面模板,不填默认为post,可以通过修改`_config.yml` 中的 `default_layout` 参数来指定默认布局。

使用上面指令后就可以看到在 /source/_[layout]/目录下查看到刚创建的.md文件.

Hexo发表文章使用的是[Markdown](http://zh.wikipedia.org/zh/Markdown)语法,具体语法规则请看这里有篇中文介绍,[Markdown 语法说明 (简体中文版)](http://wowubuntu.com/markdown/).

###生成
使用 Hexo 生成静态文件快速而且简单。
{% codeblock %}
	$ hexo generate
{% endcodeblock %}

还可以家参数监控文件使用
{% codeblock %}
	$ hexo generate --watch
	或 简写
	$ hexo g --w
{% endcodeblock %}

生成文件后就可以看见源目录下多出了`publis`文件夹,可以通过修改`_config.yml` 中的 `public_dir` 参数来指定默认文件夹。

#预览功能

hexo可以把项目发布到服务器上提供预览功能,Hexo 3.0 把服务器独立成了个别模块，您必须先安装 hexo-server 才能使用。
{% codeblock %}
	$ npm install hexo-server --save
{% endcodeblock %}

装完成后，输入以下命令以启动服务器，您的网站会在 http://localhost:4000 下启动。在服务器启动期间，Hexo 会监视文件变动并自动更新，您无须重启服务器。
{% codeblock %}
	$ hexo server
	或简写
	$ hexo s
{% endcodeblock %}

如果您想要更改端口，或是在执行时遇到了 EADDRINUSE 错误，可以在执行时使用 -p 选项指定其他端口，如下：
{% codeblock %}
	$ hexo server -p 5000
	或简写
	$ hexo s -p 5000
{% endcodeblock %}

>小技巧:一般我写文章会开两个命令行窗口 一个用于编译成静态文件并监控 `$ hexo g --w`,另一个`$ hexo s`部署到nodejs服务上.

#部署
最后说下部署功能

{% codeblock %}
	$ hexo deploy
{% endcodeblock %}

在开始之前，您必须先在 `_config.yml` 中修改参数，一个正确的部署配置中至少要有 type 参数，例如：

{% codeblock %}
	deploy:
  type: git
  repo: git@github.com:BoyceLyu/BoyceLyu.github.io.git
  branch: master
{% endcodeblock %}

  
使用Git部署需要安装[hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git).

{% codeblock %}
	$ npm install hexo-deployer-git --save
{% endcodeblock %}


至此,hexo基本使用介绍到这里.

