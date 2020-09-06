---
title: GO 语言程序设计读书笔记-接口值
date: 2020-09-06 13:10:17
categories: Go
tags: Go
---

从概念上来讲，一个接口类型的值（简称接口值）其实有两个部分：一个具体类型和该类型的一个值。二者称为接口的动态类型和动态值。比如下面声明一个接口变量 w 并赋值，那么 w 接口值可以用如下图表示：

![Alt text](/images/go-interface-value.png)

#### 接口的零值
接口的零值就是把它的动态类型和动态值都设为 nil，如下图所示：
`var w io.Writer` // 接口的零值

![Alt text](/images/go-nil-interface-value.png)

在这种情况下就是一个 nil 接口值，可以用 w == nil 或者 w != nil 来检测一个接口值是否时 nil。调用一个 nil 接口的任何方法都会导致崩溃：
`w.Write([]byte("hello"))` // panic：对空指针引用值


#### 接口值的比较
接口值可以用 == 和 != 来做比较。如果两个接口值都是 nil 或者二者的动态类型完全一致且二者动态值相等（使用动态类型的==操作符来比较），那么两个接口值相等。因为接口值是可以比较的，所以它们可以作为 map 的键，也可以作为 switch 语句的操作数。

注意：在比较两个接口值时，如果两个接口值的动态类型一致，但对应的动态值是不可以比较的（比如 slice），那么这个比较会导致代码 panic：
```go
package main

import (
   "fmt"
)

func main() {
   var x interface{} = []int{1, 2, 3}
   fmt.Println(x == x)
}
```
panic 信息:
```
panic: runtime error: comparing uncomparable type []int

goroutine 1 [running]:
main.main()
	/Users/qianghaohao/go/src/examplecode/main.go:9 +0x82
```

#### 获取接口值的动态类型
使用 fmt 包的 %T 可以打印某个接口值的动态类型，有助于问题调试：
```go
func main() {
   var w io.Writer
   fmt.Printf("%T\n", w)  // <nil>

   w = os.Stdout
   fmt.Printf("%T\n", w)  // os.File
}
```
输出：
```
<nil>
*os.File
```
#### nil 接口值和动态值为 nil 接口值的区别
**nil 接口值**：接口值的动态类型和动态值都为 nil。
```go
var w io.Writer
```

![Alt text](/images/go-nil-interface-value.png)

**动态值为 nil 的接口**：接口值的动态类型有具体类型，动态值为 nil。
```go
var buf *bytes.Buffer
var w io.Writer = buf
```

![Alt text](/images/go-interface-value-nil.png)

示例代码分析：
```go
const debug = false

func main() {
   var buf *bytes.Buffer
   if debug {
      buf = new(bytes.Buffer)
   }
   f(buf)
}

func f(out io.Writer) {
   if out != nil {
      out.Write([]byte("done!\n"))
   }
}
```
当 debug 为 true 时，buf 为执行 bytes.Buffer 对象的一个指针，当调用函数 f 时，将 buf 赋给 out，此时 out 的动态值不是 nil，而是指向 byte.Buffer 对象的指针。out 的动态类型是 *byte.Buffer。随后调用 out.Write 函数，代码正常运行，不会引发 panic。

当 debug 为 false 时，调用函数 f 时，将 buf 赋值给 out，此时 out 的动态值虽然时 nil，但其动态类型不是 nil，所以 out != nil 表达式为 true，随后调用 out.Write 函数，但是 Write 函数的接收者为 nil，所以在底层拷贝 buffer 时会导致 panic（空指针引用），具体可以看看 Write 函数的底层实现就知道了。
