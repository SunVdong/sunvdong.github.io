+++
title = 'Unix文件,别名等相关'
date = 2022-05-05T11:32:14+08:00
draft = false
author = 'vdong'
categories = ['技术']
tags = ['折腾','unix','linux']

+++

## 排序、区域设置

unix 的默认排序方式取决于你使用的区域设置，如：现在有A、a、B、b、C、c 几个文件，执行 `ls` 列举文件：

如果使用 C 区域设置，你会得到：`A B C a b c` (基于 ASCII 码)；

如果使用 en_US, 你将会得到： `a A b B c C` 

`export LC_COLLATE=C`

## 目录栈相关

`pushd`  、`popd`、`dirs` 三个命令将维护一个目录栈，我们可以通过调整栈的顺序实现快速的切换工作目录。

| 命令               | 动作                                    |
| :----------------- | :-------------------------------------- |
| dirs               | 显示名称：home 显示为 ~                 |
| dirs -l            | 显示完整名称                            |
| dirs -v            | 显示名称：每行一个，并且有数字标识      |
| pushd  *directory* | 改变工作目录：将 *directory* 压入到栈中 |
| pushd *+n*         | 改变工作目录：将 *#n* 移动到栈顶        |
| popd               | 改变工作目录：弹出栈顶                  |
| popd *+n*          | 从栈中移除目录 *#n*                     |
| dirs -c            | 除当前工作目录外，移除栈中的全部目录    |

```shell
# 定义别名快速操作
alias d='dirs -v'
alias p=pushd
```

## 检查文件类型和颜色

```shell
alias ls='ls -F --color=auto'

alias la='ls -a'
alias ldot='ls -d .??*'
```

## 防止误删文件

```shell
# 先 ls 检查删除的文件，再通过fc命令替换为rm
alias del='fc -s ls=rm'
```

## 利用历史记录

```shell
alias a=alias
alias info='date; who'
alias h="fc -l"
alias r="fc -s"

# 使用
# h 显示记录
# r #number
# r #name old=new
```

## 掌握磁盘空间情况

```shell
# -s(size，大小 单位 KB)
ls -s   

# -h(human-readable, 适合人类阅读)
ls -sh

# -a(all,全部)
# -c(count,统计) 末位显示总量
# -s(sum, 总和)
# du(disk usage,磁盘使用)
du [-achs] [name...] 

# df(disk free-space, 磁盘可用空间) 
df [-h]
```

### 块和分配单元：dumpe2fs 

文件系统中，空间以固定大小的组块进行分配，即 **块（block）**，为文件所分配的最小磁盘空间数量。所以在一个块大小为1KB（1024字节）的文件系统中，仅仅包含1字节的文件也要占用一个完整的块，1025字节的数据文件需要两个块。

出于效率的考虑，当文件写入磁盘或其他存储介质上的时候，磁盘存储空间也是以固定大小的组块分配，即 **分配单元（allocationuniut）或 簇（cluster）**。分配单元的大小取决于文件系统和存储设备。

例如，我的 linux 系统上，块大小为 1 KB，但是磁盘分配单元  8KB。因此，一个只有一个字节的文件实际上要占用 8KB 的磁盘空间。

查看自己系统块大小和分配单元：

```shell
# 创建一个很小的文件
echo a > temp

# 查看文件包含的数据的数量 文件大小在日期前面, 单位字节  2 个字节
ls -l temp 

# 查看文件占用的存储空间  4K 所以分配单元是 4K 
du -h temp


# 查看块大小 单位：字节 
sudo dumpe2fs /dev/vda1 | grep "Block size"
```

`dumpe2fs` 命令是用于显示 ext2、ext3 和 ext4 文件系统的详细信息的命令，对于 xfs 文件系统，应该使用 `xfs_info` 命令来查看其详细信息，包括块大小等。可以执行以下命令来查看 `/dev/sda1` 分区上的 xfs 文件系统的块大小。

```shell
# 查看所有块设备
lsblk

#查经分区或硬盘文件类型
blkid /dev/sda1

# /dev/sda1 分区的文件系统信息的输出结果
sudo xfs_info /dev/sda1

# 结果如下： bsize 每个块的字节数 ；isize=512: 每个inode所占用的字节数 等
meta-data=/dev/sda1              isize=512    agcount=4, agsize=65536 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=262144, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```



## 通配符

| 符号                 | 含义                     |
| -------------------- | ------------------------ |
| *                    | 任何0个或多个字符        |
| ？                   | 任何单个字符             |
| [*list*]             | *list* 中的任何字符      |
| [^*list*]            | 不在 *list* 中的任何字符 |
| {*string1\|string2*} | 其中一个指定字符串       |

### 预定义的字符

| 类          | 含义               | 类似于      |
| ----------- | ------------------ | ----------- |
| [[:lower:]] | 小写字母           | [a-z]       |
| [[:upper:]] | 大写字母           | [A-Z]       |
| [[:digit:]] | 数字               | [0-9]       |
| [[:alnum:]] | 大写小写字母与数字 | [A-Za-z0-9] |
| [[:alpha:]] | 大写和小写字母     | [A-Za-z]    |

## Unix 为新文件指定权限的方式：umask

Unix 创建文件时，将根据文件类型为文件指定以下几种模式：

- 666: 不可执行的普通文件
- 777：可执行的普通文件
- 777： 目录

在这以初始模式上，Unix再减去用户掩码（user mask）。设置掩码的命令为：`umask [mode]`

```shell
# 限制其他人写的权限
umask 022 

# 显示当前的掩码
umask
```

### 清空文件内容

当删除文件时，文件所使用的实际磁盘空间还没被清除。文件系统只是将这部分磁盘空间标识为可重用。最终旧数据会被新数据覆盖。但是时间不确定，且有一些特殊的恢复工具可以恢复数据。

shred 程序的目的就是多次覆盖硬盘上已有的数据。

```shell
# -f(force,强制)
# -u 覆盖完删除文件
# -v(verbose, 详细)
# -z 填充零，默认是随机数据 
shred -fuvz datafile
```

## 链接的概念

### `stat`、`ls -i` i 节点（i-node）

当 unix 创建文件时，unix 做了两件事。

1. Unix 在存储设备上保留一块空间用来存储数据。
2. Unix 创建一个**索引节点（index node）**或 **i 节点（i-node）**的结构。用来存放文件的基本信息。

查看文件的 i 节点内容： `stat filename`

文件系统将所有的 i 节点存放在一个大表中，称为 **i 节点表（inode table）**，每个 i 节点由所谓的索引号或 i 节点号表示。

查看文件的节点号：`ls -i filename`

处理目录时候，就好像目录包含文件一样，其实，目录并不包含文件，目录只包含文件的名称及文件的 i 节点号。因此目录的内容相当小：只有一列名称，每个名称对应一个 i 节点号。

文件名和 i 节点之间的连接称为 **链接**。

### 多重链接

Unix 文件系统允许多重链接。一个文件可以有不止一个名称。文件的唯一标识是 i 节点号，不是名称，所有多个名称可以引用到同一个 i 节点号。 

链接的基本思想是同一个文件可能拥有不同的含义（取决于文件使用的环境）。并且 Unix 平等的对待所有的链接。

### 创建新链接: ln

 ```shell
 # file 是已有的普通文件的名称，newname是希望赋予链接的名称
 ln file newname
 
 # 为一个或多个普通文件创建链接，并放到指定的目录中
 ln file... directory
 ```

`rm` 和 `rmdir` 其实是移除链接，只移除了文件名和 i 节点号之间的连接，如果文件没有链接了，Unix 会删除该文件。

### 符号链接：ln -s

以上链接有两个限制：1. 不能为目录创建链接，2. 不能为不同的文件系统创建链接。

如果要实现以上需求，需要使用 **符号链接（symbol link 或 symlink）** 。

符号链接包含的不是文件的 i 节点号，而是文件的路径名。当访问符号链接时候，Unix 借助路径名查找文件。类似于 Windows 的快捷方式。

``` shell
ls -l /bin/sh

# 输出  /bin/sh 是一个指向 /bin/bash文件的符号链接
lrwxrwxrwx 1 root root 4 Aug  7  2020 /bin/sh -> bash
```

为了区分两种链接：

- 常规的链接 :  **硬链接（hard link）** 
  - ls -l 可以查看硬链接数量，rm 可以移除硬链接

- 符号链接  :  **软链接（soft link）**
  - 如果一个文件存在符号链接，删除文件，符号链接不会被删除，使用时则会报错。
  - 当一个目录使用符号链接时，cd 和 pwd 既可以将符号链接视为一个实体，也可以将链接作为真实目录的一个跳板。 `-L` （logical， 逻辑）、`-P`（physical，物理）选项指定，默认 `-L`

