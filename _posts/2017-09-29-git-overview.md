---
layout: post
title: git简介
author: andy
tags:  git
categories:  git
excerpt: git和svn比较，git基本原理和命令，git开发最佳实践，git命令行实战演示
---

* TOC
{:toc}

# why git?git和svn相比的优势，为什么要使用git

* 最本质的区别。svn是集中式的，git是分布式的。好处就是git可以在本地进行提交，而svn必须联网才能提交。每一个开发人员的电脑上都有一个Local Repository,所以即使没有网络也一样可以Commit，查看历史版本记录，创建项目分支等操作，等网络再次连接上Push到Server端。当然git也带来了复杂性，需要建两个Repositories(Local Repositories & Remote Repositories),指令很多，需要知道哪些指令在Local Repository，哪些指令在Remote Repository。

* 分支的区别。由于svn是集中式的，新建分支必须在服务端。而git可以在本地随意建立分支，不用担心影响其他人开发，不需要的时候可以随意删除，需要合并时再处理合并。

* 最后总结一些：svn优点是原理和命令简单，缺点是很多地方使用不方便，比如需要联网提交，分支的建立和合并。git优点是使用便捷，本地提交，分支的创建和合并比较强大。缺点是原理和命令相对复杂，学习曲线相对高些。

# svn和git从架构上比较

集中式管理架构：

![git.png](/images/git/svnStructure.png)

分布式管理架构：

![git.png](/images/git/gitStructure.png)

我们在平时，会经常使用各种各样的索引，如我们根据链接，可以找到链接里的具体文本，这就是索引。反过来，如果，如果我们能根据具体文本，找到文本存在的具体链接，这就是倒排索引，可简单理解为从文本到链接的映射。我们平时在使用Google、百度时，就是根据具体文本去找链接，这就是以倒排索引为基础的。

# git和svn相关概念比较

    仓库(repository)，就是服务器端的数据。
    签出(checkout)：英语写成check out,意思就是从服务器拉取数据，这是svn中的概念，git里面类似的概念是克隆，即clone。
    克隆(clone):功能跟checkout差不多，但原理还是有一些差别的。
    提交(commit):这个svn跟git的区别还是蛮大的。svn中commit是直接将数据提交到服务器，而git中是将数据提交到本地，并生成一个新的版本号。所以，如果没有网络的话svn就commit不成功，但git是可以commit成功的，这个尤其注意。
    更新(update)：这个是svn中的概念，用于将服务器端的数据更新到本地，git中对应的是pull。
    拉和推(pull和push):这是git中的概念，之前说过，commit只是将数据提交到本地版本库，真正将数据提交到服务器的操作是push，这个时候服务器端的版本号也同步更新。

# git工作基本流程图

![git.png](/images/git/gitProcess.png)

    Workspace：工作区
    Index / Stage：暂存区
    Repository：仓库区（或本地仓库）
    Remote：远程仓库

# git基本命令

## 新建代码库

* 在当前目录新建一个Git代码库 * git init 
* 新建一个目录，将其初始化为Git代码库 * git init [project-name] 
* 下载一个项目和它的整个代码历史 * git clone [url]

## 配置

Git的设置文件为.gitconfig，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。

* 显示当前的Git配置 * git config --list 
* 编辑Git配置文件 * git config -e [--global] 
* 设置提交代码时的用户信息 * git config [--global] user.name "[name]" * git config [--global] user.email "[email address]"

## 增加/删除文件

* 添加指定文件到暂存区 * git add [file1] [file2] ... 
* 添加指定目录到暂存区，包括子目录 * git add [dir] 
* 添加当前目录的所有文件到暂存区 * git add . 
* 删除工作区文件，并且将这次删除放入暂存区 * git rm [file1] [file2] ... 
* 停止追踪指定文件，但该文件会保留在工作区 * git rm --cached [file] 
* 改名文件，并且将这个改名放入暂存区 * git mv [file-original] [file-renamed]

## 代码提交

* 提交暂存区到仓库区 * git commit -m [message] 
* 提交暂存区的指定文件到仓库区 * git commit [file1] [file2] ... -m [message] 
* 提交工作区自上次commit之后的变化，直接到仓库区 * git commit -a 
* 提交时显示所有diff信息 * git commit -v 
* 使用一次新的commit，替代上一次提交 
* 如果代码没有任何新变化，则用来改写上一次commit的提交信息 * git commit --amend -m [message] 
* 重做上一次commit，并包括指定文件的新变化 * git commit --amend [file1] [file2] ...

## 分支

* 列出所有本地分支 * git branch 
* 列出所有远程分支 * git branch -r 
* 列出所有本地分支和远程分支 * git branch -a 
* 新建一个分支，但依然停留在当前分支 * git branch [branch-name] 
* 新建一个分支，并切换到该分支 * git checkout -b [branch] 
* 新建一个分支，指向指定commit * git branch [branch] [commit] 
* 新建一个分支，与指定的远程分支建立追踪关系 * git branch --track [branch] [remote-branch] 
* 切换到指定分支，并更新工作区 * git checkout [branch-name] 
* 建立追踪关系，在现有分支与指定的远程分支之间 * git branch --set-upstream [branch] [remote-branch] 
* 合并指定分支到当前分支 * git merge [branch] 
* 选择一个commit，合并进当前分支 * git cherry-pick [commit] 
* 删除分支 * git branch -d [branch-name] 
* 删除远程分支 * git push origin --delete [branch-name] * git branch -dr [remote/branch]

## 标签

* 列出所有tag * git tag 
* 新建一个tag在当前commit * git tag [tag] 
* 新建一个tag在指定commit * git tag [tag] [commit] 
* 删除本地tag * git tag -d [tag] 
* 删除远程tag * git push origin :refs/tags/[tagName] 
* 查看tag信息 * git show [tag] 
* 提交指定tag * git push [remote] [tag] 
* 提交所有tag * git push [remote] --tags 
* 新建一个分支，指向某个tag * git checkout -b [branch] [tag]

## 查看信息

* 显示有变更的文件 * git status 
* 显示当前分支的版本历史 * git log 
* 显示commit历史，以及每次commit发生变更的文件 * git log --stat 
* 显示某个commit之后的所有变动，每个commit占据一行 * git log [tag] HEAD --pretty=format:%s 
* 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件 * git log [tag] HEAD --grep feature 
* 显示某个文件的版本历史，包括文件改名 * git log --follow [file] * git whatchanged [file] 
* 显示指定文件相关的每一次diff * git log -p [file] 
* 显示指定文件是什么人在什么时间修改过 * git blame [file] 
* 显示暂存区和工作区的差异 * git diff 
* 显示暂存区和上一个commit的差异 * git diff --cached [file] 
* 显示工作区与当前分支最新commit之间的差异 * git diff HEAD 
* 显示两次提交之间的差异 * git diff [first-branch]...[second-branch] 
* 显示某次提交的元数据和内容变化 * git show [commit] 
* 显示某次提交发生变化的文件 * git show --name-only [commit] 
* 显示某次提交时，某个文件的内容 * git show [commit]:[filename] 
* 显示当前分支的最近几次提交 * git reflog

## 远程同步

* 下载远程仓库的所有变动 * git fetch [remote] 
* 显示所有远程仓库 * git remote -v 
* 显示某个远程仓库的信息 * git remote show [remote] 
* 增加一个新的远程仓库，并命名 * git remote add [shortname] [url] 
* 取回远程仓库的变化，并与本地分支合并 * git pull [remote] [branch] 
* 上传本地指定分支到远程仓库 * git push [remote] [branch] 
* 强行推送当前分支到远程仓库，即使有冲突 * git push [remote] --force 
* 推送所有分支到远程仓库 * git push [remote] --all

## 撤销

* 恢复暂存区的指定文件到工作区 * git checkout [file] 
* 恢复某个commit的指定文件到工作区 * git checkout [commit] [file] 
* 恢复上一个commit的所有文件到工作区 * git checkout . 
* 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变 * git reset [file] 
* 重置暂存区与工作区，与上一次commit保持一致 * git reset --hard 
* 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变 * git reset [commit] 
* 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致 * git reset --hard [commit] 
* 重置当前HEAD为指定commit，但保持暂存区和工作区不变 * git reset --keep [commit] 
* 新建一个commit，用来撤销指定commit，后者的所有变化都将被前者抵消，并且应用到当前分支 * git revert [commit]

## 其他

初始化操作

    * git config -global user.name#设置提交者名字
    * git config -global user.email#设置提交者邮箱
    * git config -global core.editor#设置默认文本编辑器
    * git config -global merge.tool#设置解决合并冲突时差异分析工具
    * git config -list #检查已有的配置信息

创建新版本库

    * git clone#克隆远程版本库
    * git init #初始化本地版本库

修改和提交

    * git add . #添加所有改动过的文件
    * git add#添加指定的文件
    * git mv#文件重命名
    * git rm#删除文件
    * git rm -cached#停止跟踪文件但不删除
    * git commit -m#提交指定文件
    * git commit -m “commit message” #提交所有更新过的文件
    * git commit -amend #修改最后一次提交
    * git commit -C HEAD -a -amend #增补提交（不会产生新的提交历史纪录）

查看提交历史

    * git log #查看提交历史
    * git log -p#查看指定文件的提交历史
    * git blame#以列表方式查看指定文件的提交历史
    * gitk #查看当前分支历史纪录
    * gitk#查看某分支历史纪录
    * gitk --all #查看所有分支历史纪录
    * git branch -v #每个分支最后的提交
    * git status #查看当前状态
    * git diff #查看变更内容

撤消操作

    * git reset -hard HEAD #撤消工作目录中所有未提交文件的修改内容
    * git checkout HEAD#撤消指定的未提交文件的修改内容
    * git checkout HEAD. #撤消所有文件
    * git revert#撤消指定的提交

分支与标签

    * git branch #显示所有本地分支
    * git checkout#切换到指定分支或标签
    * git branch#创建新分支
    * git branch -d#删除本地分支
    * git tag #列出所有本地标签
    * git tag#基于最新提交创建标签
    * git tag -d#删除标签

合并与衍合

    * git merge#合并指定分支到当前分支
    * git rebase#衍合指定分支到当前分支

远程操作

    * git remote -v #查看远程版本库信息
    * git remote show#查看指定远程版本库信息
    * git remote add#添加远程版本库
    * git fetch#从远程库获取代码
    * git pull#下载代码及快速合并
    * git push#上传代码及快速合并
    * git push:/#删除远程分支或标签
    * git push -tags #上传所有标签

# git开发模式最佳实践

![git.png](/images/git/gitPractise.png)

## Git Flow常用的分支

    Production 分支

也就是我们经常使用的Master分支，这个分支最近发布到生产环境的代码，最近发布的Release， 这个分支只能从其他分支合并，不能在这个分支直接修改

    Develop 分支

这个分支是我们是我们的主开发分支，包含所有要发布到下一个Release的代码，这个主要合并与其他分支，比如Feature分支

    Feature 分支

这个分支主要是用来开发一个新的功能，一旦开发完成，我们合并回Develop分支进入下一个Release

    Release分支

当你需要一个发布一个新Release的时候，我们基于Develop分支创建一个Release分支，完成Release后，我们合并到Master和Develop分支

    Hotfix分支

当我们在Production发现新的Bug时候，我们需要创建一个Hotfix, 完成Hotfix后，我们合并回Master和Develop分支，所以Hotfix的改动会进入下一个Release

## Git Flow如何工作

初始分支

所有在Master分支上的Commit应该Tag

![git.png](/images/git/gitFlow1.png)

Feature 分支

分支名 feature/*

Feature分支做完后，必须合并回Develop分支, 合并完分支后一般会删点这个Feature分支，但是我们也可以保留

![git.png](/images/git/gitFlow2.png)

Release分支

分支名 release/*

Release分支基于Develop分支创建，打完Release分之后，我们可以在这个Release分支上测试，修改Bug等。同时，其它开发人员可以基于开发新的Feature (记住：一旦打了Release分支之后不要从Develop分支上合并新的改动到Release分支)

发布Release分支时，合并Release到Master和Develop， 同时在Master分支上打个Tag记住Release版本号，然后可以删除Release分支了。

![](/images/git/gitFlow3.png)

维护分支 Hotfix

分支名 hotfix/*

hotfix分支基于Master分支创建，开发完后需要合并回Master和Develop分支，同时在Master上打一个tag

![git.png](/images/git/gitFlow4.png)

# gitlab使用，如何down code，如何提交mergeRequest，如何进行codeReview

1）登录

    先绑定host git.youxinpai.com 172.16.100.67
    绑好之后，用域账号访问 https://git.youxinpai.com

2）下载windows git客户端

https://git-for-windows.github.io/

3）设置Git的user name和email，以及一个全局的gitignore配置，用于提交代码时忽略某些文件

下载客户端后，打开git bash命令行工具窗口，这个窗口是模拟linux系统命令，直接输入以下命令，设置git用户名和邮箱

$ git config --global user.name "yangzhichao"

$ git config --global user.email "yangzhichao@xin.com"

执行完后会在个人用户目录下生成.gitconfig文件，可以打开文件查看里面内容

![git.png](/images/git/gitC1.png)

接下来设置全局ignore。

还是在用户目录下，建立一个.gitignore_global文件:

cd ~

vim .gitignore_global

然后输入一些文件路径配置，如下图：

*.class

.idea/

classes/

target/

*.iml

/WEB-INF/

.DS_Store

.gitignore

.eclipse

.project

.classpath

.settings

.metadata

~

![git.png](/images/git/gitC2.png)

保存。

打开.gitconfig，输入如下配置[core]

![git.png](/images/git/gitC3.png)

这样就建立了关联。

4）配置sshKey

进入到个人用户目录（C:/Users/yangzhichao）：cd ~

生成sshkey：

ssh-keygen -t rsa -C “yangzhichao@xin.com"

按3个回车，密码为空。

5）下载项目

注意要切换到工作目录下载：例如进入到D:/project/git，执行：cd /d/project/git

然后执行git clone命令：

git clone https://git.youxinpai.com/javagroup/webserver_yxp.git

6）开发项目，提交文件

下载好工程后，进入工程目录：cd /d/project/git/webserver_yxp

![git.png](/images/git/gitC4.png)

可以看到路径最后括号里面显示出了当前的分支为master分支。

使用git branch命令查看当前所在分支： git branch

![git.png](/images/git/gitC5.png)

使用git branch -a命令查看远程仓库所有分支： git branch -a

![git.png](/images/git/gitC6.png)

可以看到有master、develop、feature/weibo、feature/zhangjinliang等分支，检出develop分支：git checkout develop

![git.png](/images/git/gitC7.png)

可以看到已经检出develop并且切换到了develop分支。

使用ll -a命令查看一下当前目录下有哪些内容：ll -a

![git.png](/images/git/gitC8.png)

下一步我们需要基于develop分支建立个人的工作分支，在当前分支为develop分支的情况下执行：git checkout -b feature/yangzhichao，

这个命令的意思是基于当前分支创建一个新的分支并且检出，从下图路径括号可以看到已经切换到了我的工作分支feature/yangzhichao。

![git.png](/images/git/gitC9.png)

我们开发要在自己工作分支上进行开发。

首先执行一个命令：git pull origin develop:develop，这个命令意思是拉取远程develop分支最新修改内容，并且合并到本地的develop分支。

当然，在合并到develop分支后，更新的内容自然也同步到了你的个人工作分支，因为工作分支是基于develop拉取的分支，和develop是同源的。

![git.png](/images/git/gitC10.png)

执行后，根据提示可以看出本地已经是最新的内容了。

大家在工作分支开发时，要经常使用这个命令拉取远程develop分支的别人提交的修改代码合并到自己的工作分支上，用于减少代码冲突的发生。

当然这也不能避免冲突，只是减少了冲突的几率。当发生冲突时，还是要先解决冲突。然后再提交。

接下来，我们在项目里做些修改，然后进行提交修改，推送修改内容到远程仓库等操作。

编辑一下README.md文件。然后执行git status命令。

![git.png](/images/git/gitC11.png)

可以看到，根据提示，README.md文件被修改，可以使用git add命令把修改保存到暂存区，也可以使用get checkout --命令撤销本次修改。

我们执行git add README.md命令，先保存到暂存区。然后执行git status命令查看当前状态。

![git.png](/images/git/gitC12.png)

可以看到，根据提示，已经保存到暂存区，下一步提示要进行commit操作或者执行git reset HEAD操作把文件从暂存区撤回。

我们执行git commit -m 'update README'命令，提交本次修改。写上修改的注释。然后执行git status再次查看状态。

![git.png](/images/git/gitC13.png)

可以看到，根据提示，当前目录已经没有任何修改未提交了。

由于这是新创建的分支，远程仓库还没有，所以我们下一步要把本地工作分支推到远程仓库中。执行：git push origin feature/yangzhichao

![git.png](/images/git/gitC14.png)

到此，我们已经将本地的工作分支推送到了远程仓库。当然，本地工作分支的修改内容也推送到了远程仓库。

下一步，我们需要提交merge request申请，把自己工作分支的改动内容合并到develop分支上，以用于其他同事同步此改动。

6）提交mergeRequest

登录gitlab进行操作，这里就不演示了。

# git教程推荐网址
http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000

