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

注意输出到Logstash的配置。输出到控制台是用于调试，可以看到收集到的日志。

## logstash配置
logstash.yml文件内容没有任何改动。
logstash-filebeat-es-simple.conf文件配置如下：

	input {
	  beats {
	    port => 5044
	  }
	}
	filter {
	  if [message] =~ ".+gateway_common_log+.+" {
	    dissect {
	      mapping => {
	        "message" => "%{ts} %{+ts} %{pre} %{+pre} %{+pre} - %{+pre},%{clientType}|%{clientIp}|%{userId}|%{userAgent}|%{requestId}|%{requestUri}"
	      }
	    }
	  }
	  if [message] =~ ".+gateway_time_cost_log+.+" {
	    dissect {
	      mapping => {
	        "message" => "%{ts} %{+ts} %{pre} %{+pre} %{+pre} - %{+pre},%{apiPath}|%{timeCost}|%{requestId}"
	      }
	      convert_datatype => {
	        "timeCost" => "int"
	      }
	    }
	  }
	}
	output {
	   elasticsearch {
	    hosts => ["10.109.3.171:9200","10.109.3.172:9200","10.109.3.173:9200"]
	    user => "admin"
	    password => "sradmin-1122"
	    index => "business-%{+YYYY.MM.dd}"
	    manage_template => false
	    template_name => "business"
	    template_overwrite => true
	  }
	  stdout { codec => rubydebug }
	}

重点是filter部分配置，处理正则匹配的消息，从中提取出关键字行成新的消息格式。并且新的消息格式和elasticsearch模板格式是对应的。
第二端消息处理，是为了获得接口执行时间数据。其中还处理了类型转换，将timeCost转换为int值。
再看output部分，也很关键。index定义了索引的index模式。template_name指定了elasticsearch中的模板名字，并且需要把模板提前在elastsearch中初始化。
stdout配置可以输出到控制台，调试日志格式。

## elasticsearch配置
bc_template.json配置如下：

	{
	    "template" : "business-*",
	    "order":1,
	    "settings" : { "index.refresh_interval" : "60s" },
	    "mappings" : {
	        "_default_" : {
	            "_all" : { "enabled" : false },
	            "dynamic_templates" : [{
	              "message_field" : {
	                "match" : "message",
	                "match_mapping_type" : "string",
	                "mapping" : { "type" : "string"}
	              }
	            }, {
	              "string_fields" : {
	                "match" : "*",
	                "match_mapping_type" : "string",
	                "mapping" : { "type" : "string", "index" : "not_analyzed" }
	              }
	            }],
	            "properties" : {
	                "@timestamp" : { "type" : "date"},
	                "@version" : { "type" : "integer", "index" : "not_analyzed" },
	                "path" : { "type" : "string", "index" : "not_analyzed" },
	                "host" : { "type" : "string", "index" : "not_analyzed" },
	                "timeCost": {"type": "integer", "index" : "not_analyzed"}
	            }
	        }
	    }
	}

这个模板文件需要提前初始化到elasticsearch中，使用如下命令：

	curl -XPUT http://localhost:9200/_template/business -d @bc_template.json --user admin:sradmin

*需要注意的是，在Logstash中配置的index名称和bc_template.json中配置的template名称必须要匹配才行。
否则不生效，无法在elasticsearch中生成索引数据。
还有，需要配置一下timeCost属性。
index:not_analyzed表示字段不支持模糊匹配。

### elasticsearch命令
#### 查看es所有index

	curl http://localhost:9200/_cat/indices?v --user admin:sradmin

#### 按照index名称查看es某个index内容

	curl localhost:9200/business-2017.12.11/_search --user admin:sradmin

#### 按照index名称删除es某个index内容

	curl -XDELETE http://localhost:9200/business* --user admin:sradmin

#### 按照index名称查看es某个index的mapping

	curl -XGET "http://localhost:9200/filebeat-2017.12.07/_mapping?pretty" -uadmin

#### 查看es所有模板

	curl -XGET "http://localhost:9200/_template" --user admin:sradmin

#### 查看es某个模板

	curl -XGET "http://localhost:9200/_template/business" --user admin:sradmin

#### 按照名称删除es模板

	curl -XDELETE localhost:9200/_template/business --user admin:sradmin

#### 添加模板

	curl -XPUT http://localhost:9200/_template/business -d @bc_template.json --user admin:sradmin


