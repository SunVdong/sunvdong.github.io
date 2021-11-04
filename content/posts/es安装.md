+++
title = 'Es 安装'
date = 2021-11-04T08:11:06+08:00
draft = false
author = 'vdong'
categories = ['技术'] 
tags = ['es']
+++

## Elasticsearch

```
# 安装
docker pull elasticsearch:7.14.0
```

```shell
docker network create elk_net

# 指定内存运行 
docker run -d -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -h elasticsearch --name elasticsearch --net elk_net -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.14.0

# 访问 http://localhost:9200 or http://host-ip:9200
```

head 插件（可选）

```shell
docker pull mobz/elasticsearch-head:5

docker run -d -p 9100:9100 docker.io/mobz/elasticsearch-head:5

# 访问 http://localhost:9100 or http://host-ip:9100

# 跨域问题
# 进入 elasticsearch 容器内部，修改配置文件 ./config/elasticsearch.yml
# 增加
http.cors.enabled: true
http.cors.allow-origin: "*"
# 重启
docker restart  elasticsearch

# ElasticSearch-head 查询报 406错误码 
# {"error":"Content-Type header [application/x-www-form-urlencoded] is not supported","status":406}
# _site 目录下
sed -i '6886c contentType: "application/json;charset=UTF-8",' vendor.js
sed -i '7573c var inspectData = s.contentType === "application/json;charset=UTF-8" &&' vendor.js
```



## Kibana

```shell
docker pull kibana:7.14.0

docker run -d -h kibana --name kibana --net elk_net -p 5601:5601 kibana:7.14.0

# 访问 http://localhost:5601 or http://host-ip:5601
```



## Logstash

```shell
docker pull logstash:7.14.0


1. 编辑文件 logstash.conf 文件


docker run -h logstash --name logstash --network elk_net  -it --rm  -v /usr/local/logstash/config/pipeline:/usr/local/logstash/config/pipeline logstash:7.14.0 -f /usr/local/logstash/config/pipeline/logstash.conf
```



## 网络问题

``` shell
docker network create elk_net

// 将运行中的 容器 containerName 连接到网络 networkName 中
docker network connect networkName containerName

// 将容器移除网络
docker network disconnect networkName containerName
```

