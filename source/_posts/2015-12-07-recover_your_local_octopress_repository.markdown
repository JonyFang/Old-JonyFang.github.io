---
layout: post
title: "从 Github 恢复 Octopress 到本地"
date: 2015-12-07 20:08 +0800
comments: true
categories: octopress
---

当遇到自己换到一台新的电脑，或本地的 octopress 环境出现了问题，我们都可以从 Github 恢复 Octopress 到本地，在了解了之前的部署操作后这里就很简单了的,进入正题吧。
<br>
(附部署操作：[基于 Octopress & Github Pages 搭建博客（二）](http://jonyfang.com/blog/2015/10/16/starting_blog_with_octopress_2/))
{:.info}
<!-- more -->

Github 远程目录有两个分支，一个**`master`**，一个 **`source`**，首先把 `source` 分支 `clone` 下来:

{% codeblock lang:ruby %}
$ git clone -b source git@github.com:username/username.github.io.git octopress
{% endcodeblock %}


进入 octopress 目录，安装 Octopress 所需要的依赖库（dependencies）：

{% codeblock lang:ruby %}
$ cd octopress
$ gem install bundler
$ bundle install
{% endcodeblock %}


接着执行如下命令，提示后，输入 Github 仓库的 URL （例如：git@github.com:username/username.github.io.git）：

{% codeblock lang:ruby %}
$ rake setup_github_pages
{% endcodeblock %}

最后删除生成的 `_deploy`，克隆 `master` 分支到本地：

{% codeblock lang:ruby %}
$ rm -rf _deploy
$ git clone git@github.com:username/username.github.io.git _deploy
{% endcodeblock %}

