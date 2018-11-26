---
layout: post
title: supervisor管理springboot进程
author: andy
tags:  supervisor springboot
categories:  tool
excerpt: supervisor springboot
---

supervisor是python编写一个进程管理工具，通过supervisor启动的进程都是它的子进程，在子进程down掉时supervisor可以感知到，会自动重启子进程。

supervisor官方文档：http://supervisord.org/

常用的几个命令：

supervisorctl start bc-base 启动bc-base进程

supervisorctl stop bc-base

supervisorctl restart bc-base

supervisorctl status 查看所有子进程状态

supervisorctl update 配置文件变更后，执行此命令，启动新加入的进程

supervisorctl reload 重新加载配置，重新启动所有管理的进程

注意不要再用springboot.sh启动服务，会和supervisor冲突

注意不要再用kill -9杀进程，杀掉后会自动重启，想停服务用supervisorctl stop bc-base