---
title: 'Hook的一个用法'
date: '2025-07-04T14:51:23+08:00'
draft: false
author: vdong
description: 'Hook的用法'
categories:
  - 技术
tags:
  - php
  - yii2
  - hook
  - 调试技巧
typora-root-url: ..\..\..\static
---

## 场景

在接手一个 yii2 项目时，发现一行代码 `$customizeCategory = \Yii::$app->params['customizeCategory'] ?? [];` 其中 `params['customizeCategory']` 不知道在哪里赋值的。查找了所有的 `config/params.php`  文件，甚至全局搜索了 `customizeCategory` 都没有找到赋值的代码。

## 解决方案

由于 `Yii::$app->params` 是一个 `ArrayAccess` 对象，你可以**临时替换它**，以便在赋值时触发日志：

即在入口文件中，注入如下代码：

```php
// 以上是原有的代码

class ParamsProxy implements \ArrayAccess
{
    private $params = [];

    public function offsetExists($offset): bool
    {
        return isset($this->params[$offset]);
    }

    public function offsetGet($offset)
    {
        return $this->params[$offset] ?? null;
    }

    public function offsetSet($offset, $value)
    {
        // 这是触发日志
        Yii::info("Params[$offset] was set from: " . json_encode(debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS, 3)), 'params');
        $this->params[$offset] = $value;
    }

    public function offsetUnset($offset)
    {
        unset($this->params[$offset]);
    }
}
// 临时替换发生在这里
$config['params'] = new ParamsProxy();

// 原有入口代码
(new yii\web\Application($config))->run();
```

访问任意页面，触发 `Yii::$app->params` 的赋值。

在日志中查看，搜索 `Params[customizeCategory] ` ，如下：

```text
Params[customizeCategory] was set from: [
{"file":"D:\\laragon\\www\\api\\vendor\\yiisoft\\yii2\\helpers\\BaseArrayHelper.php","line":140,"function":"offsetSet","class":"ParamsProxy","type":"->"},
{"file":"D:\\laragon\\www\\api\\tesoon\\base\\Controller.php","line":43,"function":"merge","class":"yii\\helpers\\BaseArrayHelper","type":"::"},
{"file":"D:\\laragon\\www\\api\\vendor\\yiisoft\\yii2\\base\\BaseObject.php","line":109,"function":"init","class":"tesoon\\base\\Controller","type":"->"}
]
```

这时就找到了赋值的位置。

原来是 `base\\Controller.php  ` 文件中，  `merge` 了一个数据表的值。

