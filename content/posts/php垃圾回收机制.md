+++
title = "PHP 垃圾回收机制"
date = 2021-11-04T19:11:06+08:00
draft = false
author = 'vdong'
categories = ['技术']
tags = ['php']
+++
## 变量的存储结构

众所周知，php 是用 c 语言写的，所以其变量存储也依赖于 c 语言。php 的变量的内部是使用一种 zval 的数据结构来保存的。

```c
// php源码 Zend/zend.h 文件中
typedef struct _zval_struct zval;
// ...
struct _zval_struct {
    zvalue_value value;     /* 变量的值 */
    zend_uint refcount__gc; /* 指向该变量容器的变量(也称符号即symbol)个数 */
    zend_uchar type;    /* 变量的类型 */
    zend_uchar is_ref__gc; /* 是否是引用 默认 false */ 
};
```

refcount__gc 为 0 表示没有变量指向这个内存容器， 就可以被释放。

```php
// refcount__gc 举例

$a = 1; // 内存中存在 一个 zval 结构 { value:1; type:integer; refcount__gc:1; is_ref__gc:false } ps: zval 结构的值是 zvalue_value（数据结构） 的，这里说 1, 不准确

$b = $a; // zval 中的 refcount__gc 就加 1， 为 { value:1; type:integer; refcount__gc:2; is_ref__gc:false }

unset($b) // zval 中的 refcount__gc 就减 1，为 { value:1; type:integer; refcount__gc:1; is_ref__gc:false }
```

## 顽固垃圾的产生

简单变量 refcount__gc = 0 时，直接释放 zval 内存，没什么问题。

```php
<?php
$a = array( 'one' );
$a[] =& $a;
xdebug_debug_zval( 'a' );
```

以上将输出：

```
a: (refcount=2, is_ref=1)=array (
   0 => (refcount=1, is_ref=0)='one',
   1 => (refcount=2, is_ref=1)=...
)
```

此时 `unset($a)` ， 引用情况将变成：

```
(refcount=1, is_ref=1)=array (
   0 => (refcount=1, is_ref=0)='one',
   1 => (refcount=1, is_ref=1)=...
)
```

尽管 $a 已经被 unset， 但是内存中的数据还在，refcount=1， 不能被清理。

## 清理顽固垃圾

php 5.3 引入了新的垃圾清理算法来清理以上垃圾。

具体参考 [https://www.php.net/manual/zh/features.gc.collecting-cycles.php](https://www.php.net/manual/zh/features.gc.collecting-cycles.php)

可以这么理解：

对于一个包含环形引用的数组，对数组中包含的每个元素的 zval 进行减1操作，之后如果发现数组自身的 zval 的 refcount 变成了0，那么可以判断这个数组是一个垃圾，对象也类似。

## 配置

php 中 GC 默认开启，配置项是 ini 文件 zend.enable_gc 项 。也可以通过 gc_enable() 和 gc_disable()函数来开启和关闭GC。

垃圾分析算法将在节点缓冲区(roots buffer)满了之后启动。缓冲区默认可以放10,000个节点，当然你也可以通过修改Zend/zend_gc.c中的_GC_ROOT_BUFFER_MAX_ENTRIES_ 来改变这个数值，需要重新编译链接PHP。你也可以通过调用gc_collect_cycles()在节点缓冲区未满的情况下强制执行垃圾分析算法。
