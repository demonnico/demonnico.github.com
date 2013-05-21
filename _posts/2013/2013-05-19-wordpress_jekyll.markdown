---

author: demon
comments: true
date: 2013-05-19 20:00:00
layout: post
slug: wordpress_jekyll_exchange
title: 从wordpress到jekyll
wordpress_id: 208
keywords: wordpress jekyll github pages markdown cname
description: wordpress到jekyll
categories:
- 总结
tags:
- wordpress
- jekyll
- github

---

周末花了点时间把之前一些写的东西从wordpress换到了基于jekyll的github pages，因为没有数据库，界面也都是静态生成的，所以访问的速度很快。文章用markdown来写，这样的话每次要新增一篇文章只要新建一个.markdown文件后保存后提交，如果修改文章，在git下面也有更改的记录。还有关键的一点是这个空间是免费的，速度快，感觉很不错。下面简单说一下Mac下的一些安装和迁移步骤。

首先通过wordpress自带的export工具，把之前所有的文章导出为xml。然后借助github上的[exitwp](https://github.com/thomasf/exitwp)工具将xml中文章导出为markdown的文件。注意一下，exitwp是用Python写的，而且需要html2text、PyYAML、Beautiful soup的支持，exitwp的wiki上也有写，不过我在安装的时候碰到个小问题，Beautiful soup虽然安装成功，但是运行exitwp的脚本时import Beautiful soup报错，因为最新版的Beautiful soup 4的import目录有所改变，不过刚看了下exitwp的作者已经修改好了。

如果你想写好文章之后先在自己电脑上预览一下然后post，那么[本地安装一下jekyll](http://jekyllrb.com/docs/installation/)是必须的。然后基于[jekyllbootstrap](http://jekyllbootstrap.com/)进行安装。代码高亮的话我用[Pygments](http://pygments.org/)搞定。

因为之前的评论也是用disqus，所以迁移过来非常方便。但是有一点要注意一下，默认在`_config.yml`里的`permalink`是`/:categories/:year/:month/:day/:title `，但是我之前所有的post都是不带categories的，所以这里应该改为`/:year/:month/:day/:title `，不然之前在其他地方引用到的地址都会变成死链接，当然也找不到disqus上对应的正确评论。然后把disqus上的`short_name`换一下就可以了。

最后cd至你的jekyll站点目录下，启动一下`jekyll serve`，默认在`localhost:4000`就可以访问到了，没问题的话`git push`一下，第一次提交的话可能需要过一会儿，你就可以在你的github page上访问了。如果你已经有自己的域名了，只要在目录下添加一个CNAME文件，然后写上你自己的域名，在域名提供商那边修改解析规则，添加一条`207.97.227.245`到A记录。

最后需要补充一点，默认使用的markdown引擎应该是maruku，所以会碰到有时候markdown-->html的时候出错，类似`REXML could not parse this XML/HTML:`，这里可以修改_config.yml，我用了rdiscount，所以添加`markdown : rdiscount `后重新启动服务，整个世界就清净了~~

推荐一个免费的Markdown编辑神器[Mou](http://mouapp.com/)





