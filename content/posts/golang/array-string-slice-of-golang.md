+++
title = "Golang 中的数组字符串和切片"
date = 2022-09-09T10:57:55+08:00
draft = false
author = 'vdong'
categories = ['技术']
tags = ['golang']

+++

## 数组

### 定义方式

```go
var a[3] int
var b = [...]int{1, 2, 3}

var c = [...]int{2:3, 1:2}  // 长度为3  {0,2,3}

// 混合以上方式
var d = [...]int{1,2,4:5,6} // {1, 2, 0, 0, 5, 6}
```

Go 语言中数组是值语义。一个数组变量即表示整个数组，它并不是隐式的指向第一个元素的指针（比如 C 语言的数组），而是一个完整的值。当一个数组变量被赋值或者被传递的时候，实际上会复制整个数组。如果数组较大的话，数组的赋值也会有较大的开销。为了避免复制数组带来的开销，可以传递一个指向数组的指针，但是数组指针并不是数组。

### 循环

```go
// 数组元素大小为 0 ， 所以没有内存占用
var times [5][0]int
for range times {
    fmt.Println("aaa")
}
```

### 多种数组

```go
// 函数数组 图像解码器
var decoder1 [2]func(io.Reader) (image.Image, error)
var decoder2 = [...]func (io.Reader) (image.Image,error) {
    png.Decode,
    jpeg.Decode,
}

// 接口数组
var unknown1 [2]interface {}
var unknown2 = [...]interface {}{ 123 , "你好" }

// 管道数组
var chanlist = [2]chan int{}
```

### 空数组

长度为 0 的数组部占用空间。可用于强调某种特有的类型。如管道的同步操作

```go
c1 := make(chan [0]int)
go func() {
    fmt.Println("c1")
    c1 <- [0]int{}
}()
<-c1
```

我们并不关心管道中传输数据的真实类型，其中管道接收和发送操作只是用于消息的同步。对于这种场景，我们用空数组来作为管道类型可以减少管道元素赋值时的开销。当然一般更倾向于用无类型的匿名结构体代替：

```go
c2 := make(chan struct{})
go func(){
    fmt.Println("c2")
    c2 <- struct{}{} // struct{} 部分是类型， {} 表示对应结构体的值
}()
<-c2 
```

## 字符串

字符串是一个不可改变的字节序列。go 源码要求 `UTF8` 编码。所以源码中的字符串字面量通常被解释为 `UTF8`编码的 `unicode` `rune` 序列。

## 切片

切片是一种简化版的动态数组。

```go
x := []int{2, 3, 5, 7, 11}
y := x[1:3]

// output: [2 3 5 7 11]  len(x): 5  cap(x): 5
fmt.Println(x, " len(x):", len(x), " cap(x):", cap(x))
// output: [3 5]  len(y): 2  cap(y): 4
fmt.Println(y, " len(y):", len(y), " cap(y):", cap(y))

```

### 定义方式

```go
var (
    a []int               // nil 切片, 和 nil 相等, 一般用来表示一个不存在的切片
    b = []int{}           // 空切片, 和 nil 不相等, 一般用来表示一个空的集合
    c = []int{1, 2, 3}    // 有 3 个元素的切片, len 和 cap 都为 3
    d = c[:2]             // 有 2 个元素的切片, len 为 2, cap 为 3 , [0,2)
    e = c[0:2:cap(c)]     // 有 2 个元素的切片, len 为 2, cap 为 3 , [0,2) 且指定 cap 为 c 的 cap  
    f = c[:0]             // 有 0 个元素的切片, len 为 0, cap 为 3
    g = make([]int, 3)    // 有 3 个元素的切片, len 和 cap 都为 3
    h = make([]int, 2, 3) // 有 2 个元素的切片, len 为 2, cap 为 3
    i = make([]int, 0, 3) // 有 0 个元素的切片, len 为 0, cap 为 3
)

```

### 添加切片元素

```go
var a []int
a = append(a, 1)               // 追加 1 个元素
a = append(a, 1, 2, 3)         // 追加多个元素, 手写解包方式
a = append(a, []int{1,2,3}...) // 追加 1 个切片, 切片需要解包
```

当容量不足时，append 会重新分配内存，导致巨大的内存分配和复制数据开销。

头部追加元素

```go
var a = []int{1,2,3}
a = append([]int{0}, a...)
a = append([]int{-3,-2,-1}, a...)
```

中间插入元素

```go
var a []int
a = append(a[:i], append([]int{x}, a[i:]...)...)     // 在第 i 个位置插入 x
a = append(a[:i], append([]int{1,2,3}, a[i:]...)...) // 在第 i 个位置插入切片


// 以上会创建中间的临时切片，使用 copy 和 append 会避免
a = append(a, 0)     // 切片扩展 1 个空间
copy(a[i+1:], a[i:]) // a[i:] 向后移动 1 个位置
a[i] = x             // 设置新添加的元素

a = append(a, x...)       // 为 x 切片扩展足够的空间
copy(a[i+len(x):], a[i:]) // a[i:] 向后移动 len(x) 个位置
copy(a[i:], x)            // 复制新添加的切片
```

### 删除切片元素

尾部删除

```go
a = []int{1,2,3}
a = a[:len(a)-1] // 删除尾部一个元素
a = a[:len(a)-n] // 删除尾部n个元素
```

删除开头

```go
a = []int{1, 2, 3}
a = a[1:] // 删除开头 1 个元素
a = a[N:] // 删除开头 N 个元素

a = []int{1, 2, 3}
a = append(a[:0], a[1:]...) // 删除开头 1 个元素
a = append(a[:0], a[N:]...) // 删除开头 N 个元素

a = []int{1, 2, 3}
a = a[:copy(a, a[1:])] // 删除开头 1 个元素
a = a[:copy(a, a[N:])] // 删除开头 N 个元素
```

删除中间

```go
a = []int{1, 2, 3, ...}

a = append(a[:i], a[i+1:]...) // 删除中间 1 个元素
a = append(a[:i], a[i+N:]...) // 删除中间 N 个元素

a = a[:i+copy(a[i:], a[i+1:])]  // 删除中间 1 个元素
a = a[:i+copy(a[i:], a[i+N:])]  // 删除中间 N 个元素
```

### 切片内存技巧

`len` 为 `0` 且 `cap` 不为 `0` 的切片非常有用，可以降低内存分配的次数。

```go
func TrimSpace(s []byte) []byte {
    b := s[:0]
    for _, x := range s {
        if x != ' ' {
            b = append(b, x)
        }
    }
    return b
}

func Filter(s []byte, fn func(x byte) bool) []byte {
    b := s[:0]
    for _, x := range s {
        if !fn(x) {
            b = append(b, x)
        }
    }
    return b
}
```

### 避免内存泄漏

切片操作并不会复制底层的数据。底层的数组会被保存在内存中，直到它不再被引用。但是有时候可能会因为一个小的内存引用而导致底层整个数组处于被使用的状态，这会延迟自动内存回收器对底层数组的回收。

例如，`FindPhoneNumber` 函数加载整个文件到内存，然后搜索第一个出现的电话号码，最后结果以切片方式返回。

```go
func FindPhoneNumber(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    return regexp.MustCompile("[0-9]+").Find(b)
}
```

这段代码返回的 `[]byte` 指向保存整个文件的数组。因为切片引用了整个原始数组，导致自动垃圾回收器不能及时释放底层数组的空间。一个小的需求可能导致需要长时间保存整个文件数据。这虽然这并不是传统意义上的内存泄漏，但是可能会拖慢系统的整体性能。

要修复这个问题，可以将感兴趣的数据复制到一个新的切片中（数据的传值是 Go 语言编程的一个哲学，虽然传值有一定的代价，但是换取的好处是切断了对原始数据的依赖）：

```go
func FindPhoneNumber(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    b = regexp.MustCompile("[0-9]+").Find(b)
    return append([]byte{}, b...)
}
```

指向指针的切片也可能遇到类似的问题：

```go
var a []*int{ ... }
a = a[:len(a)-1]    // 被删除的最后一个元素依然被引用, 可能导致 GC 操作被阻碍


var a []*int{ ... }
a[len(a)-1] = nil // GC 回收最后一个元素内存
a = a[:len(a)-1]  // 从切片删除最后一个元素
```

> 此为《go 语言高级编程》 读书笔记
