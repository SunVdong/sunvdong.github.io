+++
title = "正则相关"
date = 2022-09-22T22:22:58+08:00
draft = false
author = 'vdong'
categories = ['技术']
tags = ['正则']

+++

## 语法一览图

[维基百科](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)

[截图地址](https://cdn.nlark.com/yuque/0/2022/jpeg/215704/1663857012372-126b1a3e-c3c0-420e-b253-ff013e13f76d.jpeg)

## 补充说明
### 分组

非捕获组

```sh
`What is -?\d+(?: (?:plus|minus|divided by|multiplied by) -?\d+)*\?`
```

上面的正则表达式可以匹配

"What is 5?"

"What is -10?"

"What is 4 plus 5?"

"What is 3 minus 2 divided by 1?"

"What is -2 multiplied by -4 divided by 2 plus 6?"

## 测试网站
[https://regex101.com/](https://regex101.com/)

## 练习网站
[https://alf.nu/RegexGolf](https://alf.nu/RegexGolf)

## 正则可视化
[https://regex-vis.com/](https://regex-vis.com/)

[https://jex.im/regulex/#!flags=&re=%5E(a%7Cb)*%3F%24](https://jex.im/regulex/#!flags=&re=%5E(a%7Cb)*%3F%24)

[https://regexper.com/](https://regexper.com/)
