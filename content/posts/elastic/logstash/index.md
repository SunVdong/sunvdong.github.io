+++
title = 'Logstash'
date = 2021-11-04T15:20:06+08:00
draft = false
author = 'vdong'
categories = ['技术']
tags = ['es','logstash']
+++

## 条件判断语句

- 相等: ==, !=, <, >, <=, >= 

- 正则: =~(匹配正则), !~(不匹配正则) 

- 包含: in(包含), not in(不包含) 

- 布尔操作： and(与), or(或), nand(非与), xor(非或) 

- 一元运算符： !(取反) 

- 复合表达式： ()(复合表达式), !()(对复合表达式结果取反) 

## 测试管道

以下 generator 会触发一个生成事件，即产生一条记录。
```
input {
    generator { 
        lines => ["Generated line"] 
        count => 1 
    } 
}
```

## 一些 filter

### 使用本地Geo数据库，并添加标签
```
geoip {
        source => "source_ip"
        target => "source_geoip"
        database => "/usr/share/logstash/data/geolite/GeoLite2-City.mmdb"
        add_tag => ["%{[source_geoip][country_name]}","%{[source_geoip][city_name]}"]
        tag_on_failure => "_geoip_source_fail"
    }
```

### 运行ruby代码

- 行内运行

```
# 原数据 [{"interface_name":"xxx" ...},{...}] 修改 interface_name key名 为 IfName 等
ruby {
    code => '
        interface_msg = event.get("interface_msg")
        if interface_msg
            interface_msg.each_with_index do |interface, index|
                interface["IfName"] = interface["interface_name"]
                interface.delete("interface_name")

                interface["IfMAC"] = interface["interface_mac"]
                interface.delete("interface_mac")

                if interface["interface_status"]==1
                    interface["IfStatus"] = "up"
                else
                    interface["IfStatus"] = "down"
                end
                interface.delete("interface_status")
                event.set("interface_msg", interface_msg)
            end
        end
    '
}
```

- 单独文件

```
ruby  {
    path => "/usr/share/logstash/script/rt_device_interface_data.rb"
}
```

rt_device_interface_data.rb 文件内容如下：

```ruby
def register(params)
end

# 将 interface_state 转为 json 字符串
# {RxDropped=0, IfMtu=1500},{RxDropped=0, IfMtu=1500}
# [{"RxDropped":"0","IfMtu":"1500"},{"RxDropped":"0","IfMtu":"1500"}]
def filter(event)
    str = event.get('interface_state')

    log_dict_list = str.split("},{").map do |log|
      log.gsub(/[{}]/, '').split(", ").map do |kv|
        k, v = kv.split("=")
        [k.to_sym, v]
      end.to_h
    end
    # 将字典列表转换为JSON字符串
    #log_json_string = JSON.generate(log_dict_list)

    event.set('interface_state', log_dict_list)

    return [event]
end
```

## 配置管道

在 logstash 配置文件 `pipelines.yml` 中新增管道和配置路径:

```
- pipeline.id: main
  path.config: "/usr/share/logstash/pipeline"
- pipeline.id: dp 
  path.config: "/usr/share/logstash/dp_pipeline"
```

管道 main 是默认的管道，我们新增了dp管道。

在配置路径下新增配置文件，配置管道处理数据的逻辑即可：

```
input {
    udp {
        port => 10523
        type => "gateway"
    }
}
filter {
    # code more ...
}
output {
    # code more ...
}

```

### 两个管道相连

我们希望数据先经过 main 管道，再经过 dp 管道，可以做如下操作：

1. 在 main 管道 配置文件的 output 中将数据发送至一个管道虚拟地址，下例为 `dp_router`

```
output {
    pipeline { send_to => dp_router }
}
```

2. 在 dp 管道 配置文件的 input 中从虚拟地址接收数据, 然后即可处理数据：

```
input {
    pipeline {
        address => dp_router
    }
}
```

#### pipeline to pipeline 的常用模式

1. 分发 (一对多)

有多种类型的数据通过单个输入传入，且每种数据都有自己复杂的处理规则集的情况下，您可以使用分发器模式。

```
input { beats { port => 5044 } }
output {
    if [type] == apache {
      pipeline { send_to => weblogs }
    } else if [type] == system {
      pipeline { send_to => syslog }
    } else {
      pipeline { send_to => fallback }
    }
}
```

2. 隔离

如果配置了多个输出，任意一个输出服务down了，将阻塞其他输出，使用输出隔离器模式和持久队列，即使一个输出出现故障，我们也可以继续发送到 Elasticsearch。

```
# config/pipelines.yml
- pipeline.id: intake
  config.string: |
    input { beats { port => 5044 } }
    output { pipeline { send_to => [es, http] } }
- pipeline.id: buffered-es
  queue.type: persisted
  config.string: |
    input { pipeline { address => es } }
    output { elasticsearch { } }
- pipeline.id: buffered-http
  queue.type: persisted
  config.string: |
    input { pipeline { address => http } }
    output { http { } }
```
3. fork 

我们在自己的系统中收集完整的索引信息，但同时我们也需要将数据发送给合作伙伴，这是我们需要对索引进行额外处理（例如：删除敏感信息等）

```
# config/pipelines.yml
- pipeline.id: intake
  queue.type: persisted
  config.string: |
    input { beats { port => 5044 } }
    output { pipeline { send_to => ["internal-es", "partner-s3"] } }
- pipeline.id: buffered-es
  queue.type: persisted
  config.string: |
    input { pipeline { address => "internal-es" } }
    # Index the full event
    output { elasticsearch { } }
- pipeline.id: partner
  queue.type: persisted
  config.string: |
    input { pipeline { address => "partner-s3" } }
    filter {
      # Remove the sensitive data
      mutate { remove_field => 'sensitive-data' }
    }
    output { s3 { } } # Output to partner's bucket
```

4. 收集 (多对一)

许多管道流入单个管道，在其中共享输出和处理。

```
# config/pipelines.yml
- pipeline.id: beats
  config.string: |
    input { beats { port => 5044 } }
    output { pipeline { send_to => [commonOut] } }
- pipeline.id: kafka
  config.string: |
    input { kafka { ... } }
    output { pipeline { send_to => [commonOut] } }
- pipeline.id: partner
  # This common pipeline enforces the same logic whether data comes from Kafka or Beats
  config.string: |
    input { pipeline { address => commonOut } }
    filter {
      # Always remove sensitive data from all input sources
      mutate { remove_field => 'sensitive-data' }
    }
    output { elasticsearch { } }
```
