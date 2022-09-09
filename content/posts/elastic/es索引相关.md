+++
title = 'Es 索引相关'
date = 2021-11-04T11:11:06+08:00
draft = false
author = 'vdong'
categories = ['技术']
tags = ['es']
+++

##  RESTful 接口

> - GET（SELECT）：从服务器取出资源（一项或多项）。
> - POST（CREATE）：在服务器新建一个资源。
> - PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。
> - PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）。
> - DELETE（DELETE）：从服务器删除资源。

以下通过 kibanan 操作。

## 基本 curd 操作

### 新增

```
# post vdong/_doc/<id> ,不写id会自动生成随机的 _id
POST vdong/_doc
{
  "name": "vodng",
  "age": 23,
  "sex":"男"
}

GET vdong/_doc/t1pqInwB9bbxQEztANxo
```

### 查看

```
GET vdong/_doc/t1pqInwB9bbxQEztANxo 

// 查看 id 为 1 的文档是否存在 
HEAD vdong/_doc/1

// 查看索引是否存在
HEAD vdong

// 获取 vdong 索引文档个数
GET vdong/_count
```

### 修改

```
// 需要完整资源信息
PUT vdong/_doc/12
{
  "name": "栋",
  "age": 23,
  "sex":"男"
}

// 不需要完整资源
POST vdong/_update/12
{
  "doc": {
    "name": "ss"
  }
}
```

#### 宽松的修改

```
// id 1002 的如果不存在，不会报错, 会新增一个文档
POST vdong/_update/1002
{
  "doc": {
    "name": "ss"
  },
  "doc_as_upsert": true
}
```

#### 查询并修改

```
// 查询并修改
POST vdong/_update_by_query
{
  "query":{
    "match": {
      "name": "ccc"
    }
  },
  "script":{
    "source": "ctx._source.age=params.age;ctx._source.sex=params.sex",
    "lang": "painless",
    "params": {
      "age":77,
      "sex":"女"
    }
  }
}

// 处理中文字段
POST vdong_cn/_update_by_query
{
  "query":{
    "match": {
      "姓名": "ss"
    }
  },
  "script":{
    "source": "ctx._source[\"年纪\"]=params[\"年纪\"]",
    "lang": "painless",
    "params": {
      "年纪":6
    }
  }
}

// ctx['_op'] 来删除文档
POST vdong_cn/_update_by_query
{
  "query":{
    "match": {
      "姓名": "ss"
    }
  },
  "script":{
    "source": """
    if(ctx._source["年纪"]<34){
      ctx.op = 'delete'
    }
    """
  }
}
```

### 删除

``` 
// 删除文档
DELETE vdong/_doc/1002

// 删除索引
DELETE vdong

// 根据搜索删除
POST vdong/_delete_by_query
{
  "query": {
    "term": {
      "realname": {
        "value": "dong"
      }
    }
  }
}
```

### 批量操作

```
// 创建索引 index 总会成功，如果 _id 已经存在，则更新数据
POST _bulk
{ "index" : { "_index" : "vdong", "_id" : "1" } }
{ "name" : "小明", "sex" : "男", "age" : "18" }
{ "index" : { "_index" : "vdong", "_id" : "2" } }
{ "name" : "小红", "sex" : "女", "age" : "20" }

// 创建索引 create ，如果 _id 存在，该条不会成功
POST _bulk
{ "create" : { "_index" : "vdong", "_id" : "2" } }
{ "name" : "小明", "sex" : "男", "age" : "18" }
{ "create" : { "_index" : "vdong", "_id" : "20" } }
{ "name" : "小红", "sex" : "女", "age" : "20" }

// 删除
{ "delete" : { "_index" : "vdong", "_id" : "2" } }

// 更新
{ "update" : {"_id" : "1", "_index" : "vdong"} }
{ "doc" : {"field2" : "value2"} }
```

[批量相关文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)
