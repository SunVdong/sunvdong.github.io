+++
title = "BitMap"
date = 2022-10-11T10:17:05+08:00
draft = false
author = 'vdong'
categories = ['技术']
tags = ['算法', 'bitmap']

+++

## BitMap原理

使用一个 bit 位来存储某种状态（比如签到与否，是否存在等）。

如记录 0-7 这几个数字中哪些存在，只需要 8 个 bit 位。0 表示数字不存在，1 表示数字存在，1，2，3，5 这几个数存在， 存储如下：

| 代表数字  |  7   |  6   |  5   |  4   |  3   |  2   |  1   |  0   |
| :-------: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| bit位表示 |  0   |  0   |  1   |  0   |  1   |  1   |  1   |  0   |

例如：记录一个英文句子（小写）中是否存在 `a` - `z` 所有字母：

```go
// 方法1： 使用 hash 存储
var hash = map[rune]bool{
    'a':false,
    'b':false,
    // ...
    'z':false,
}

// 方法2：使用数组，使用下标表示字母
var arr [26]bool

// 方法3：使用 int32 的低 26 位存储，每位 0 表示不存在，1 表示存在，从低到高依次表示 a-z
var bit int32
bit |= 1<<0 // 存储 a 存在
bit |= 1<<1 // 存储 b 存在
bit == 0x3ffffff // 判断 a-z 全部存在, 0x3ffffff 转为 2 进制 低 26 位全部为 1   

// 内存占用情况
fmt.Printf("%d\n", unsafe.Sizeof(hash)) // 8 字节
fmt.Printf("%d\n", unsafe.Sizeof(arr)) // 26 字节
fmt.Printf("%d\n", unsafe.Sizeof(bit)) // 4 字节
```

## 示例对比

[题目](https://github.com/SunVdong/Exercism/blob/main/go/sum-of-multiples/README.md)

```go
//  hashmap 方式
package summultiples
func SumMultiples(limit int, divisors ...int) int {
	sum := 0
	set := make(map[int]bool)
	for _, d := range divisors {
		if d == 0 {
			continue
		}
		for m := d; m < limit; m += d {
			if _, ok := set[m]; !ok {
				sum += m
				set[m] = true
			}
		}
	}
	return sum
}
// 性能测试结果
// BenchmarkSumMultiples-12            1593            700847 ns/op          487055 B/op        451 allocs/op

// bitmap 方式
func SumMultiples(limit int, divisors ...int) (sum int) {
	bits := make([]byte, (limit>>3)+1)

	for _, divisor := range divisors {
		if divisor == 0 {
			continue
		}
		for i := 1; ; i++ {
			num := divisor * i
			if num >= limit {
				break
			}

			index := num >> 3
			pos := num & 0x07

			if bits[index]&(1<<pos) == 0 {
				sum += num
				bits[index] |= 1 << pos
			}
		}
	}

	return
}
// 性能测试结果
// BenchmarkSumMultiples-12           66572             17749 ns/op            4040 B/op         16 allocs/op
```

## 缺点

（1）数据碰撞。比如将字符串映射到 BitMap 的时候会有碰撞的问题，那就可以考虑用 Bloom Filter 来解决，Bloom Filter 使用多个 Hash 函数来减少冲突的概率。

（2）数据稀疏。又比如要存入(10,8887983,93452134)这三个数据，我们需要建立一个 99999999 长度的 BitMap ，但是实际上只存了3个数据，这时候就有很大的空间浪费，碰到这种问题的话，可以通过引入 Roaring BitMap 来解决。



## 关于 Roaring BitMap

后续再补充，先放两个链接吧

[一文读懂比BitMap有更好性能的Roaring Bitmap](https://www.modb.pro/db/218968)

[不深入而浅出 Roaring Bitmaps 的基本原理](https://cloud.tencent.com/developer/article/1136054)


## 参考链接

> [go bitmap 实现](https://studygolang.com/articles/18575)
>
> [bitmap原理及应用](https://www.cnblogs.com/dragonsuc/p/10993938.html)