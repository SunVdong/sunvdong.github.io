+++
title = "Unix 过滤器"
date = 2021-11-04T19:11:06+08:00
draft = false
author = 'vdong'
categories = ['技术']
tags = ['unix', 'linux']
+++

过滤器可以用于管道，通过组合，解决实际问题，优雅而强大。

常用过滤器

## cat

```shell
cat name.txt mobile.txt > data # 合并文件

# -n 显示行号
```

## split ：拆分文件

```shell
split [-d] [-a num] [-l lines] [file [prefix]]

split data # 默认每个文件1000行 命名 xaa xab ...

# -l 100 : 指定每个文件行数
# -d ：改命名为 00 01 02
# -a 2 ： 命名改为 3 位 aaa aab / 001 002
```

## tac：反转文本行的顺序

和 cat 类似， 写入标准输出前将文本行反转，适用于日志文件等。

## rev：反转字符

```shell
rev data

# data 内容： 
# 1234
# abcd
# 结果如下：
# dcba
# 4321

rev data | tac # 倒背如流
```

## head / tail ：从数据开头和末尾选择数据行

```shell
head [-n lines]
tail [-n lines]
# 默认 10 行数据
```

## 列操作

### colrm ：删除数据列

```shell
colrm [startcol [endcol]]
# 列编号 从 1 开始

# 把 students_old 文件删除 14 列到 20 列后 保存到 student_new 文件中  
colrm 14 20 < students_old > students_new
```

### cut ： 抽取数据列 （与 colrm 相反）

```shell
cut -c list [file ...]

# 抽取data中的 14-30 和 42-49 列
cut -c 14-30,42-49 data 

# 结合其他命令
who | cut -c 1-8 | sort | uniq -c

# 定界符分割，如下形式的数据也可以抽取
张三:男:23岁
王麻子:女:105岁

# 抽取按照 ':' 分割后的， 第 1 和第 3 列数据
cut -f 1,3 -d ':' data # 可以删除选项的空格 如 cut -f1,3 -d':' data

# ps 如果一行没有定界符，默认返回整行，如果想抛弃这样的行，可以使用 -s
```

### paste 组合数据列

```shell
paste [-d char...] [file ...]

# 合并三个文件
paste file1 file2 file3

# 使用定界符 | 
paste -d '|' file1 file2

# 定界符轮换使用
paste -d '|&' file1 file2
```

## 比较文件

### cmp ：逐字节比较

```shell
cmp file1 file2

# 相同的话就 不做任何处理
```

### comm ： 比较有序文本文件

```shell
comm [-123] file1 file2

# 第一列 仅输出第一个文件有的行
# 第二列 仅输出第二个文件有的行
# 第三列 输出两个文件都有的行
# -1 -2 -3 表示抑制某一列的输入
```

### diff  / sdiff： 比较无序文件

```shell
diff file1 file2
# 默认 有指示符 （c, d,  a）和行号构成
# c 代表改变 d 代表删除 a 代表增加
# 中间 ------- 分割

# 文件2 新增两行
0a1,2
> 沁园春·雪
>

# 第5 第7行不一致
5c7
< 须晴日5，看红装素裹，分外妖娆6。
---
> 须晴日，看红装素裹，分外妖娆。

# 文件2 删除了一行
10d11
< 俱往矣，数风流人物，还看今朝。

# 选项
# -c 和 -u 可以显示 2 行上下文信息 
# -C3 -U3 可以显示 3 行上下文信息

# 并列比较 等价的
 diff -y file1 file2 
 sdiff file1 file2
 
 # 选项 
 -l 相同的行只显示左侧的
 -s 不显示相同的行
 -w 50 指定列宽度
```

## 统计和格式化

### nl ：创建行号

```shell
nl [-v start] [-i increment] [-b a] [-n ln|rn|rz] [file ...]
# start 起始号
# increment 增量
# 默认 nl 不对空行编号， -b a （all lines 所有行）
# ln=左对齐，没有前导 0
# rn=右对齐，没有前导 0
# rz=右对齐，有前导 0
```

### wc : 单词统计

```shell
wc [-clLw] [file ...]
# wc 默认输出三个数字：行数（line），单词数（word），字符数（char），LWC （look at Women Carefully）
# -clw 显示指定列 
# -L 显示最长的行
```

### expand: 将制表符转换成空格

```shell
expand [-i] [-t size | -t list] [file...]
# -i 只转换开头的制表符
# -t size 默认size等于8 ，每8个位置一个制表符
# -t list 指定制表符位于特定的位置，0指第一个位置，如 expand -t 7,15,21 data
```

### unexpand : 将空格转换成制表符

```shell
unexpand [-a] [-t size | -t list] [file ...]
# 默认只替换开头的空格
# -a 标识替换所有空格
# -t 的参数和 expand 一致
```

### fold: 长行分成短行

```shell
fold [-s] [-w width] [file ...]
# 默认 80 字符每行，
# -w width 可以重新指定宽度
# -s 不从中间分割单词
```

### fmt：将段落中的各行连接起来

```shell
fmt [-su] [-w width] [file...]
# -w width 行宽
# -u 统一间距，单词间有多个空格，减少，只保留一个
# -s 仅拆分长行，而不连接短行
```

## 选取、排列、组合与变换

### grep：选取包含特定模式的行

```shell
grep [-cilLnrsvwx] pattern [file...]

# -c 统计数量, ls -F 会在子目录会添加/字符，以下可以统计反斜线的数量 即子目录的数量
ls -F /etc | grep -c "/"
# 统计总数
ls /etc | wc -l

# -i 忽略大小写
# -n 显示行号
grep -in pizza food-list.md # 忽略大小写查找出含 pizza 的行并显示行号

# -l 不显示包含的行，而显示包含的文件名 查找哪个班有小明同学
grep -il xiaoming class1_names class2_names class3_names # output class1_names
# -L 与 -l相反，显示不包含的文件
grep -iL xiaoming class1_names class2_names class3_names # output class2_names class3_names

# -w 单词完整匹配，
grep now data # 会匹配 now，也会匹配 know 
grep -w now # 只匹配 now 

# -v 取反，选取不包含的模式的所有行
grep -v done todo_list # 列出所有未完成的项目
grep -cv done todo_list # 统计为完成的项目数目

# -x 查找占用整行的
grep -x "hello world" data # 返回 由 'hello world' 构成的行

# -r 递归搜索目录树
grep -r initialize admin # 搜索admin目录及子目录文件中是否有单词 “inttialize” 

# -s 抑制错误显示
```

### look：查找以特定模式开头的所有单词

```shell
look [-df] pattern file

# look 搜索以字母顺序排列的数据，并以特定的模式开头的行
# 因为 look 使用二分法，搜索数据，要求同时获取所有数据，故只能再管道的开头使用数据，不能再管道中间使用
# -d 只考虑字母和数字
# -f 忽略大小写
```

### sort : 排序数据

```shell
sort [-dfnru] [-o outfile] [infile...]
# 排序数据和查看数据是否排序

# 排序后保存到原文件时，这是错误的 重定向输出时， shell 运行命令前会清空输出的文件，（除非设置了变量 noclobber ）
sort names > names # wrong
sort -o names names # right

# -d 只查看字母、数字和空白符，如果数据中有妨碍排序的标点时，使用此选项
# -f 忽略大小写
# -n 识别开头的数字，按数字大小排序
# -r 倒序
# -u 唯一，相同的行，只留下一行

sort -c[u] [file]
# -c 检查数据是否有序，无序输出，开始无序的行，有序无消息（没有消息就好的消息）
# -u 配合-c使用，可以确保数据是有序， 唯一

# 排序规则：取决于你系统的字符组织方式
# 早期使用 ASCII 顺序， （SNUL） 空格 数字 大写字母 小写字母
# 现在 使用环境变量 LC_COLLATE 指定使用哪一种排序  c 是 ASCII 
locale # 可以查看所有区域设置的当前值
locale -a # 查看自己的系统支持哪些区域设置
```

### uniq:  查找重复行

uniq 可以做四个不同的任务


1. 消除重复行 `uniq`
1. 选取重复行 `uniq -d`
1. 选取唯一行 `uniq -u`
1. 统计重复行数量 `uniq -c`

```shell
uniq [-cdu] [infile] [outfile]

# 输入必须是有序的，即重复行是连续的
# -d 只查看重复行
# -u 只查看唯一行
# -c 统计唯一行的数量
```

### jion: 基于特定字段的值将两个有序文件组合在一起

```shell
join [-i] [-a1|-v1] [-a2|v2] [-1 field1] [-2 field2] file1 file2

# eg: 
cat file1
111 张三
222 李四
333 王五
999 孙悟空 

cat file2 
111 10086
222 10010
333 110
444 120
555 119

# join file1 file2
111 张三 10086
222 李四 10010
333 王五 110

# join -a1 file1 file2
111 张三 10086
222 李四 10010
333 王五 110
999 孙悟空

# join -v1 file1 file2
999 孙悟空

# 如上， join 关联类似于关系数据库的的join，
# 没-a选项时 内关联 inner join ，只输出关联字段匹配的行 
# -a1 -a2 时还输出关联字段不匹配的行
# -v1 -v2 只查看不匹配的行
# -i 忽略大小写

# join 默认使用的是 文件的第一个字段 可以使用 -1 -2 指定不同的字段
# 特别注意： join 使用的有序数据
```

### tsort: 以偏序创建全序

```shell
tsort [file]

# 偏序：只指定了一部分活动的顺序
# file 中的每一行必须包含一堆空白符分割的字符串 每个字符串代表一个偏序
# 最终结果生成一个完整序列
```

### strings: 二进制文件中搜索

### tr: 转换字符

1. 转换字符 ：小写转大写，制表符转空格
1. 挤压字符：连续数字 替换为 "X" ,多个空格转换为一个
1. 删除指定字符：删除所有指标符

```shell
tr [-cds] [set1 [set2]]

# 只能从标准输入接收数据，不能从文件读取，如果要从文件读取请重定向
tr a A < old > new # old 文件中的 a 替换成 A 了, 并写入 new 文件
tr abc ABC < old > new # abc 替换 ABC

# 以下情况，第二组最后一个字符是重复的，等价
tr abcd Ax 
tr abcd Axxxx 

# 特殊字符需要引用或转义 
tr ':;?' \. # :;? 均替换成了 .

# 支持范围
tr A-Z a-z 
tr 0-9 A-J

# 缩写 (预定义字符)
tr [:upper:] [:lower:] # 大写转小写
tr [:diggit:] A-J 

# 不可显示的字符 (转义 or ASCII码的值)
tr '\r' '\n' < macfile > unixfile # 回车符转换为换行符
tr '\015' '\012' < macfile > unixfile
tr '\t' ' ' # 制表符转换为空格

# ********* 挤压 *********
tr -s [:digit:] X # 数字转换成 x
tr -s ' ' ' ' # 挤压空格

# **** 删除 ****
tr -d '()' 

# **** 补集 *****
tr -c ' \n' x < olddata > newdata # hello world ==> xxxxx xxxxx

eg: 统计两个文件的单词数
cat greek roman | tr -cs [:alpha:]\' "\n" | sort -fu | wc -l
```

### sed: 非交互式文本编辑(提前设计命令，命令发给程序，自动执行)(流编辑器)

```shell
sed [-i] command | -e command ... [f]

-i # 直接修改原文件
# 命令 
# a 行后插入， 多行的话 末尾"\"
# i 行前插入， 多行 "\"
# c 替换 
# d 删除
# p 打印 
# s 替换 可用正则表达式
   

# 查看某一行
sed -n '1003,1p' data.txt

# 修改某一行
sed -i '1003c new line content' data.txt

# 将 11 修改为 12，g 代表一行的所有 
sed -i 's/11/12/g'  xxx.log

# 每行 开头/末尾 追加 aa
sed -i 's/^/aa/g' xxx.log
sed -i 's/$/aa/g' xxx.log

# 修改每行第2个 匹配的11
sed  -i 's/11/12/2' xxx.log

# 修改每行弟2个以及以后的 匹配的11 
sed -i 's/11/12/2g' xxx.log

# & 符号代表的是你前面的匹配的模式 将每行的第一个 hello 替换为 (hello)
sed 's/hello/(&)/' xxx.log
```