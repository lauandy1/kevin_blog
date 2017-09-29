---
layout: post
title: 客户端和服务端请求加密认证
author: andy
tags:  security
categories:  security
excerpt: 场景、加密方式
---

* TOC
{:toc}

# 前言

目的：APP应用的服务端接口，因返回数据格式较规整，内容也较全面，容易被第三方作为获取数据的信息源。在APP应用与服务端接口之间交互信息时建立认证机制，能很大程度避免数据被抓取的可能。
比如：参数中的拍品id换一个有效的拍品id，即可把该拍品的信息获取到。

# 概要

## 使用方式
在APP请求服务端接口时，增加一个认证的参数_ptk，参数值为和其它所有参数有关的加密串，服务端接收到请求时，先验证此参数有效性，验证不通过直接拒绝请求，中止程序运行。


加密串放到请求header头里，以Access-tk标识发送给服务端验证。比如：

	<header Access-tk:ue82ed>

加密算法在客户端和服务端分别计算，获取的值一致即通过。这要求客户端对加密算法这块做好保护，比如android加固，防止被反编译后算法泄漏从而模拟加密串请求通过验证。

## 加密算法

publishid=1676549&session=70f638907a795f5e1c66434f5574e79e&so=g*o&tp=th ok&username=gues/tt&version=7.0&ilikeuxinc36c9e20ed891a38337837e79f989b6f33c23e



    把所有请求参数按参数名索引排序。
    按 参数1名=参数1值&参数2名=参数2值…进行拼接，然后拼接一个&符，再拼接私钥。即：k1=v1&k2=v2&k3=v3…&pk （注意在私钥前还有一个&符）
    把拼接字符串进行MD5，然后取MD5值的第20，17，0，6，1，5位（0位表示第一个）。


