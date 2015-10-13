---
layout: post
title: "基于 Octopress & Github Pages 搭建博客（一）"
date: 2015-10-13 13:13 +0800
comments: true
categories: Octopress
---

<p class="info">
一直以来想有个属于自己的博客空间，或许是出于一种归属感吧。就这样知道了 WordPress、Jekyll、Hexo 和 Octopress。一番对比后选择了 Octopress，相信追随大神的脚步应该不会错。Octopress 接触有一个多星期了，幸运的是经过一番折腾算是弄出点了模样。这里总结下基于 Octopress 及 Github搭建博客的过程及自己中间遇到的一些问题的解决办法。技术上不一定完全精确，若有大神围观望指正：)
<br><br>
使用的是 Mac OS X 系统，不一定适用于 Windows 的童鞋。（勿拍砖...）
<br><br>
这是最终的实现效果：<a href="http://jonyfang.github.io/" target="_blank"> I'm Jony</a>
</p>


* list element with functor item
{:toc}


## 1. Octopress 与 Jekyll & Github Pages 的关系

Octopress 是基于 Jekyll 的静态博客框架。

GitHub Pages 这里用于显示托管在 GitHub 上的静态网页，是 GitHub 提供的一项服务。

总的来说也就是我们使用基于 Jekyll 的 Octopress 生成本地的静态网站，然后将静态的网站托管到 Github 为我们提供的 Github Pages 服务上。最后访问 博客地址 就可以显示我们的个人博客网站了。


## 2. 准备工作
<!-- more -->

### 2-1. 安装 Git

前往 Git 官网 [点击这里](http://git-scm.com/) ，按下图提示下载安装（一般 Mac OS X自带 Git 环境，终端执行 git -v 可查看 Git 版本）。

![git_downloads](http://jonyfang.github.io/images/octopress/git_downloads.png)

### 2-2. 安装 Ruby

这是 [Ruby 官网](https://www.ruby-lang.org/en/)，这里就不详细介绍 Ruby 啦，总之 Ruby 很 NB，后面静下心研究下。好吧，回到 Ruby 的安装。

打开终端，执行如下命令，安装 RVM，同时也会安装最新的 Ruby：

{% codeblock lang:ruby %}
  $ curl -L https://get.rvm.io | bash -s stable --ruby
{% endcodeblock %}

安装完，执行如下命令，查看 Ruby 版本 (-v = --version)

{% codeblock lang:ruby %}
  $ ruby -v
{% endcodeblock %}

如果你的 Ruby 版本不低于 1.9.3，可直接跳转到 安装 RubyGems。否则需要执行如下命令：

{% codeblock lang:ruby %}
  $ rvm install 2.0.0
  $ rvm use 2.0.0
{% endcodeblock %}

安装 RubyGems：

{% codeblock lang:ruby %}
  $ rvm rubygems latest
{% endcodeblock %}

现在我们再执行命令 ruby -v 查看 Ruby版本，会看到现在已经是 2.0.0 了。

呼，准备工作搞定！

## 3. 本地安装 Octopress

前面做了那么多准备，主角总算要上场了。

首先，将 Octopress 的项目 clone 到本地，终端执行如下命令：

{% codeblock lang:ruby %}
  $ git clone git://github.com/imathis/octopress.git octopress
{% endcodeblock %}

进入 octopress 目录，安装 Octopress 所需要的依赖库（dependencies）：

{% codeblock lang:ruby %}
  $ cd octopress

  # 安装过程中可能会遇到权限问题，我们需要在命令前面加上 sudo 再执行，并输入登录密码。
  # sudo 全称：super user do，也就是以 root 用户身份来执行

  $ sudo gem install bundler
  $ bundle install
{% endcodeblock %}

<p class="warning">
这里在不翻墙的情况下，可能会遇到一个问题：sudo gem install bundler 执行后，一直没有响应。这是由于国内网络原因（你懂的），导致<a href="http://rubygems.org/" target="_blank"> rubygems.org </a>存放在 Amazon S3 上面的资源文件间歇性连接失败。所以你会遇到 gem install rack 或 bundle install 的时候半天没有响应的情况。
</p>

幸运的是国内某大神帮我们解决了这一心头大患，我们可以用淘宝的Ruby镜像来替换原来的镜像。只需终端执行下面的命令即可：

{% codeblock lang:ruby %}
  $ gem sources -a https://ruby.taobao.org/ -r https://rubygems.org/
  
  # 下一命令查看切换后结果
  $ gem sources -l
{% endcodeblock %}

然后会看到这样的输出：

{% codeblock lang:ruby %}
  *** CURRENT SOURCES ***

  https://ruby.taobao.org
{% endcodeblock %}

这就说明我们切换到淘宝的 Ruby 镜像了，再次安装 Octopress 所需要的依赖库就会发现现在可以了。

最后安装下默认主题：

{% codeblock lang:ruby %}
  # rake 全称：ruby make
  $ rake install
{% endcodeblock %}


> 上面提到的 Ruby 镜像问题，还有另外两种解决方法:

（1）比较原始的方法——手动更改：打开 octopress 文件夹 -> 打开 Gemfile 文件 -> 将 source "https://rubygems.org" 改为 source "https://ruby.taobao.org" 就可以了。

（2）相对方便点，因为我们使用的是 Gemfile，所以我们可以用 Bundler 的 [Gem 源代码镜像命令](http://bundler.io/v1.5/bundle_config.html#gem-source-mirrors)，这样我们就不用改 Gemfile 的 source 了。

{% codeblock lang:ruby %}
  $ cd octopress
  $ bundle config mirror.https://rubygems.org https://ruby.taobao.org
{% endcodeblock %}


## 4. 预览效果

好，经过上面的功夫，我们已经在本地搭建了一个简易版的 Octopress 博客。

我们来看看效果吧。在终端执行命令：

{% codeblock lang:ruby %}
  $ sudo rake preview
{% endcodeblock %}

打开浏览器，输入 `http://localhost:4000/`，就可以看到效果了。虽然比较简陋，但让人挺高兴的，你觉得呢？

![octopress 本地显示效果](http://jonyfang.github.io/images/octopress/octo_newpage.png)


到这里，我们算是初步结束了本地安装过程，下一篇我们会把本地的 Octopress 部署到 Github，那么下篇再见喽～


> 本篇参考：
>	
> [Octopress Setup](http://octopress.org/docs/setup/)
>	
> [Gem Source Mirrors](http://bundler.io/v1.5/bundle_config.html#gem-source-mirrors)