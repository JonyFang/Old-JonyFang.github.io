---
layout: post
title: "Octopress 适配 OS X El Capitan"
date: 2015-11-08 16:30 +0800
comments: true
categories: octopress
---

* list element with functor item
{:toc}

##1、前言
这几天装上 OS X El Capitan ，Time Machine 恢复后测试 Octopress 遇到无法使用问题，这里总结下问题的解决过程。

首先打开终端，运行：

{% codeblock lang:ruby %}
 $ rake preview
{% endcodeblock %}


报错如下：

{% codeblock lang:ruby %}
$ rakepreviewStarting to watchsourcewith Jekyll and Compass. Starting Rack on port

4000rake aborted!Errno::ENOENT: No such file or directory -

compass/Users/user/git/octopress/Rakefile:85:in

spawn/Users/user/git/octopress/Rakefile:85:in block inTasks:TOP=> preview
{% endcodeblock %}

在没有升级 OS X EI Capitan 之前一切如常，首先想到是不是 Ruby 的问题

{% codeblock lang:ruby %}
 $ ruby -v
 
 #输出
 ruby 2.0.0p645 (2015-04-13 revision 50299) [universal.x86_64-darwin15]
{% endcodeblock %}

和以前一样还是 2.0.0 版本，到官网查看最新版本是 2.2.3，多次尝试更新 `ruby` 失败。


##2、解决办法
更新 Ruby 可以通过 `rbenv` 或 `RVM`，因为我之前安装是通过 `RVM` 的方式，这里自己也是通过 `RVM` 的方式解决的。（附 Google 看到的：[rbenv 更新 Ruby 方法](https://gorails.com/setup/osx/10.11-el-capitan)）
<!-- more -->

###1. 清理 git 缓存

{% codeblock lang:ruby %}
 $ rm -rf /usr/local/.git
{% endcodeblock %}

###2. 安装 RVM

{% codeblock lang:ruby %}
 $ curl -L https://get.rvm.io | bash -s stable --ruby
 $ rm -rf /usr/local/.git
{% endcodeblock %}

###3. 安装 Ruby 2.2.3

{% codeblock lang:ruby %}
 $ rvm install 2.2.3
 $ rvm use 2.2.3
 $ rvm rubygems latest
{% endcodeblock %}

查看下 Ruby 版本:

{% codeblock lang:ruby %}
 $ ruby -v
 $ ruby 2.2.3p173 (2015-08-18 revision 51636) [x86_64-darwin14]
{% endcodeblock %}

啊哈，切换过来了，其实也很简单，到这里就修复了吗？其实还没有，如你所见还有第4步。

###4. 安装 Octopress 依赖库

{% codeblock lang:ruby %}
 $ cd octopress
 $ sudo gem install bundler
 $ bundle install
{% endcodeblock %}

当 bundler 安装完毕之后，来测试下 `rake` 命令是否修复了：


{% codeblock lang:ruby %}
 $ rake generate
 ## Generating Site with Jekyll
    write source/stylesheets/screen.css
Configuration file: /Users/JonyFang/Desktop/octopress/_config.yml
            Source: source
       Destination: public
      Generating... 
                    done.
 Auto-regeneration: disabled. Use --watch to enable.
{% endcodeblock %}

可以了～
 
如果你也同样遇到升级 OS X EI Capitan 后 Octopress 无法使用的情况，希望能对你有所帮助。


>本文参考：
<br><br>
[When I upgraded the Mac system, I can't Preview](https://github.com/imathis/octopress/issues/1749)
<br><br>
[Setup Ruby On Rails on Mac OS X 10.11 El Capitan](https://gorails.com/setup/osx/10.11-el-capitan)