---
layout: post
title: Install csshx on Mac OSX
author: andy
tags:  mac
categories:  mac
excerpt: Install csshx on Mac OSX
---

* TOC
{:toc}

# 安装home brew
Press Command+Space and type Terminal and press enter/return key.

Run in Terminal app:

ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" < /dev/null 2> /dev/null

and press enter/return key. 

If the screen prompts you to enter a password, please enter your Mac's user password to continue. When you type the password, it won't be displayed on screen, but the system would accept it. So just type your password and press ENTER/RETURN key. Then wait for the command to finish.

# 安装csshx
brew install csshx

# 配置csshx
在用户目录下创建文件.csshrc，编辑类似如下的集群。

clusters=sr-prod sr-emr sr-new
 
sr-prod=r1 r2 r3 r4 r5 r6 r7 r8 r9 r10
 
sr-emr=e1 e2 e3 e4 e5 e6 e7 e8 e9 e10 e11 e12
 
sr-new=r11 r12 r13

# 执行csshx命令打开多窗口控制台
csshX cluster_name

# 参考
1、http://macappstore.org/csshx/

