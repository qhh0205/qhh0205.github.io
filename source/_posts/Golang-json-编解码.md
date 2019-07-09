---
title: Golang json 编解码
date: 2019-07-09 19:23:03
categories: Go
tags: Go
---

#### Json 编码
Json 编码的过程即为将程序的数据结构转化为 json 串的过程，比如 Golang 里面的结构体、Python 中的字典，这些有结构的数据转化为 json 串。在 Golang 中编码 Json 使用 `encoding/json` 包的 Marshal() 函数，函数原型为：
```go
func Marshal(v interface{}) ([]byte, error)
```
举例：将 Book 结构体对象编码为 json 串，然后输出到控制台
```go
import (
    "encoding/json"
    "fmt"
)

type Book struct {
    Title       string
    Authors     []string
    Publisher   string
    IsPublished bool
    Price       float64
}

func main() {
    var gobook Book = Book{
        "Go语言编程",
        []string{"XuShiwei", "HughLv"},
        "ituring.com.cn",
        true,
        9.99,
    }
    b, _ := json.Marshal(gobook)
    fmt.Println(string(b))
}
```

#### Json 解码
Json 解码的过程和编码刚好相反，将普通的 json 字符串转化为有结构的程序数据。比如将 json 串转化为 Golang 的结构体。在Golang 中解码 json 的函数为 `encoding/json` 包的 `Unmarshal()` 函数，函数原型为：
```go
func Unmarshal(data []byte, v interface{}) error
```
举例：将 json 字符串解码成 Golang Book 结构体对象，并打印每个字段的值
```go
package main

import (
    //"encoding/json"
    "encoding/json"
    "fmt"
)

type Book struct {
    Title       string
    Authors     []string
    Publisher   string
    IsPublished bool
    Price       float64
}

func main() {
    jsonStr := `{"Title":"Go语言编程","Authors":["XuShiwei","HughLv"],"Publisher":"ituring.com.cn","IsPublished":true,"Price":9.99}`
    var book Book
    json.Unmarshal([]byte(jsonStr), &book)
    fmt.Println(book.Title, book.Authors, book.Publisher, book.IsPublished, book.Price)
}
```
