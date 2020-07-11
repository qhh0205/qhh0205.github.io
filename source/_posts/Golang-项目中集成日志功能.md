---
title: Golang 项目中集成日志功能
date: 2020-07-11 20:44:02
categories: Go
tags: Go
---

在一个 web 项目中，日志打印功能是必须的，有了详细的日志能为问题排查带来很大的便利。Golang 有很多开源的日志包可供使用，这里我还是使用非常流行的 [logrus](https://github.com/sirupsen/logrus) 包，结合 [file-rotatelogs](https://github.com/lestrrat-go/file-rotatelogs) 包实现日志的自动切割轮转。

集成方法：
- 1.单独定义一个 package 名为 logger，里面只有一个 init.go 文件，初始化日志配置，这个配置是全局的：
  - 日志输出格式为 json；
  - 日志自动轮转，保留最近 7 天日志，一天产生一个日志文件，防止服务长时间运行占用系统过多的磁盘资源；
  - 日志在输出到文件的同时并打到 stdout；
  
	```go
package logger

import (
	"io"
	"os"
	"path"
	"time"

	log "github.com/sirupsen/logrus"
	rotatelogs "github.com/lestrrat-go/file-rotatelogs"

	"github.com/demo/pkg/common/config"
	"github.com/demo/pkg/utils/fs"
)

// 全局日志配置初始化
// 1.日志同时输出到到文件和标准输出；
// 2.保留最近 7 天日志，一天一个日志文件；
//
func init() {
	logDir := config.GetConfig().GetString("log_dir")
	if !fs.PathExists(logDir) {
		if err := os.MkdirAll(logDir, 0755); err != nil {
			panic(err)
		}
	}

	logfile := path.Join(logDir, "app")
	fsWriter, err := rotatelogs.New(
		logfile+"_%Y-%m-%d.log",
		rotatelogs.WithMaxAge(time.Duration(168)*time.Hour),
		rotatelogs.WithRotationTime(time.Duration(24)*time.Hour),
	)
	if err != nil {
		panic(err)
	}

	multiWriter := io.MultiWriter(fsWriter, os.Stdout)
	log.SetReportCaller(true)
	log.SetFormatter(&log.JSONFormatter{})
	log.SetOutput(multiWriter)
	log.SetLevel(log.InfoLevel)
}

```

- 2.在项目入口 main 包引入该包，初始化全局日志配置
```go
package main
import (
    _ "github.com/demo/pkg/logger" // 引入包时自动执行了 logger 包的 init 函数完成了日志初始化配置
)
```

- 3.至此，在项目其他地方如果要打印日志，直接引用 logrus 包即可，在 main 包中已经完成了日志配置，无需在其他使用的地方再次初始化配置；
```go
package hello

import (
    log "github.com/sirupsen/logrus"
)

func HelloLog() {
    log.Info("HelloLog func called")
}
```

