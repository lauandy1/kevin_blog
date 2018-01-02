---
layout: post
title: ELK日志平台搭建
author: andy
tags:  elasticsearch logstash filebeat kibana
categories:  elasticsearch
excerpt: ELK日志平台搭建，包括filebeat日志收集，logstash日志收集和过滤日志，elasticsearch存储，kibana展示
---

* TOC
{:toc}

# ELK日志平台简介
网上介绍ELK的文章很多，这里我记录一下自己的理解。
Elasticsearch是核心，用于存储日志，kibana提供了API读取elasticsearch中的数据进行展示。
Logstash用于收集日志和处理日志，最后形成elasticsearch中的索引数据。
Filebeat是轻量级的日志收集器，监控文件变化进行收集，占用的资源较少，可以部署在产生日志的服务器上。
Logstash功能强大，但是比较占用资源。因此，一般是通过filebeat在服务器端收集日志，然后吐到统一的Logstash集群上进行日志的集中处理，
最后把处理过的日志吐到elasticsearch中进行存储。
ELK三大组件中，Logstash可以被替换，可以通过任何其他方式把日志写入到elasticsearch中进行存储。

# ELK关键配置

## filebeat配置
	output.logstash:
	  # The Logstash hosts
	  hosts: ["10.109.3.181:5044","10.109.3.182:5044"]
	  loadbalance: true
	output.console:
	  # Boolean flag to enable or disable the output module.
	  enabled: true
	  # Pretty print json event
	  pretty: true
	processors:
	 - drop_fields:
	     fields: ["offset","input_type","beat"]