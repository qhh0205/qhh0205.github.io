---
title: 使用 Go 开发命令行应用
date: 2019-07-24 11:07:31
categories: Go
tags: Go
---

作为一个程序员，命令行工具是我们再熟悉不过的了，我们每天或多或少都会用到命令行工具。比如项目构建、打包、启动等等。那么如何用 Go 语言编写类似的工具呢？调研了下，大概有下面三种方法：
- **os.Args 函数**
os.Args 功能类似于 Shell 脚本的 `$@` 功能，获取到命令行输入，然后进行人工解析处理，这种方式对于编写简单的工具还行，对于复杂点的工具，光解析输入参数就是一场"灾难"了。
- **使用 Go 标准 [flag](https://go-zh.org/pkg/flag/) 包**
[flag](https://go-zh.org/pkg/flag/) 包是 Golang 官方提供的命令行参数解析包，省去人工解析工作，对于构建一个一般功能的命令行应用足够了，但是对于复杂应用还是显得比较麻烦。
- **使用第三方包 [cli](https://github.com/urfave/cli) 或者 [cobra](https://github.com/spf13/cobra)**
使用第三方包就比较专业了，提供的功能更加丰富，使用起来也很顺手。目前对于 Golang 命令行应用开发业界比较主流的就是 [cli](https://github.com/urfave/cli) 和 [cobra](https://github.com/spf13/cobra) 了。其实两者的流行度差不多，cobra 的学习成本稍微高点，而且更加专业，看文档介绍，Kubernetes、Moby、rkt、etcd 等都是基于 cobra 构建。我个人更推荐使用 [cli](https://github.com/urfave/cli)，因为 cli 比较轻量级、容易学习，使用起来也更加得心应手，对于开发日常应用足够了。

本文主要介绍后面两种方式，即 flag 包和第三方包 cli 的使用，对于 cobra 的使用这里不做具体介绍。

### 使用 flag 包构建命令行应用
 flag 包的使用很简单，能很方便地解析命令行输入，支持的命令行参数类型有  bool, int, uint, string, time.Duration, float 类型，另外还可以自定义类型。这里介绍下一般的使用方法，假如要开发一个命令行工具，使用方式：
```go
./go-curl -v -X "GET" https://example.com
```
具体实现代码：
```go
func main() {
	// 定义一个 bool 类型的参数，默认值为 false，第三个参数为 Usage 说明
	// 函数返回值为对应类型的指针
	v := flag.Bool("v", false, "Makes  curl  verbose  during the operation.")

	// 定义一个 String 类型的参数，默认值为 GET，第三个参数为 Usage 说明
	// 与上面那种不同的是函数第一个参数为变量的指针
	var X string
	flag.StringVar(&X, "X", "GET", "(HTTP) Specifies a custom request method to use when communicating with the HTTP server.")

	// 在参数定义完后必须调用 flag.Parse() 完成命令行参数的解析
	flag.Parse()

	// 返回其余参数的列表
	args := flag.Args()
	fmt.Printf("v: %t, X: %s, args: %v\n", *v, X, args)
	os.Exit(0)
}
```
可以看出定义参数有两种方式：
- 指针类型参数：调用函数为 `flag.Type` 形式，返回值为对应类型的指针；
- 值类型参数：调用函数为 `flag.TypeVar` 形式，调用时传递变量的指针；

两种方法效果都是一样的，只不过一种解析返回的是指针，另一种是直接使用变量。
### 使用 cli 包构建命令行应用
使用 cli 包开发命令行工具能省很多事，而且写出来的代码结构非常清晰，很容易理解。具体使用见代码仓库 [README.md](https://github.com/urfave/cli) 。在这里举一个例子，我最近写的服务部署命令行工具：https://github.com/qhh0205/deploy-kit   通过看官方文档结合这个例子能很容易掌握 cli 包的是使用技巧。下面为该部署工具的 `--help` 输出：
```go
NAME:
   deploy - deploy application

USAGE:
   deploy [global options] command [command options] [arguments...]

VERSION:
   v1.0

COMMANDS:
     list, ls           list all of services
     app                deploy microservice application
     web                deploy web application
     lsbranch, lsb      list the code branches of service
     upload-cdn, upcdn  upload file or directory to gcs bucket
     help, h            Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --help, -h     show help
   --version, -v  print the version
```
该工具基于 cli 包构建，cli 包的使用核心是通过 `app.Commands = []cli.Command{}`  定义一些列命令、选项，并且通过 `Action` 绑定对应选项参数的处理函数。

### 参考文档
https://github.com/urfave/cli | cli GitHub
https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter13/13.1.html | flag 包介绍
https://blog.yumaojun.net/2016/12/30/go-cobra/ | cobra 使用，如何使用golang编写漂亮的命令行工具
https://blog.rapid7.com/2016/08/04/build-a-simple-cli-tool-with-golang/ | Building a Simple CLI Tool with Golang
https://medium.com/what-i-talk-about-when-i-talk-about-technology/dealing-with-command-line-options-in-golang-flag-package-e5fb6ef1a79e | Dealing with Command Line Options in Golang: flag package

