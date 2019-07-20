---
title: Go 语言 exec 实时获取外部命令的执行输出
date: 2019-07-20 18:11:47
categories: Go
tags: Go
---

在 Go 语言中调用外部 Linux 命令可以通过标准的 [os/exec](https://go-zh.org/pkg/os/exec/) 包实现，我们一般的使用方式如下：
```go
package main

import (
	"fmt"
	"os/exec"
)

func main() {
	cmd := exec.Command("ls", "-al")
	output, _ := cmd.CombinedOutput()
	fmt.Println(string(output))
}
```
上面这种使用方式虽然能获取到外部命令的执行结果输出 output，但是必须得命令执行完成后才将获取到的结果一次性返回。很多时候我们是需要实时知道命令执行的输出，比如我们调用一个外部的服务构建命令： `mvn build`，这种情况下实时输出命令执行的结果对我们来说很重要。再比如我们 ping 远程 IP，需要知道实时输出，如果直接 ping 完在输出，在使用上来说体验不好。

要实现外部命令执行结果的实时输出，需要使用 `Cmd` 结构的 `StdoutPipe()` 方法创建一个管道连接到命令执行的输出，然后用 for 循环从管道中实时读取命令执行的输出并打印到终端。具体代码如下：
```go
func RunCommand(name string, arg ...string) error {
	cmd := exec.Command(name, arg...)
    // 命令的错误输出和标准输出都连接到同一个管道
	stdout, err := cmd.StdoutPipe()
	cmd.Stderr = cmd.Stdout

	if err != nil {
		return err
	}

	if err = cmd.Start(); err != nil {
		return err
	}
    // 从管道中实时获取输出并打印到终端
	for {
		tmp := make([]byte, 1024)
		_, err := stdout.Read(tmp)
		fmt.Print(string(tmp))
		if err != nil {
			break
		}
	}

	if err = cmd.Wait(); err != nil {
		return err
	}
	return nil
}
```

