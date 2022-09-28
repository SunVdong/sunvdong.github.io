+++
title = "expect-自动交互脚本"
date = 2022-06-01T09:15:46+08:00
draft = false
author = 'vdong'
categories = ['技术']
tags = [ 'linux', 'unix','脚本']

+++

## scp 多个主机并填写密码

```shell
#!/usr/bin/expect

set filename [lindex $argv 0]
set dir_path [lindex $argv 1]

set IP_NOS_1 "192.168.122.11"
set IP_NOS_2 "192.168.122.12"
set IP_NOS_3 "192.168.122.13"
set IP_NOS_4 "192.168.122.14"
set IP_NOS_5 "192.168.122.15"

set username "root"
set passwd "root"
set timeout 200

if { $argc!=2 } {
        puts "WARNING ! srcipt needs two parameter !"
        puts "          Usage: (parameter 1: filename) filename you want to transfer"
        puts "          Usage: (parameter 2: path)     path you want to save to where."
        puts "this scripts will cp file into 5 kvm!"
        exit 0
}



for {set i 1} {$i < 6} {incr i} {
        switch $i {
        1 {set ip $IP_NOS_1}
        3 {set ip $IP_NOS_3}
        4 {set ip $IP_NOS_4}
        5 {set ip $IP_NOS_5}
        }
        puts "#---------------------copy file to kvm$i------------------#"
        spawn scp ./$filename $username@$ip:$dir_path
        expect {
              "(yes/no)?" { send "yes"; exp_continue }
              "assword:" { send "$passwd\r" }
                }
        expect "100%"
        sleep 1
}
```

## 参考网站

[expect范例](http://xstarcd.github.io/wiki/shell/expect.html)

[expect安装](https://www.cnblogs.com/hanby/p/15524774.html)

