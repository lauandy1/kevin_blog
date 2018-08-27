---
layout: post
title: linux创建用户及root权限
author: andy
tags:  linux
categories:  linux
excerpt: linux ubuntu创建用户及root权限
---

* TOC
{:toc}

# 创建用户
注意Ubuntu下创建用户用adduser命令。

# 赋予root权限
需要添加要sudo组里，用sudo adduser yangzhichao sudo

# 设置ssh免密登录
新创建的用户目录下并没有.ssh目录，需要手动创建。
注意切换到yangzhichao用户下创建。
注意要设置.ssh目录的权限，一般设置为700或者755。
因为sshd为了安全，对属主的目录和文件权限有所要求。如果权限不对，则ssh的免密码登陆不生效。
用户目录权限为 755 或者 700，就是不能是77x、777，需要保障other用户不能有w权限。
rsa_id.pub 及authorized_keys权限一般为644。
rsa_id权限必须为600。

# 参考
1、https://askubuntu.com/questions/7477/how-can-i-add-a-new-user-as-sudoer-using-the-command-line

2、https://blog.csdn.net/levy_cui/article/details/59524158

