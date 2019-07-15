---
title: Go 语言读写文件
date: 2019-07-15 21:45:52
categories: Go
tags: Go
---

在这里演示下如何通过 Go 读写文件，Go 读写文件有很 IO 多函数可以使用，在这里使用 os 包的 OpenFile 和 Open 函数打开文件，然后用 bufio 包带缓冲的读写器读写文件。查看 OpenFile 源码，其实 Open 函数底层还是调用了 OpenFile。
```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"os"
)

func main() {
	//           写文件
	outputFile, outputError := os.OpenFile("file.txt", os.O_WRONLY|os.O_CREATE|os.O_APPEND, 0666)
	if outputError != nil {
		fmt.Println(outputError)
		return
	}
	defer outputFile.Close()
	outputWriter := bufio.NewWriter(outputFile)
	for i := 0; i < 10; i++ {
		outputWriter.WriteString("hello, world\n")
	}
	// 一定得记得将缓冲区内容刷新到磁盘文件
	outputWriter.Flush()
	//           读文件
	inputFile, inputError := os.Open("file.txt")
	if inputError != nil {
		fmt.Println(inputError)
		return
	}
	defer inputFile.Close()
	inputReader := bufio.NewReader(inputFile)
	for {
		inputString, readerError := inputReader.ReadString('\n')
		fmt.Printf(inputString)
		if readerError == io.EOF {
			return
		}
	}
}
```
