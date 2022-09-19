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

 Go 语言中，函数传参是**传值**，只是针对了数据结构中固定部分传值。如：字符串或切片传参时，只是复制了对应数据结构中的指针、字符串长度、切片容量等值，并不包含指针指向的内容。可以将字符串和切片类型的参数转换为 `reflect.StringHeader ` `reflect.SliceHeader`的结构体，可以更好的理解传值的含义：

```go
func twice(x []int){
    for i := range x{
        x[i] *= 2
    }
}

type IntSliceHeader struct {
    Data []int
    Len int
    Cap int
}

func twice(x IntSliceHeader){
    for i:= 0; i<x.Len; i++{
        x.Data[i] *= 2
    }
}
```

如上，切片中的**底层数组部分**是通过**隐式指针传递**(**指针**本身依然是**传值**的，但是指针指向的却是同一份的内存数据)，所以被调用函数是可以通过**修改指针指向的数据**（指针本身的指向并没有发生变化），从而达到修改掉调用参数切片中数据的效果。

Go 语言支持递归调用，且调用深度逻辑上没有限制，函数调用栈不会出现溢出错误，这是因为 Go 语言运行时会根据需要动态调整函数栈的大小。栈最大可达到GB级。

Go 1.4 之前，动态栈采用的时分段式的动态栈，通俗说就是采用一个链表实现动态栈。链表节点位置不变，缺点: 跨节点地址位置不是一定是连续的，CPU缓存命中率低。

Go 1.4 之后改用连续的动态栈实现，通俗来说是采用一个类似动态数组来实现栈。缺点：当连续栈需要动态增长时候，需要重新分配内存空间并复制数据到新的空间，这导致了栈中之前变量的地址发生变化（Go 的运行时会自动更新这些变化）。但是这意味着 ，Go 语言中的指针不是固定不变的，不能随意将指针保存在数值变量中，不能将 Go 语言的地址随意保存到非 GC 的环境中（如：使用 CGO 时，C 语言不能长期持有 Go 与语言的地址）。

Go 语言隐藏了堆栈的细节，编译器和运行时帮我们做了还多工作，以下代码在 C/C++ 中不可行的，但在 Go 语言中并没问题：

```go
// 如果参数变量在栈上的话，函数返回之后栈变量就失效了，返回的地址自然也应该失效了
func f(x int) *int {
    return &x
}

// 内部虽然调用 new 函数创建了 *int 类型的指针对象，但是依然不知道它具体保存在哪里
func g() int {
    x := new(int)
    return *x
}
```

需要注意的是：不要假设变量在内存中的位置是固定不变的，指针随时可能会变化，特别是在你不期望它变化的时候。

## 方法

Go 语言方法关联到结构体上，在编译阶段完成静态绑定。

我们可以给任意自定义的结构体添加一个或多个方法。方法和结构体定义必须在同一个包里。

其实，方法是由函数演化而来，只是将函数的第一个对象参数移动到函数名前面而已。因此我们可以使用方法表达式的特性将方法还原为普通类型函数：

```go
// func ReadFile(f *File, offset int64, data []byte) int
var ReadFile = (*File).Read

// func CloseFile(f *File) error
var CloseFile = (*File).Close

f, _:= OpenFile("foo.txt")
ReadFile(f,0,data)
CloseFile(f)
```

对于有些场景，我们并不关心具体操作对象的类型，只要满足通用的 行为 就可以了。在Go语言中，我们可以通过闭包做参数绑定，从而消除上边函数参数类型的的限制：

```go
f, _:=OpenFile("foo.txt")

// 绑定了 f 对象
// func Close() error
var Close = func() err(){
    return (*File).Close(f)
}
// 绑定到了 f 对象
// func Read(offset int64, data []byte) int
var Read = func(offset int64, data []byte) int {
    return (*File).Read(f, offset, data)
}
// 文件处理
Read(0, data)
Close()

// 使用方法值简化以上问题：

// 方法值: 绑定到了 f 对象
// func Close() error
var Close = f.Close
// 方法值: 绑定到了 f 对象
// func Read(offset int64, data []byte) int
var Read = f.Read
// 文件处理
Read(0, data)
Close()
```

Go语言使用组合的方式支持继承。使用结构体中的匿名成员来实现继承。因此**继承来的方法的接收者参数依然是匿名成员自身，而不是当前变量**。

传统继承，子类的方法是运行时动态绑定到对象的，`this` 可能不是集类类型对应的对象，不确定性。

Go语言继承，子类方法是编译时静态绑定的，`this` 就是实现该方法的类型对象，确定性。

## 接口

Go 语言的接口满足隐式的鸭子类型。即：走路看着像鸭子，叫起来也像鸭子的，就可以当作鸭子。go语言中，一个对象只要看起来像是某个接口的实现，那么就可以把他作为该接口使用。

这种设计可以让你创建一个新的接口类型满足已经存在的具体类型却不用去破坏这些类型原有的定义。

Go语言的接口类型是延迟绑定的，可以实现虚函数的多态功能。

```go
// fmt.Fprintf 签名
func Fprintf(w io.Writer, format string, args ...interface{}) (int, error)

// io.Writer 是用于输出的接口
type io.Writer interface {
    Write(p []byte) (n int, err error)
}
```

我们可以定制自己的输出对象：

```go
type UpperWrite struct{
    io.Writer
}

func (p *UppperWriter) Write(data []byte) (n int, err error){
    return p.Writer.Write(bytes.ToUpper(data))
}
```

如果满足 `fmt.Stringer` 接口，则默认使用对象的 `String` 方法的返回结构打印：

```go
type UpperString string

func (s UpperString) String() string {
	return strings.ToUpper(string(s))
}
```

对于基础类型（非接口），go 不支持隐式的类型转换，例如我们无法将一个 `int` 类型直接赋值给 `int64`类型的变量。

对于接口类型，可以隐式转换，对象和接口，接口和接口都可以转换：

```go
var (
    a io.ReadCloser = (*os.File)(f) // 隐式转换, *os.File 满足 io.ReadCloser 接口
    b io.Reader     = a             // 隐式转换, io.ReadCloser 满足 io.Reader 接口
    c io.Closer     = a             // 隐式转换, io.ReadCloser 满足 io.Closer 接口
    d io.Reader     = c.(io.Reader) // 显式转换, io.Closer 不满足 io.Reader 接口
)
```

防止这种对象和接口太灵活的方式：

1. 包含特殊方法，来区分接口。(君子协定)

   ```go
   type runtime.Error interface {
       error
       // RuntimeError 方法 用于避免其它类型无意中适配了该接口
       RuntimeError()
   }
   
   type proto.Message interface {
       Reset()
       String() string
       // 用于避免其它类型无意中适配了该接口
       ProtoMessage()
   }
   ```

   这是可以伪造的。

2. 更严格一点，可以定义一个私有方法。

   ```go
   type testing.TB interface {
       Error(args ...interface{})
       Errorf(format string, args ...interface{})
       ...
   
       // A private method to prevent users implementing the
       // interface and so future additions to it will not
       // violate Go 1 compatibility.
       private()
   }
   ```

   这是有代价的：

   1. 这个接口只能内部使用。

   2. 这种防护也不是绝对的。（可以在结构体中嵌入匿名类型的成员来绕过）。

      我们在自己的 `TB` 结构体类型中重新实现了 `Fatal` 方法，然后通过将对象隐式转换为 `testing.TB` 接口类型（因为内嵌了匿名的 `testing.TB` 对象，因此是满足 `testing.TB` 接口的），然后通过 `testing.TB` 接口来调用我们自己的 `Fatal` 方法。

      ```go
      package main
      
      import (
          "fmt"
          "testing"
      )
      
      type TB struct {
          testing.TB
      }
      
      func (p *TB) Fatal(args ...interface{}) {
          fmt.Println("TB.Fatal disabled!")
      }
      
      func main() {
          var tb testing.TB = new(TB)
          tb.Fatal("Hello, playground")
      }
      ```

      这种通过嵌入匿名接口或嵌入匿名指针对象来实现继承的做法其实是一种纯虚继承，我们继承的只是接口指定的规范，真正的实现在运行的时候才被注入。比如，我们可以模拟实现一个gRPC的插件：

      ```go
      type grpcPlugin struct {
          *generator.Generator
      }
      
      func (p *grpcPlugin) Name() string { return "grpc" }
      
      func (p *grpcPlugin) Init(g *generator.Generator) {
          p.Generator = g
      }
      
      func (p *grpcPlugin) GenerateImports(file *generator.FileDescriptor) {
          if len(file.Service) == 0 {
              return
          }
      
          p.P(`import "google.golang.org/grpc"`)
          // ...
      }
      
      ```

      构建的对象必须满足 https://github.com/golang/protobuf/blob/master/protoc-gen-go/generator/generator.go 中 `Plugin` 接口。

      也就是说 `grpcPlugin` 类型的 `GenerateImports` 方法中使用的 `p.P(...)` 函数却是通过 `Init` 函数注入的 `generator.Generator` 对象实现（因为grpcPlugin 类型中比没有 P 方法，且只要一个成员）。

> https://chai2010.cn/advanced-go-programming-book/ch1-basic/ch1-04-func-method-interface.html

