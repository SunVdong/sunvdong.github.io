+++
title = "Function-Method-Interface-of-Golang"
date = 2022-09-09T15:22:58+08:00
draft = false
author = 'vdong'
categories = ['技术']
tags = ['golang']

+++

Go 语言中方法分为 具名和匿名，当匿名函数引用了外部变量时候就成了闭包函数。

方法是绑定到一个具体类型的特殊函数，方法依托类型。必须在编译时静态绑定。

接口定义了方法的集合，这些方法依托于运行时的对象（实现了接口中的方法），因此接口对应的方法是运行时动态绑定的。

Go 语言程序的初始化和执行化总是从 `main.main` 开始。如果 `main` 导入了其他包，则按顺序将他们包含进来，一直递归；包含时，先创建和初始化包的常量、变量，然后调用包中的 `init` 函数。如下：

![image-20220909154914868](/imgs/function-method-interface-of-golang/image-20220909154914868.png)

要注意的是，在 `main.main` 函数执行之前所有代码都运行在同一个 `Goroutine` 中，也是运行在程序的主系统线程中。如果某个 `init` 函数内部用 go 关键字启动了新的 `Goroutine` 的话，新的 `Goroutine` 和 `main.main` 函数是并发执行的。

## 函数

Go 语言中函数可以有**多个**返回值。

**参数**和**返回值**都是以**值**的形式和被调用者交换数据的。

函数支持**可变数量**的参数，可变数量的参数必须是最后一个。可变数量的参数其实是一个切片类型的参数。

```go
func Swap(a,b int) (int, int){
    return b,a
}

func Sum(a int, more ...int) int{
    for _,v :=range more{
        a += v
    }
    return a
}
```

函数的返回值也可以用名字：

```go
func Find(m map[int]int, key int) (value int, ok bool) {
    value, ok = m[key]
    return 
}
```

`derfer` 可以在 `reurn` 之后修改返回值：

```go
func Inc() (v int) {
    defer func(){ v++ } ()
    return 42
}
```

 

