---
title: Golang 从 Json 串中快速取出需要的字段
date: 2020-07-11 20:38:32
categories: Go
tags: Go
---

在 web 编程中很多情况下接口的数据是 json 格式，在我们拿到接口的 json 数据后如何方便地从中提取出需要的字段呢？我们可以自定义一个结构体，然后通过 Golang 的标准库 json 解析到我们定义的结构体中。但是当 json 格式比较复杂，嵌套层级比较深的时候，还是用这种方法就比较麻烦了。在这里推荐一个包: [gojsonq](https://github.com/thedevsaddam/gojsonq)，可以很简便地从 json 串中提取出需要的字段，无需定义额外的结构体，然后解析，直接链式地从 json 串中提取需要的字段即可，有点类似动态语言：
```bash
go get github.com/thedevsaddam/gojsonq/v2
```
- Github 地址：
[https://github.com/thedevsaddam/gojsonq](https://github.com/thedevsaddam/gojsonq)

- Package 简介：
A simple Go package to Query over JSON Data. It provides simple, elegant and fast ODM like API to access, query JSON document.

示例一：从如下 json 串中提取 name.first 字段数据
```json
{
    "name":{
        "first":"Tom",
        "last":"Hanks"
    },
    "age":61
}
```
Find 方法返回的结果为 interface{} 类型。
```Go
package main

import gojsonq "github.com/thedevsaddam/gojsonq/v2"

func main() {
	const json = `{"name":{"first":"Tom","last":"Hanks"},"age":61}`
	name := gojsonq.New().FromString(json).Find("name.first")
	println(name.(string)) // Tom
}
```

示例二：从如下 json 串中提取  temperatures 字段数据，并计算平均值
```json
{
    "city":"dhaka",
    "type":"weekly",
    "temperatures":[
        30,
        39.9,
        35.4,
        33.5,
        31.6,
        33.2,
        30.7
    ]
}
```
支持列表计算：
```go
package main

import (
	"fmt"

	gojsonq "github.com/thedevsaddam/gojsonq/v2"
)

func main() {
	const json = `{"city":"dhaka","type":"weekly","temperatures":[30,39.9,35.4,33.5,31.6,33.2,30.7]}`
	avg := gojsonq.New().FromString(json).From("temperatures").Avg()
	fmt.Printf("Average temperature: %.2f", avg) // 33.471428571428575
}
```

示例三：从如下 json 串中提取 data.pageNumber 数组，赋值给 Golang 整型数组

上面的例子提取简单的数值类型，返回的是 interface{} 类型，可以通过类型断言得到需要的数据。如果提取的是数组类型，是无法通过类型断言来获取 interface{} 类型的值的，可以使用 FindR 拿到结果，然后通过结果的 As 方法将数据解析到我们定义的变量类型中。
```json
{
    "code":"0",
    "message":"success",
    "data":{
        "thumb_up":0,
        "system":0,
        "im":0,
        "avatarUrl":"https://profile.csdnimg.cn/F/4/B/helloworld",
        "invitation":0,
        "comment":0,
        "follow":0,
        "totalCount":0,
        "coupon_order":0,
        "pageNumbers":[
            1,
            2,
            3,
            4,
            5,
            6
        ]
    },
    "status":true
}
```

```go
func main() {
	jsonStr := `{
   "code":"0",
   "message":"success",
   "data":{
       "thumb_up":0,
       "system":0,
       "im":0,
       "avatarUrl":"https://profile.csdnimg.cn/F/4/B/helloworld",
       "invitation":0,
       "comment":0,
       "follow":0,
       "totalCount":0,
       "coupon_order":0,
       "pageNumbers":[
           1,
           2,
           3,
           4,
           5,
           6
       ]
   },
   "status":true
   }`

	// 保存 json 字符串中 pageNumbers 字段列表数据
	pageNumbers := make([]int, 0)
	result, err := gojsonq.New().FromString(jsonStr).FindR("data.pageNumbers")
	if err != nil {
		fmt.Println(err)
	}
	// 将提前的数据解析为自定义变量类型
	err = result.As(&pageNumbers)
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println(pageNumbers)
}
```

更多功能及示例见官方文档：
[https://github.com/thedevsaddam/gojsonq/wiki/Queries](https://github.com/thedevsaddam/gojsonq/wiki/Queries)

