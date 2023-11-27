+++
title = "索引模板设置"
date = 2023-11-14T15:25:14+08:00
draft = true
author = 'vdong'
categories = ['技术']
tags = ['es']

+++

## 索引模板

索引模板会告诉 es 在创建索引是如何配置索引（设置索引分片shard数，mapping字段类型映射，别名等）。

`Elasticsearch  7.8` 后 一直引入了可组合的组件模板 `component_template`，故当前有两种类型 index templates 和 component templates。组件模板不能直接被用于索引，但是可以被重复使用，可用于配置 mappings, settings, and aliases。

## 内置模板和优先级

Elasticsearch 具有内置索引模板，每个模板的优先级为 `100` ，适用于以下索引模式：

- `logs-*-*`
- `metrics-*-*`
- `synthetics-*-*`
- `profiling-*`

Elastic Agent 使用这些模板来创建数据流，由 Fleet 集成创建的索引模板使用聚合组件索引的模式，且优先级最高为 `200` 。如果要覆盖该索引，需设置优先级 `priority ` 大于 200。

## 创建模板

```console
PUT /_index_template/template_name
{
  "index_patterns" : ["logstash-switch-*"],
  "priority" : 201,
  "template": {
    "settings" : {
      "number_of_shards" : 1
      "number_of_replicas" : 1
    }
  },
  "mappings": {
  	"dynamic_templates":{
  	
  	},
  	"properties": {
  	
  	}
  }
}
```

## 其他api

```console
// 获取所有模板，过滤只显示名字
GET _index_template
GET _index_template?filter_path=*.name

// 获取模板表格
GET _cat/templates?v

// 返回200 代表存在，404 代表不存在
HEAD _index_template/m_template

// 获取 metrics-system.diskio 内置的模板
GET _index_template/metrics-system.diskio

// 删除模板
DELETE _index_template/m_template
```

