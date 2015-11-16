---
layout: post
title: "Git 常用命令和 Git Flow 梳理"
date: 2015-11-12 00:30 +0800
comments: true
categories: git
---

![git](http://7xob7d.com1.z0.glb.clouddn.com/git.png)

用 git 有一段时间了，之前没有详细地了解 git flow，导致协作过程中或多或少出现了一些头疼问题。最近静下心来理了下 git flow 的整个流程，再回头看开朗了不少，总结到这里。介绍的是一些常用的 git 基础命令和 git flow，当然也很重要的，过程中自己在 Github 上建了一个模拟的 [Demo](https://github.com/JonyFang/GitTestDemo) 用来熟悉 git flow。其实从理解到动手完成还是有点距离的，笨人有笨法嘛。如有不准确的地方欢迎指正。: )
{:.info}

* list element with functor item
{:toc}

#Git 常用命令

这里列出了一些比较常用的 git 命令，每个命令介绍后面都带一个简单例子～

###1. 新开分支

{% codeblock lang:ruby %}
 $ git branch 新分支名
	
 #新建分支 develop
 $ git branch develop
{% endcodeblock %}


###2. 切换到另一个分支

{% codeblock lang:ruby %}
 $ git checkout 分支名

 #切换到 develop 分支
 $ git checkout develop
{% endcodeblock %}

###3. 新开分支并切换到新分支

{% codeblock lang:ruby %}
 $ git checkout -b 新分支名

 #新开 develop 分支，并切换到此分支
 $ git checkout -b develop
{% endcodeblock %}

<!-- more -->

###4. 查看分支列表

{% codeblock lang:ruby %}
 $ git branch -a
{% endcodeblock %}

头部带 remotes/origin 的，表示远程分支

###5. 查看远程分支列表

{% codeblock lang:ruby %}
 $ git branch -r
{% endcodeblock %}

###6. 向远程仓库提交本地新开的分支

{% codeblock lang:ruby %}
 $ git push origin 新开分支名

 #提交新建的 develop 分支
 $ git push origin develop
{% endcodeblock %}

###7. 删除远程分支

{% codeblock lang:ruby %}
 $ git push origin --delete 远程分支名

 #删除远程仓库中的 develop 分支
 $ git push origin --delete develop
{% endcodeblock %}

###8. 删除本地分支

{% codeblock lang:ruby %}
 $ git branch -d 分支名

 #删除本地的 develop 分支
 $ git branch -d develop
{% endcodeblock %}

###9. 更新分支列表信息

{% codeblock lang:ruby %}
 $ git fetch -p
{% endcodeblock %}

用于协作时，项目队友添加或删除了远程分支的分支，可以通过这种方式来刷新分支列表信息


------------------------


#Git Flow 梳理

<span class='caption-wrapper'><img class='caption' src='http://7xob7d.com1.z0.glb.clouddn.com/git_flow.png' width='575' height='' title='git flow 图示'><span class='caption-text'>git flow 完整图示</span></span> 

Git 开发模式本质上是一套流程，团队每个成员遵守这套流程以确保完成可控的软件开发过程。[原文参考](http://nvie.com/posts/a-successful-git-branching-model/)

##1.主要分支

在远程仓库中有两个主要分支的生命期可以无限长，分别是：

**`Master`**

**`Develop`**

<span class='caption-wrapper'><img class='caption' src='http://jonyfang.github.io/images/git_flow/develop_branch.png' width='267' height='' title='develop 和 master 关系图'><span class='caption-text'>develop 和 master 关系图</span></span> 


**master 分支（origin/master）**

代码仓库中有且仅有的一条主分支，默认为 master ，在创建版本库时会自动创建。所有提供给用户使用的正式版本的源码，都会在这个分支上发布。也就是说主分支 master 用来发布重大版本。

**develop 分支（origin/develop）**

日常开发工作都会在 develop 分支上面完成。develop 分支可以用来生成代码的最新隔夜版本（nightly builds）。

**创建 `develop` 分支**

{% codeblock lang:ruby %}
 $ git checkout -b develop master
 
 #push develop 到远程仓库
 $ git push origin develop
{% endcodeblock %}

当我们在`develop`上完成了新版本的功能，最终会把所有的修改 `merge` 到 **`master`** 分支。针对每次 **`master`** 的修改都会打一个 Tag 作为可发布产品的版本号。


##2.辅助分支

开发过程中不可能项目人所有都在一个 `develop` 分支中开发，版本管理会很混乱。所以除了主要分支外，我们还需要一些辅助分支来协助团队成员间的并行开发。

**所用到的辅助分支大体分三类：**

* **Feature branches（功能分支）**
* **Release branches（预发布分支）**
* **Hotfix branches（热修复分支）**

通过分支名我们能知道各类型分支都有特定作用，对于他们各自的起始分支和最终的合并分支也都有严格规定。呼，虽然可能会麻烦点，但让人一目了然的效果还是很诱人的。

下面逐一介绍下各类型分支的创建使用和移除方法，过程中我在 `Github` 中创建一个虚拟的项目用来熟悉整个流程，或许你也可以像我一样做一遍。哈，动手总会有意外收获嘛。废话少说，继续正题～

###2.1.Feature branches（功能分支）

<span class='caption-wrapper'><img class='caption' src='http://7xob7d.com1.z0.glb.clouddn.com/feature_branches.png' width='133' height='' title='feature_branches'><span class='caption-text'>feature_branches</span></span> 


**应用场景：**

当要开始一个新功能的开发时，我门可以创建一个 `Feature branche` 。等待这个新功能开发完成并确定应用到新版本中就合并回 `develop`，那么如果不是就会被很遗憾的丢弃。。。

**应用规则：**

1. 从 `develop` 分支创建，最终合并回 `develop` 分支;

2. 分支名：feature／＊;

Tips：这里很多地方说用 **feature-＊** 的方式命名，因为公司项目中用的 **feature／＊**方式，也就习惯了，其实意思是一样的。

####(1).Creat a feature branch

{% codeblock lang:ruby %}
 $ git checkout -b feature/test develop
{% endcodeblock %}

 do something in `feature/test` branch
 
 push 本地 `feature/test` 到远处代码库；
 
{% codeblock lang:ruby %}
 $ git push origin feature/test
{% endcodeblock %}

####(2).切换到 **develop** 合并 **feature/test**

{% codeblock lang:ruby %}
 $ git checkout develop

 $ git merge --no-ff feature/test
{% endcodeblock %}
	
"- -no-ff" 的作用是创建一个新的 "commit" 对象用于当前合并操作。这样既可以避免丢失该功能分支的历史存在信息，又可以集中该功能分支所有历史提交。并且如果想回退版本也会比较方便。

<span class='caption-wrapper'><img class='caption' src='http://7xob7d.com1.z0.glb.clouddn.com/git_merge_noff.png' width='478' height='' title='git merge --no-ff 图示'><span class='caption-text'>git merge - -no-ff 图示</span></span> 

####(3).移除本地和远程仓库的 **feature/test** 分支

{% codeblock lang:ruby %}
 $ git branch -d feature/test

 $ git push origin --delete feature/test
{% endcodeblock %}


###2.2.Release branches（预发布分支）

**应用场景：**

"Release branches" 用来做新版本发布前的准备工作，在上面可以做一些小的 bug 修复、准备发布版本号等等和发布有关的小改动，其实已经是一个比较成熟的版本了。另外这样我们既可以在预发布分支上做一些发布前准备，也不会影响 "develop" 分支上下一版本的新功能开发。

**应用规则：**

1. 从 `develop` 分支创建，最终合并回 `develop` 和 `master`;

2. 分支名：release-＊;

####(1).Creat a release branch

{% codeblock lang:ruby %}
 $ git checkout -b release-1.1 develop
 
 #push 到远程仓库（可选）
 $ git push origin release-1.1
{% endcodeblock %}

do something in `release-1.1` branch

####(2).切换到 **master** 合并 **release-1.1**

{% codeblock lang:ruby %}
 $ git checkout master

 $ git merge --no-ff release-1.1

 $ git tag -a 1.1

 $ git push origin 1.1
{% endcodeblock %}

当我们的 `release-1.1` 的 Review 完成，也就预示着我们可以发布了。打上相应的版本号，再 **push** 到远程仓库。

####(3).切换到 **develop** 合并 **release-1.1**

预发布分支所做的修改同时也要合并回 **develop**

{% codeblock lang:ruby %}
 $ git checkout develop

 $ git merge --no-ff release-1.1
{% endcodeblock %}

####(4).移除本地和远程仓库的 **release-1.1**

{% codeblock lang:ruby %}
 $ git branch -d release-1.1

 $ git push origin --delete release-1.1
{% endcodeblock %}


###2.3.Hotfix branches（热修复分支）

<span class='caption-wrapper'><img class='caption' src='http://7xob7d.com1.z0.glb.clouddn.com/hotfixes.png' width='316' height='' title='Hotfix branches 图示'><span class='caption-text'>Hotfix branches 图示</span></span> 

**应用场景：**

"Hotfix branches" 主要用于处理线上版本出现的一些需要立刻修复的 bug 情况.

**应用规则：**

1. 从 `master` 分支上当前版本号的 `tag` 处切出，也就是从最新的 `master` 上创建，最终合并回 `develop` 和 `master`;

2. 分支名：hotfix-＊;

####(1).Creat a fixbug branch

{% codeblock lang:ruby %}
 $ git checkout -b fixbug-1.1.1 master
 
 #push 到远程仓库（可选）
 $ git push origin fixbug-1.1.1
{% endcodeblock %}

do something in `fixbug-1.1.1` branch

####(2).切换到 **master** 合并 **fixbug-1.1.1**

{% codeblock lang:ruby %}
 $ git checkout master

 $ git merge --no-ff fixbug-1.1.1

 $ git tag -a 1.1.1

 $ git push origin 1.1.1`
{% endcodeblock %}

bug 修复完成，合并回 `master` 并打上版本号；

####(3).切换到 **develop** 合并 **fixbug－1.1.1**

{% codeblock lang:ruby %}
 $ git checkout develop

 $ git merge --no-ff fixbug-1.1.1
{% endcodeblock %}

####(4).移除本地和远程仓库的 **fixbug-1.1.1**

{% codeblock lang:ruby %}
 $ git branch -d fixbug-1.1.1

 $ git push origin --delete fixbug-1.1.1
{% endcodeblock %}


#总结

啊哈，又到尾声了。上面的 `Git 常用命令`和 `Git Flow` 只是一些基本常识，并没有什么新的东西加入。在平时的团队协作中有些对于我自身而言还是比较新鲜的，比如 tag ，这些一般都交给老大来弄（因为他要 review）：D 。好像跑偏了。。。对自己而言，理一遍后的收获还是不小的，对 Git Flow 的整个流程有了大概的了解，但具体每个 git 命令内部是怎么处理的呢，这个留到后面再梳理一下～


>参考内容：
<br><br>
[Git分支管理策略](http://www.ruanyifeng.com/blog/2012/07/git.html)
<br>
[A successful Git branching model](http://nvie.com/posts/a-successful-git-branching-model/)

