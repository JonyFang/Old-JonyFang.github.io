---
layout: post
title: "Cocoapods 版本升级"
date: 2015-11-09 06:40 +0800
comments: true
categories: cocoapods
---


升级 OS X El Capitan 后，Time Machine 恢复下，随后因为遇到 octopress 无法更新博客问题，一番折腾 [octopress 适配了 OS X El Capitan](http://www.jianshu.com/p/ce5eff554279)，然而事情总没有那么简单，又丢过来一个问题。
{:.info}

<!-- more -->

* list element with functor item
{:toc}

和往常一样使用 Cocoapods ，执行命令：

{% codeblock lang:ruby %}
 $ pod install
 #输出信息
/System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/ruby/2.0.0/rubygems/dependency.rb:296:in `to_specs': Could not find 'cocoapods' (>= 0) among 59 total gem(s) (Gem::LoadError)
	from /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/ruby/2.0.0/rubygems/dependency.rb:307:in `to_spec'
	from /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/ruby/2.0.0/rubygems/core_ext/kernel_gem.rb:47:in `gem'
	from /usr/local/bin/pod:22:in `<main>'
{% endcodeblock %}

查看 `Ruby` 版本：

{% codeblock lang:ruby %}
 $ ruby -v
 ruby 2.2.3p173 (2015-08-18 revision 51636) [x86_64-darwin14]
{% endcodeblock %}

看来是升级 `Ruby 2.2.3` 导致的问题，更新下 Cocoapods 即可，更新步骤：

1、 更新 gem ，国内需切换 `gem source`

{% codeblock lang:ruby %}
 $ sudo gem update --system
{% endcodeblock %}

* 切换 `gem source`

{% codeblock lang:ruby %}
$ gem sources --add https://ruby.taobao.org/ --remove https://rubygems.org/

$ gem sources -l
*** CURRENT SOURCES***

https://ruby.taobao.org
{% endcodeblock %}

2、 安装 `cocoapods`

{% codeblock lang:ruby %}
 $ sudo gem install cocoapods
 $ pod setup
{% endcodeblock %}

和安装过程是一样的，再次查看 `pod` 版本：
{% codeblock lang:ruby %}
 $ pod --version

 0.39.0
{% endcodeblock %}

搞定～