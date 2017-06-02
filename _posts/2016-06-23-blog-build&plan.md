---
layout: post
title: "博客搭建"
description: "第一次个人博客搭建过程记录"
category: 技术分享
tags: [博客搭建]
---
{% include JB/setup %}
# 博客搭建过程记录
---
	
　　至此，博客终于可以正常访问了！通过github pages功能搭建博客其实难度不大，就几个步骤，各种账号申请，然后简单的git使用就完事了。主要的工作量还是在静态页面的设计上，我这里用的是基于jekyll引擎的模板，这里感谢[enml](https://github.com/enml/blog/tree/jekyll-blog)的博客模板，很简洁，一直爱不释手。  
<!--break-->

　　博客的搭建和维护其实就是代码仓库的创建和代码的提交与修改，所以在搭建博客之前git的使用方法一定要熟悉一下，推荐几篇 [git - 简易指南](http://www.bootcss.com/p/git-guide/)、[Git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/)。git使用方法熟悉了，博客搭建就基本没有问题了。github pages分为两种 *User or organization site*、*Project site*。 我理解两者的差别有： 1. 意义不同（废话）。 2. 创建的仓库名格式不同。 user/organization 创建的仓库名格式必须为 **username.github.io**，这也就是你博客访问的域名。Project仓库名格式没有要求，随意。 3. 分支不同。 User/Organization博客代码（博文）提交到master分支；Project主页提交到gh-pages分支，可以自己手动创建，也可以通过github页面上的操作来点击完成。具体github pages使用访问请移步[github pages](https://pages.github.com/)。我创建的是User/Organization类型，按照[github pages](https://pages.github.com/)上面的操作后成功访问域名 *https://henanCaiyao.github.io/*（论一个好的ID是多么重要！！！！ henanCaiyao什么鬼，太老土了，推荐[如何设计好记又唯一的ID](https://www.zhihu.com/question/19613148?q=%E5%A6%82%E4%BD%95%E8%AE%BE%E8%AE%A1%E5%A5%BD%E8%AE%B0%E5%8F%88%E5%94%AF%E4%B8%80%E7%9A%84ID)）。当你看到熟悉的Hello World时表明你已经完成了博客的搭建。但是只显示个Hello World不算个事啊。下面来丰富一下网页。  
　　有两种方式提交你的博客格式：　1. 写好Html文件提交（有很多静态博客页生成程序，比如Hexo） 2. 提交符合Jekyll格式的文件，由github自动生成静态网页。我用的是Jekyll格式。Jekyll格式模板有很多，可以在这里找--> [Jekyll Themes](http://jekyllthemes.org/)，也可以自己手动写。 我是从[enml](https://github.com/enml/blog/tree/jekyll-blog)哪里fork来的，其实也不算fork，因为他的是Project Page，而我的是User page 。所以我是把他的代码复制到我的henanCaiyao.github.io仓库里的。当然复制以后是需要修改一些个人的东西的，比如博主名字、博客内容等。将个人的东西修改完了以后提交就可以看到自己的一些信息了。  
　　博客还有两个比较重要的是评论和访问分析。有的模板带的有这两块代码，但是需要自己给一些配置更新一下，评论现在用的比较多的就是[disqus](https://disqus.com/)，我也用的这个，disqus网站上有教怎么一步一步将disqus添加到网页上。添加了以后的效果就像这样
![disqus](http://7xvn6m.com1.z0.glb.clouddn.com/disqus.png)
如果想在上面评论需要注册disqus账号，当然上面的评论都是存储在disqus上，你博客上的上的评论可以登录你自己disqus账号后查看。而且点击你自己的账号可以显示你所有的评论记录
![disqus](http://7xvn6m.com1.z0.glb.clouddn.com/disqus-2.png)
。评论弄完了现在来弄访问分析插件，你网站访问量多少应该比较关心的事情了，大一点的网站应该有自己作统计的，但是作为连空间都用github代码托管空间冒充的我当然还是用现成的51啦![51啦](http://7xvn6m.com1.z0.glb.clouddn.com/analysics.png)。还有cnzz，百度统计、Google Analytics。。。。 跟评论插件一样的步骤，到51啦官网注册账号然后将获取的嵌入代码放在自己想放的位置上就ok了，当想看访问数据是用自己的账号登录就可以看了
![analysis](http://7xvn6m.com1.z0.glb.clouddn.com/analysis-2.png).  
　　现在基本上都处理完了，就剩最重要的一步： 写博客了~~ 。 这是一场持久战，不能一时兴奋搭了个站，然后搭好以后再也没有访问过。。。。 写博客之前需要了解一些基本知识，第一个就是markdown，[认识与入门 Markdown](http://sspai.com/25137)。还有 Liquid的简单语法，我用的模板引擎是jekyll，Liquid是它支持的标记语言，就类似jsp一样，可以在页面中嵌入一些变量，markdown是用来写博客的标记语言，比html还要简单很多，它的主要目标就是让写者注重内容而不是样式，当然花里胡哨的也不适合博客的，window平台大家都在推荐的是MarkdownPad。挺好用的，我的这篇博文就是在这个上面写的，当然我也是刚入门，还很难看。 写完保存，git add --all git commit -m "" git push origin master。 搞定~~~~
