---
layout:     post
title:      "Hello Blog - 2020"
subtitle:   "记录搭建这个博客的初衷和过程"
date:       2020-03-11 19:47:00
author:     "Lindada"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 博客
---

>说来惭愧，作为一个常年依靠各种博客教程才能勉强活得下去的coder，却很少写博客。主要是觉得要工工整整地把所有知识点记录下来，需要花费大量的时间。所以总是遇到一个问题就查一个问题，解决完就扔到脑后了，最多在记事本上简单记两下。Emmm，随着时间流逝，那些仅存的“足迹”也被遗忘在各个角落，感觉自己什么也没有留下来。幡然醒悟，做人还是体系化一点好，养成良好的习惯很重要...最近也开始在网上记录一些东西了，主要是在CSDN和有道云笔记上，但CSDN的界面实在太丑，广告又多；有道云笔记对内容的公开不是特别方便，所以决定搭建一个自己的博客，看起来顺心一点。本模板使用github pages 和 jekyll搭建而成，具体的搭建过程写在下面了，希望对想使用github pages搭建个人博客的人提供一点帮助。

### 前言
本文参照了 [Hux](https://github.com/Huxpro/huxpro.github.io) 的博客模板，使用是github pages 和 jekyll搭建而成。github pages一般多用于托管个人的静态网站，可以理解为github提供的免费的搭建个人主页的资源，选择github pages主要原因如下：
1. 免费提供1G以内的静态资源托管，利用它我们可以搭建个人主页或者博客，而省去了购买服务器的钱，不用白不用。
2. github是程序员经常使用的平台，将博客挂在上面方便我们的管理。
3. 比较有bigger

github pages只能放置静态资源，也就是不能进行后台的逻辑交互，所以我们需要把我们的博客转成静态资源，再放到上面去。怎么转？自己手撸html代码吗？当然不，jekyll就是帮我们来做这件事情的。它能将markdown文档转换成静态的资源文件，让我们只专注于写博客的过程。因为缺少后台的交互，所以不能评论和留言，不能在线编辑文章，但其实我们也不一定需要这两件事情。总的来说，要完成这个博客的搭建和文章写作，你应：
1. 安装jekyll
2. 拥有github账户
3. 安装Git环境
4. 会用markdown

之后的教程将按照如下顺序展开：
- 安装环境（Windows）
	-	安装Ruby+Devkit
	-	安装RubyGem
	-	安装jekyll
	-	安装jekyll-pageinate
	-	安装bundler
- github设置
- 模板修改
- 自定义域名

### 安装环境
jekyll在Windows、Mac和Linux下都能安装，而且除了Windows其他两个还很简单。没办法。。用的是Windows，麻烦就麻烦了。

#### 1.安装Ruby环境
首先在[官网下载](https://rubyinstaller.org/downloads/)Ruby+Devkit安装包,我下载的是推荐的版本。安装过程记得勾选添加到PATH，免去设置环境变量的操作。
![Ruby](/img/in-post/blogbuild/ruby.png "Ruby"){: width="400"}


最后一步勾选这个：
![RubyFinish](/img/in-post/blogbuild/rubyfinish.png "RubyFinish"){: width="400"}

在弹出的框中选择3：
![RubyInstaller](/img/in-post/blogbuild/rubyinstaller.png "RubyInstaller")

#### 2.安装RubyGem
Gem是Ruby下的一个下载工具，类似于Python的pip。到[这里](https://rubygems.org/pages/download)下载安装，选择压缩包zip下载，这里就不贴图了。下载完之后将zip文件解压到任意文件夹，并进入该文件夹下，在当前路径打开cmd命令行，输入：
`ruby setup.rb`

#### 3.安装jekyll
在cmd中输入：
`gem install jekyll`

#### 4.安装jekyll
在cmd中输入：
`gem install jekyll-paginate`

#### 5.安装bundler
在cmd中输入：
`gem install bundler`

#### 6.安装时遇到的问题
在进行第3步的时候，发现输入命令后等待时间很久都没反应，考虑可能是服务器在国外，下载网速太慢导致，这里我通过更换源解决。

先查看当前的源：
`gem sources -l`  如无意外，默认的源是 https://rubygems.org/，我们将其删除：
```gem sources -r https://rubygems.org/```

再添加国内可用的源:
`gem sources --add https://gems.ruby-china.com`

再执行3、4、5步，发现飞一般的感觉~

#### 7.验证环境
如果上述步骤完成，jekyll环境应该是搭好了，cmd中使用指令`jekyll -v`查看当前版本，我的版本是4.0.0

接下来我们可以使用jekyll生成一个默认的博客模板，看一下是否能正常运行。先进入你准备存放博客数据的文件夹，然后在cmd中输入：
`jekyll add blogtest`

上述命令在我电脑中运行实测要运行非常久，不知是何原因，生成的文件夹只有几kb大小而已，一定要等它运行完哦，不然生成的博客模板不完整，运行不了哦。反正经过半个小时的漫长等待，它终于跑完了。。。
跑完之后在你选择的路径应该会出现blogtest文件夹，里面的数据内容为：
![blogtest](/img/in-post/blogbuild/blogtest.png "blogtest")

回到命令行，输入以下指令：
```
cd blogtest
jekyll serve
```

这样，jekyll就把我们刚刚建立的模板运行起来啦，可以在浏览器访问 127.0.0.1:4000 访问（下面的页面我做了简单修改，你看到的可能不是这个样子的）：
![testview](/img/in-post/blogbuild/testview.png "testview")

说明环境配置莫得问题！接下来就是要替换其他的模板啦！


### github设置
我们首先需要将模板项目fork到自己的github上，这里推荐fork 原作者[Hux Github](https://github.com/Huxpro/huxpro.github.io) 的，不要fork我的。
打开链接，右上角fork，搞定。

然后回到自己的主页，打开刚刚fork的项目,点“Setting”页，将项目重命名为 \*\*\*.github.io ，其中\*\*\*的部分一定要跟你的用户名一模一样，这样github才能识别到这是你的github pages项目。
![rename](/img/in-post/blogbuild/rename.png "rename")

现在可以把项目clone下来了，这里需要一些Git的知识，不了解的话可以参考以下教程：
[最新GitHub新手使用教程(Windows Git从安装到使用)——详细图解](https://blog.csdn.net/qq_41782425/article/details/85183250)

### 模板修改
模板下载下来之后，可以按照上述运行测试环境的步骤，运行下载下来的模板：进入该文件夹，命令行输入：
`jekyll serve` 然后到127.0.0.1:4000 观察是否看到页面。

下面我们修改模板，让它变成我们的博客。具体配置可参考 [Hux的中文教程](https://github.com/Huxpro/huxpro.github.io/blob/master/README.zh.md)，我们下面简单说一下我在配置中认为比较重要的几个文件：

文件夹内的文件跟刚刚的blogtest结构差不多，最重要的配置文件在根目录下的_config.yml ，这个文件设置了博客的主要信息，比如：简介、友情链接、是否使用标签、统计接口等。

_posts 文件夹存放了你的markdown文档，你只需要把markdown文档放在这里，jekyll会自动帮你转成静态页面，非常方便。哦对了，使用`jekyll`命令之后，你实时改动文档，它会自动更新哦，你只需要刷新一下页面就可以看到你写的内容了，这样可以实现一边写一边看，非常方便。

_includes文件夹下的about文件夹存放了两个markdown文件，分别是博客的about界面的中/英文内容，修改这两个文档即可修改博客的about页面。

根目录下的about.html和archive.html分别是about和archive页面的配置内容，打开可修改，修改头部分即可。如果你想改URL，可以直接将文件名改成你想要的名字，比如我将about改成了resume，然后访问lindada.com.cn/resume就能看到我的简历啦（自定义域名等下再讲）。

写博客的时候，在markdown文件前加上一段YAML头，这样jekyll才知道你这篇是博客内容，才会帮你自动转成静态页面。本文的YAML头如下，很简单的信息，看看就懂。

```
---
layout:     post
title:      "Hello Blog - 2020"
subtitle:   "记录搭建这个博客的初衷和过程"
date:       2020-03-11 19:47:00
author:     "Lindada"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 博客
---
```

差不多也就这样吧，修改完看看127.0.0.1:4000有没有正常显示，正常显示就没毛病，可以提交到github上了。
```
git add -A
git commit -m "write something"
git push -u origin master
```


提交后等几分钟，应该就能访问了。打开\*\*\*.github.io （\*\*\*是你的用户名），应该能正常显示页面。


### 自定义域名
显然用这么长的域名非常没有逼格，我们可以自定义我们的域名，首先你得拥有一个域名，腾讯云、阿里云或者其他主机厂商都能购买。
这里要执行几步操作：

#### 1. 查看博客主页的ip地址
这个很简单，ping一下就能知道，在cmd中输入命令：
`ping \*\*\*.github.io`  （\*\*\*是你的用户名），将返回的ip地址复制下来


#### 2.到域名管理后台添加解析
在你购买的主机厂商那里为你的域名添加解析，解析的地址就填刚刚获取的ip。不懂解析的可以百度一下

#### 3.github pages中设置域名
解析是添加了，但还需要在github pages后台设置你的域名，否则直接访问发现打不开。在网页中打开你的github项目，点到Setting（跟刚刚的地方一样），往下拉，会看到一个Github Pages设置面板，在这里填上你的域名即可
![GithubPages](/img/in-post/blogbuild/githubpages.png "GithubPages")

#### 4.修改CNAME文件
这个修改了应该是你访问\*\*\*.github.io的时候会为你跳转到你自定义的域名，嗯，应该是这样。
在项目根目录下有一个CNAME文件，打开，修改内容为你的域名，就填个域名就行了，保存关闭，push上去，完事。


不出意外的话，应该可以访问你的域名来查看你的博客了。

### More
除了上面的操作，你还可以
- 添加评论功能，支持 多说Duoshuo 评论系统，也支持 Disqus 评论系统。我没有使用
- 添加统计功能，支持 百度统计 和 谷歌统计。我使用了百度统计，可以查看博客的访问报告，嘿嘿。
- 使用CDN加速，Github有时候有点慢，墙内体验不太好。。可以使用CDN加速，腾讯云的每个月还送10G流量呢，可是...需要域名备案，我。。。有时间的小伙伴可以去备案一下

完，嘿嘿~

### 更多阅读
- [Hux的中文教程](https://github.com/Huxpro/huxpro.github.io/blob/master/README.zh.md)
- [Jekyll + Github Pages 博客搭建入门](https://www.jianshu.com/p/9f198d5779e6)
- [你想写自己的博客吗，jekyll](https://zhuanlan.zhihu.com/p/35859756)
- [三步搞定Github Pages自定义域名](https://www.jianshu.com/p/2647e079741f)
- [Github Pages + CDN全站加速](https://www.jianshu.com/p/ccc0cc8c14a0)
- [Markdown 教程](https://www.runoob.com/markdown/md-tutorial.html)




