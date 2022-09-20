+++
title = "Shell编程"
date = 2022-09-20T14:37:32+08:00
draft = false
author = 'vdong'
categories = ['技术']
tags = ['unix','shell']

+++

## 字符串

```shell
# 单引号：原样， 双引号：会解析变量
a=1 # = 两边不加空格
str_1='a=$a' # a=$a
str_2="a=$a" # a=1

str_3=`pwd` # `(反引号)内的字符串被当作shell命令，执行完结果返回给 str_3 

# 字符串长度
s='abcd'
len=${#s} # 等价于 ${#s[0]}

# 截取 从第1个字符开始，截取3个字符，索引从0开始
sub_s=${s:1:3} 
```

## 获取当前脚本的绝对路径

```shell
BASE_DIR=$(cd $(dirname $0) && pwd)
```

## 获取配置文件内容

```shell
#  config.ini 内容如下：
#   [mysql]
#   host="1.1.1.3"
#   [redis]
#   host="1.1.1.4"
#   db="7"
# 获取配置信息
function getconf(){
    Section=$1; Item=$2
    _readIni=`awk -F '=' '/\['$Section'\]/{a=1}a==1&&$1~/'$Item'/{print $2;exit}' $BASE_DIR/config.ini`
    echo ${_readIni}
}

mysql_ip=$(getconf mysql host) # mysql_ip的值为：1.1.1.3
```

## 显示帮助文档

```shell
#!/bin/sh
###
### my-script — start or end data analysis server
###
### Usage:
###   my-script <command>
###
### Command:
###   up        Start data analysis server.
###   down      End data analysis server.
###   -h        Show this message.

function help() {
    sed -rn 's/^### ?//;T;p' "$0"
}

if [[ $# == 0 ]] || [[ "$1" == "-h" ]]; then
    help
    exit 1
fi
```

## shell 参数

- $# 参数个数
- $0 执行文件名（含路径）$1.. n 第几个参数
- $$ 当前进程 id
- $* 和 $@  引用所有参数，区别 在双引号中，假设在脚本运行时写了三个参数 1、2、3，，则 " * " 等价于 "1 2 3"（传递了一个参数），而 "@" 等价于 "1" "2" "3"（传递了三个参数）
- $? 显示最后命令的退出状态 0 表示没有错误，其余表示有错误

