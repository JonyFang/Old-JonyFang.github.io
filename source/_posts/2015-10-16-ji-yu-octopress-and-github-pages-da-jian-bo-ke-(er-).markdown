---
layout: post
title: "基于 Octopress &amp; Github Pages 搭建博客（二）"
date: 2015-10-16 02:37:44 +0800
comments: true
categories: octopress
---
* list element with functor item
{:toc}


## 1. 前言

经过[上一篇](http://jonyfang.github.io/blog/2015/10/13/ji-yu-octopress-he-github-da-jian-bo-ke/)，我们在本地搭出了 Octopress 雏形，这一篇首先我们要将本地的 Octopress 博客部署到 Github Pages ，然后发布一篇博文。部署过程中分析了原理，可能会比较枯燥，但是能让我们更了解 Octopress，所以我依旧坚持写下来了。

废话少说，开工～


## 2. 将 Octopress 部署到 Github Pages

Github 大家应该都有了解过，也是我很喜欢的平台之一，功能真心强大并且可免费使用，这里我们拿来托管我们的博客。

### 1.1 新建 Github repository

注册 Github 账号，新建 [Github repository](https://github.com/new)。`项目名称`（Repository name）命名格式为 `username.github.io` ，`username` 是你的 `Github 用户名`（或 organization name，这里和后面我们先不讨论 origanization）。例如我的用户名是 JonyFang，所以输入 JonyFang.github.io 即可。点击 `Create repository` 创建。

<p class="info">
PS：创建完后不要添加任何内容，另外自己过程中产生了两个疑问

1.为什么用 github.io 而不是 github.com？

2.为什么是 Repository name 一定要按照 username.github.io 填写？
</p>

第一个问题，[这里](https://github.com/blog/1452-new-github-pages-domain-github-io#security-vulnerability)解释了为什么把 `github.com` 改为了 `github.io`，简而言之是为了安全。

第二个问题，和 Github 内部的结构有关，其次后面会通过 URL 截取填写的 `username.github.io` 作为博客域名。这样填写格式与 Github 内部结构的具体联系还需要再研究下。若有大神围观，望指教下：）


## 1.2. 配置 Github Pages

终端执行如下命令：

{% codeblock lang:ruby %}
  $ cd octopress
  $ rake setup_github_pages
{% endcodeblock %}

该命令会要求我们输入 Github 仓库的 URL 。复制粘贴下我们新建仓库的 `SSH` 或 `HTTPS` URL 即可。
（例如：git@github.com:username/username.github.io.git）

那么这里 rake setup_github_pages 做了什么呢？
<!-- more -->

用户（users）的 Github Pages 使用 `master` 分支作为 `Web 服务`（web server）的公开目录，为我们的 `Pages url` （http://username.github.io）提供内容文件。因此，我们会有这样的需求，`source` 分支用来做与`博客源码`相关的事（存放全部博客源码），`master` 分支上 commit 生成的博客内容`供 Web 访问`。而 Octopress 帮我们把这件事给搞定了，通过这行 code（好 NB～）。

下面具体分析下 Octopress 是怎么做的（可通过查看 Rakefile 得知）：

(1). 命令要求我们输入 Github Pages 仓库的 URL，也就是我们新建的名为 username.github.io 仓库的 URL。这样命名是为了通过字符串截取 URL 拿到子串（http://username.github.io）作为我们博客的域名；

{% codeblock lang:ruby %}
  # Rakefile 中可查看 URL 截取方式：
  repo_url = get_stdin("Repository url: ")
{% endcodeblock %}

(2). 将指向（pointing to）imathis/octopress 远程库的名字 `‘origin’` 改为 `‘octopress’`；

Git clone 一个仓库时，会将 clone 下来的仓库命名为 origin，没有限定 clone 条件的情况下，会下载仓库中所有数据，并建立一个指向该仓库 master 分支的指针，本地命名为 `origin/master`。

当我们 clone 了 octopress 仓库，执行如下命令可看到远程仓库信息：

{% codeblock lang:ruby %}
  $ cd octopress
  $ git remote -v
  
  # 输出如下，可看到远程仓库名为 origin
  origin	git://github.com/imathis/octopress.git (fetch)
  origin	git://github.com/imathis/octopress.git (push)
{% endcodeblock %}

`Rakefile` 中可看出是通过如下的方式，将 origin 改为 octopress:

{% codeblock lang:ruby %}
  # 查看 Rakefile
  system "git remote rename origin octopress"
{% endcodeblock %}

这里内部执行了命令 `git remore rename origin octopress`，当 `rake setup_github_pages` 执行完毕，再 `git remote -v` 发现远程库名改为了 octopress。

{% codeblock lang:ruby %}
  $ git remote -v
  # 输出如下
  octopress	  git://github.com/imathis/octopress.git (fetch)
  octopress	  git://github.com/imathis/octopress.git (push)
{% endcodeblock %}

(3). 添加你的 Github Pages 仓库作为默认的 origin remote，并将远程库中指向 imathis/octopress 中 master 分支的指针指向现在的 origin（即 username/username.github.io）的 master 分支，

{% codeblock lang:ruby %}
  # 查看 Rakefile
  system "git remote add origin #{repo_url}"
  system "git config branch.master.remote origin"
{% endcodeblock %}

当 `rake setup_github_pages` 执行完毕查看远程库，可以看到远程库 origin 指向了 Github Pages。

{% codeblock lang:ruby %}
  $ cd octopress
  $ git remote -v
  
  # 输出信息如下
  octopress	git://github.com/imathis/octopress.git (fetch)
  octopress	git://github.com/imathis/octopress.git (push)
  origin	git@github.com:JonyFang/JonyFang.github.io.git (fetch)
  origin	git@github.com:JonyFang/JonyFang.github.io.git (push)
{% endcodeblock %}

到这里，应该能猜到上一步将指向 `imathis/octopress` 远程库的名字 `origin` 改为 `octopress` 的原因了。

(4). 将本地 master 分支名字从 `master` 改为 `source`

{% codeblock lang:ruby %}
  # 查看 Rakefile
  system "git branch -m master source"
{% endcodeblock %}

执行如下命令查看本地分支（拿到第6条解释为什么要改名为 source）：

{% codeblock lang:ruby %}
  $ git branch
  
  # 输出如下
  * source
{% endcodeblock %}

(5). 根据提供的 Github Pages 仓库的 `SSH` 或 `HTTPS` 的 URL，截取仓库名 `username.github.io` 作为博客的 URL（上面 1 有提到）。然后将 octopress 目录下 `_config.yml` 文件中 url 参数值改为 `http：//username.github.io`。

{% codeblock lang:ruby %}
  # octopress 下 Rakefile 查看
  
  url = blog_url(user, project, source_dir)
  jekyll_config = IO.read('_config.yml')
  jekyll_config.sub!(/^url:.*$/, "url: #{url}")
  File.open('_config.yml', 'w') do |f|
  f.write jekyll_config
{% endcodeblock %}

(6). 在 octopress 目录下新建 `_deploy` 目录，并在 _deploy目录下新建 `master` 分支；

前面4 ，我们将本地 master 分支名字从 `master` 改为 `source`，这里一起分析下原因。4和6放在一起容易理解点。

其实本地 octopress 博客部署到 Github Pages 之后，远程 Github 下会有两个分支， `master` 和 `source`。远程 `master` 分支作为 `Web 服务`公开目录，当你访问 `http://username.github.io` 时，提供网站内容；远程 `source` 分支存放的是整个 `octopress 框架`的源码。

之所以第4步将本地 `master` 改为 `source`，是为了把 `master 指针`让出来，让它指向这一步（6）新建的 `_deploy` 目录下的 master 分支。这样，`octopress/_deploy` 目录下的本地 master 分支 就对应了 Github Pages 远程库中的 master 分支，本地 source 分支同理。

{% codeblock lang:ruby %}
  # 查看 Rakefile 
  cd "#{deploy_dir}" do
  system "git init"
  system 'echo "My Octopress Page is coming soon …" > index.html'
  system "git add ."
  system "git commit -m \"Octopress init\""
  system "git branch -m gh-pages" unless branch == 'master'
  system "git remote add origin #{repo_url}"
{% endcodeblock %}

最后修改了 `Rakefile` 中 `deploy_default` 和 `deploy_branch` 变量的初始值：

{% codeblock lang:ruby %}
  # deploy 时执行的命令，"push" 为 Rakefile 中定义的一个 rake task
  deploy_default = "push" # 初始为 "rsync"
  
  # deploy 时执行上述 rake task 命令 "push" 到 "master" 分支
  deploy_branch  = "master" # 初始为 "gh-pages"
{% endcodeblock %}

再回头来看 `rake setup_github_pages` ，是不是清晰多了呢？


### 1.3. 生成并部署站点

执行如下命令，（将 `octopress/_deploy` 下数据 push 到 `master` 分支）：

{% codeblock lang:ruby %}
  $ sudo rake generate
  $ sudo rake deploy
{% endcodeblock %}

这时在浏览器输入 `http://username.github.io` 就可以访问你的博客网页啦～

最后别忘了 commit 你的 `octopress 框架源码`到 `source` 分支：

{% codeblock lang:ruby %}
  $ git add .
  $ git commit -m'init'  #init 可随意填写
  $ git push origin source
{% endcodeblock %}

好，到这里，如果顺利完成前面所有内容的话，我们已经将 `Octopress` 部署到 `Github Pages` 了。如果你想换成自己的域名可以参考这里的方法（[Custom Domains](http://octopress.org/docs/deploying/github/#custom_domains)），不再赘述了。

<p class="info">
这里分析下，`rake generate` 和 `rake deploy`。
</p>

rake generate：生成 `jekyll` 站点（Generating Site with Jekyll）

{% codeblock lang:ruby %}
  # 查看 Rakefile
  system "compass compile --css-dir #{source_dir}/stylesheets"
  system "jekyll build"
{% endcodeblock %}

rake deploy：将站点部署到 `Github Pages`。由于 `_deploy` 目录所代表的本地仓库的 `master` 分支对应 `Github Pages` 远程仓库的 `master` 分支，该分支目录的内容即 `Github Pages` 在互联网上供公开访问的站点内容。因此这里做的主要就是将改动的内容（博客、DIY 布局等...） copy 到 `_deploy` 目录下，然后将此修改 push 到 `Github Pages` 远程库的 `master` 分支。

1. 查看是否在 `preview-mode`(预览模式)，有则删除，重新执行 `rake generate`；

2. 将 `octopress/source` 目录下的文件拷贝到 `octopress/public` 目录下；

3. 进入 `octopress/_deploy` 目录，执行 `git pull` 操作；

4. 将 `octopress/public` 目录的内容拷贝到 `octopress/_deploy` 目录下；

5. 将 `octopress/_deploy` 目录所对应的本地 `master` 分支的修改 `push` 到 `Github Pages` 远程库的 master 分支。

6. 下面可以看到 `master` 分支 `commit` 信息格式是 `"Site updated at 2015-10-14 12:52:36 UTC"` 。

{% codeblock lang:ruby %}
  # 查看 Rakefile
  system "git add -A"
  message = "Site updated at #{Time.now.utc}"
  system "git commit -m \"#{message}\""
  
  # 前面说过 deploy_branch  = "master"
  Bundler.with_clean_env { system "git push origin #{deploy_branch}" }
{% endcodeblock %}


## 2. 发布博文过程

### 2.1. 新建一篇博文

打开终端，输入：

{% codeblock lang:ruby %}
  $ cd octopress
  $ rake new_post["Hi, octopress"]  # "Hi, octopress" 是文章的标题
{% endcodeblock %}

然后在 `octopress/source/_posts` 目录下我们会看到命名格式为 `"2015-10-14-Hi-octopress.markdown"` 的文件。

{% codeblock lang:ruby %}
  # 查看 Rakefile ，这是实现方法
  filename = "#{source_dir}/#{posts_dir}/#{Time.now.strftime('%Y-%m-%d')}
  #{title.to_url}.#{new_post_ext}"
{% endcodeblock %}

打开新建的 `"2015-10-14-Hi-octopress.markdown"` 文件（我用的 [Mou](http://mouapp.com/)，免费还好用），会发现头文件有如下内容（不要删除这段信息）：

{% codeblock lang:ruby %}
  ---
  layout: post             #代表是一片博文
  title: "hello world" 
  date: 2015-10-14 19:59:22 +0800
  comments: true         #是否允许评论
  categories:             #分类
  ---
{% endcodeblock %}

头部布局实现原理：

{% codeblock lang:ruby %}
  # 查看 Rakefile，头部布局实现
  open(filename, 'w') do |post|
  post.puts "---"
  post.puts "layout: post"
  post.puts "title: \"#{title.gsub(/&/,'&')}\""
  post.puts "date: #{Time.now.strftime('%Y-%m-%d %H:%M:%S %z')}"
  post.puts "comments: true"
  post.puts "categories: "
  post.puts "---"
{% endcodeblock %}

在最后面的－－－下面就可以开始我们的正文啦～

### 2.2. 预览新建的博文

终端执行如下命令：

{% codeblock lang:ruby %}
  $ cd octopress
  $ sudo rake generate
  $ rake preview
{% endcodeblock %}

在浏览器中打开[http://localhost:4000](http://localhost:4000/),即可以预览我们刚 post 的博客效果。

### 2.3. 发布新建的博文

终端执行：

{% codeblock lang:ruby %}
  $ sudo rake deploy  #deploy blog 到 Github Pages
{% endcodeblock %}

最后别忘了，push 下本地的 octopress 到 source：

{% codeblock lang:ruby %}
  $ git add .
  $ git commit -m'post test blog'
  $ git push origin source
{% endcodeblock %}

OK，到此，本地 octopress 博客部署到 Github Pages 完成了，打开访问你的个人博客看看效果吧。额，因为还没有做布局修改，留到下一篇介绍～


<p class="info">
部署这一块，自己花了挺长时间研究的，大体上理清了自己之前在做时的一些疑问。现在回头再看 Github 上的目录和本地的目录，一下子明朗了不少。不知道你是不是也觉得呢？
</p>