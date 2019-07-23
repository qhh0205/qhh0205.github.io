---
title: Golang 协程顺序打印
date: 2019-07-23 16:26:55
categories: Go
tags: Go
---

### A、B 两个协程分别打印 1、2、3、4 和 A，B，C，D
实现：定义 A、B 两个 channal，开 A、B 两个协程，A 协程输出[1, 2, 3, 4]、B 协程输出[A, B, C, D]，通过两个独立的 channal 控制顺序，交替输出。
```go
func main() {
	A := make(chan bool, 1)
	B := make(chan bool)
	Exit := make(chan bool)
	go func() {
		s := []int{1, 2, 3, 4}
		for i := 0; i < len(s); i++ {
			if ok := <-A; ok {
				fmt.Println("A: ", s[i])
				B <- true
			}
		}
	}()
	go func() {
		defer func() {
			close(Exit)
		}()
		s := []byte{'A', 'B', 'C', 'D'}
		for i := 0; i < len(s); i++ {
			if ok := <-B; ok {
				fmt.Printf("B: %c\n", s[i])
				A <- true
			}
		}
	}()
	A <- true
	<-Exit
}
```
### A、B 两个协程顺序打印 1~20
实现：与上面基本一样，定义 A、B 两个 channal，开 A、B 两个协程，A 协程输出奇数、B 协程输出偶数，通过两个独立的 channal 控制顺序，交替输出。
```go
package main

import "fmt"

func main() {
	A := make(chan bool, 1)
	B := make(chan bool)
	Exit := make(chan bool)

	go func() {
		for i := 1; i <= 10; i++ {
			if ok := <-A; ok {
				fmt.Println("A = ", 2*i-1)
				B <- true
			}
		}
	}()
	go func() {
		defer func() {
			close(Exit)
		}()
		for i := 1; i <= 10; i++ {
			if ok := <-B; ok {
				fmt.Println("B : ", 2*i)
				A <- true
			}
		}
	}()

	A <- true
	<-Exit
}
```

