---
title: Golang logrus 日志包及日志切割
date: 2020-03-01 11:56:11
categories: Go
tags: Go
---

本文主要介绍 Golang 中最佳日志解决方案，包括常用日志包 [logrus](https://github.com/sirupsen/logrus) 的基本使用，如何结合 [file-rotatelogs](https://github.com/lestrrat-go/file-rotatelogs) 包实现日志文件的轮转切割两大话题。

Golang 关于日志处理有很多包可以使用，标准库提供的 log 包功能比较少，不支持日志级别的精确控制，自定义添加日志字段等。在众多的日志包中，更推荐使用第三方的 logrus 包，完全兼容自带的 log 包。logrus 是目前 Github 上 star 数量最多的日志库，logrus 功能强大，性能高效，而且具有高度灵活性，提供了自定义插件的功能。

很多开源项目，如 docker，prometheus，dejavuzhou/ginbro 等，都是用了 logrus 来记录其日志。

### logrus 特性
- 完全兼容 golang 标准库日志模块：logrus 拥有六种日志级别：debug、info、warn、error、fatal 和 panic，这是 golang 标准库日志模块的 API 的超集。
 - logrus.Debug("Useful debugging information.")
 - logrus.Info("Something noteworthy happened!")
 - logrus.Warn("You should probably take a look at this.")
 - logrus.Error("Something failed but I'm not quitting.")
 - logrus.Fatal("Bye.") //log之后会调用os.Exit(1)
 - logrus.Panic("I'm bailing.") //log之后会panic()
- 可扩展的 Hook 机制：允许使用者通过 hook 的方式将日志分发到任意地方，如本地文件系统、标准输出、logstash、elasticsearch 或者 mq 等，或者通过 hook 定义日志内容和格式等。
- 可选的日志输出格式：logrus 内置了两种日志格式，JSONFormatter 和 TextFormatter，如果这两个格式不满足需求，可以自己动手实现接口 Formatter 接口来定义自己的日志格式。
- Field 机制：logrus 鼓励通过 Field 机制进行精细化的、结构化的日志记录,而不是通过冗长的消息来记录日志。
- logrus 是一个可插拔的、结构化的日志框架。
- Entry: logrus.WithFields 会自动返回一个 *Entry，Entry里面的有些变量会被自动加上
 - time:entry被创建时的时间戳
 - msg:在调用.Info()等方法时被添加
 - level，当前日志级别

### logrus 基本使用
```go
package main

import (
    "os"

    "github.com/sirupsen/logrus"
    log "github.com/sirupsen/logrus"
)

var logger *logrus.Entry

func init() {
    // 设置日志格式为json格式
    log.SetFormatter(&log.JSONFormatter{})
    log.SetOutput(os.Stdout)
    log.SetLevel(log.InfoLevel)
    logger = log.WithFields(log.Fields{"request_id": "123444", "user_ip": "127.0.0.1"})
}

func main() {
    logger.Info("hello, logrus....")
    logger.Info("hello, logrus1....")
    // log.WithFields(log.Fields{
    //  "animal": "walrus",
    //  "size":   10,
    // }).Info("A group of walrus emerges from the ocean")

    // log.WithFields(log.Fields{
    //  "omg":    true,
    //  "number": 122,
    // }).Warn("The group's number increased tremendously!")

    // log.WithFields(log.Fields{
    //  "omg":    true,
    //  "number": 100,
    // }).Fatal("The ice breaks!")
}
```

### 基于 logrus 和 file-rotatelogs 包实现日志切割
很多时候应用会将日志输出到文件系统，对于访问量大的应用来说日志的自动轮转切割管理是个很重要的问题，如果应用不能妥善处理日志管理，那么会带来很多不必要的维护开销：外部工具切割日志、人工清理日志等手段确保不会将磁盘打满。
>file-rotatelogs: When you integrate this to to you app, it automatically write to logs that are rotated from within the app: No more disk-full alerts because you forgot to setup logrotate!

[logrus](https://github.com/sirupsen/logrus) 本身不支持日志轮转切割功能，需要配合 [file-rotatelogs](https://github.com/lestrrat-go/file-rotatelogs) 包来实现，防止日志打满磁盘。[file-rotatelogs](https://github.com/lestrrat-go/file-rotatelogs)  实现了 io.Writer 接口，并且提供了文件的切割功能，其实例可以作为 logrus 的目标输出，两者能无缝集成，这也是 [file-rotatelogs](https://github.com/lestrrat-go/file-rotatelogs) 的设计初衷：
>It's normally expected that this library is used with some other logging service, such as the built-in log library, or loggers such as github.com/lestrrat-go/apache-logformat.

示例代码：
应用日志文件 /Users/opensource/test/go.log，每隔 1 分钟轮转一个新文件，保留最近 3 分钟的日志文件，多余的自动清理掉。
```go
package main

import (
	"time"

	rotatelogs "github.com/lestrrat-go/file-rotatelogs"
	log "github.com/sirupsen/logrus"
)

func init() {
	path := "/Users/opensource/test/go.log"
	/* 日志轮转相关函数
	`WithLinkName` 为最新的日志建立软连接
	`WithRotationTime` 设置日志分割的时间，隔多久分割一次
	WithMaxAge 和 WithRotationCount二者只能设置一个
	  `WithMaxAge` 设置文件清理前的最长保存时间
	  `WithRotationCount` 设置文件清理前最多保存的个数
	*/
	// 下面配置日志每隔 1 分钟轮转一个新文件，保留最近 3 分钟的日志文件，多余的自动清理掉。
	writer, _ := rotatelogs.New(
		path+".%Y%m%d%H%M",
		rotatelogs.WithLinkName(path),
		rotatelogs.WithMaxAge(time.Duration(180)*time.Second),
		rotatelogs.WithRotationTime(time.Duration(60)*time.Second),
	)
	log.SetOutput(writer)
	//log.SetFormatter(&log.JSONFormatter{})
}

func main() {
	for {
		log.Info("hello, world!")
		time.Sleep(time.Duration(2) * time.Second)
	}
}
```

### Golang 标准日志库 [log](https://golang.org/pkg/log/) 使用
虽然 Golang 标准日志库功能少，但是可以选择性的了解下，下面为基本使用的代码示例，比较简单：
```go
package main

import (
    "fmt"
    "log"
)

func init() {
    log.SetPrefix("【UserCenter】")                           // 设置每行日志的前缀
    log.SetFlags(log.LstdFlags | log.Lshortfile | log.LUTC) // 设置日志的抬头字段
}

func main() {
    log.Println("log...")
    log.Fatalln("Fatal Error...")
    fmt.Println("Not print!")
}
```
自定义日志输出
```go
package main

import (
    "io"
    "log"
    "os"
)

var (
    Info    *log.Logger
    Warning *log.Logger
    Error   *log.Logger
)

func init() {
    errFile, err := os.OpenFile("errors.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
    if err != nil {
        log.Fatalln("打开日志文件失败：", err)
    }

    Info = log.New(os.Stdout, "Info:", log.Ldate|log.Ltime|log.Lshortfile)
    Warning = log.New(os.Stdout, "Warning:", log.Ldate|log.Ltime|log.Lshortfile)
    Error = log.New(io.MultiWriter(os.Stderr, errFile), "Error:", log.Ldate|log.Ltime|log.Lshortfile)
}

func main() {
    Info.Println("Info log...")
    Warning.Printf("Warning log...")
    Error.Println("Error log...")
}
```
### 相关文档
[https://mojotv.cn/2018/12/27/golang-logrus-tutorial](https://mojotv.cn/2018/12/27/golang-logrus-tutorial)
[https://github.com/lestrrat-go/file-rotatelogs](https://github.com/lestrrat-go/file-rotatelogs)
[https://www.flysnow.org/2017/05/06/go-in-action-go-log.html](https://www.flysnow.org/2017/05/06/go-in-action-go-log.html)

