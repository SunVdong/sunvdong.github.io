+++
title = "PHP 基础"
date = 2021-11-04T19:11:06+08:00
draft = false
author = 'vdong'
categories = ['技术']
tags = ['php']
+++

## php 假值 与 empty 和 is_null

1. 当 $a 是以下 值时候 `(boolean) $a`  为假 false：

- 未设置的变量
- var $a;  未初始化 $a 的值
- $a = null;
- false
- 0
- ''
- '0'
- []

2. 以上值 empty() 是均为 true 。
3. isset() 检测 一个变量设置了，且不为 null ，所以仅仅前 3 条 为 false 。
4. is_null() 正好与 isset() 相反， 仅仅 前 3 条 为 true 。
5. gettype() 前3条 ，均返回 null，即 类型 未知。

## php array_map 和 array_walk 的区别

相同点 都是利用回调函数对数组中每个元素进行操作。

|  | array_map(callable $callback, $$arr, ...$$arr):array | array_walk( array &$arr, callable $$callback [,$$userData]) |
| --- | --- | --- |
| 描述 | 对数组的每个元素应用回调函数 | 使用用户子定义函数对每个元素做回调处理 |
| 返回值 | 返回数组，如果回调函数没有返回值 返回 [] | 返回 bool ， 成功 true，否则 false |
| 参数顺序 | 先回调，再数组，再额外数组 | 先数组，再回调，再用户数据 |
| 回调函数参数 | 只有数组的 value， 且 个数 与 传入的数组一致，即可以转入多个数组 | 默认 value, key [可选用户数据], 如果要修改原数组的值以引用的方式 (&) 传第一个值 |

## php 的魔术方法

[__construct()](https://www.php.net/manual/zh/language.oop5.decon.php#object.construct)， [__destruct()](https://www.php.net/manual/zh/language.oop5.decon.php#object.destruct)，

[__call()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.call)， [__callStatic()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.callstatic)，

[__get()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.get)， [__set()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.set)， [__isset()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.isset)， [__unset()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.unset)，

[__sleep()](https://www.php.net/manual/zh/language.oop5.magic.php#object.sleep)， [__wakeup()](https://www.php.net/manual/zh/language.oop5.magic.php#object.wakeup)， [__serialize()](https://www.php.net/manual/zh/language.oop5.magic.php#object.serialize), [__unserialize()](https://www.php.net/manual/zh/language.oop5.magic.php#object.unserialize), // 序列化和反序列化

[__toString()](https://www.php.net/manual/zh/language.oop5.magic.php#object.tostring)，

[__invoke()](https://www.php.net/manual/zh/language.oop5.magic.php#object.invoke)， // 以函数方式调用一个对象时 $a= new A; $a(1); class A 中的 __invoke($val) 方法将被调用

[__set_state()](https://www.php.net/manual/zh/language.oop5.magic.php#object.set-state)，

[__clone()](https://www.php.net/manual/zh/language.oop5.cloning.php#object.clone)  // 深度复制时 调用

[__debugInfo()](https://www.php.net/manual/zh/language.oop5.magic.php#object.debuginfo)
