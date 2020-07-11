---
title: Golang 项目配置文件读取之 viper 实践
date: 2020-07-11 20:53:43
categories: Go
tags: Go
---

在我们做一个工程化项目的时候，经常涉及到配置文件的读取，[viper](https://github.com/spf13/viper) 包很好地满足这一需求，而且在 Golang 生态中是流行度最高的。导入方式：
```bash
import "github.com/spf13/viper"
```

这里分享下我对 viper 包的使用关键实践：

首先，在代码工程中单独定义一个包（我一般起名为 config 或者 configloader），这个包专门用来读取加载配置文件，并做相关校验，包里面我定义 3 个函数和 1 个全局变量：
- var viperConfig *viper.Viper:  全局配置变量；
- func Init(configDir string) error: 初始化加载配置文件；
- func GetConfig() *viper.Viper: 获取配置文件，供其他包调用，拿到配置文件实例；
- func validateConfig(v *viper.Viper) error: 校验配置文件；

接下来在工程入口处引用上面这个配置包的 Init 函数做配置文件的初始化和加载，加载的结果就是实例化一个 *viper.Viper 全局变量，在其他包中用的时候调用这个配置包的 func GetConfig() *viper.Vipe 函数即可拿到这个全局变量，即配置文件内容。

示例代码（代码仅供参考，截取字自前做的爬虫程序部分代码）：
```go
package configloader

import (
	"fmt"

	"github.com/spf13/viper"

	"example.com/pkg/util/fs"
)

// viperConfig 全局配置变量
var viperConfig *viper.Viper

// 初始化配置文件相关设置，在 main 包中调用进行初始化加载
func Init(configDir string) error {
	if configDir == "" {
		return fmt.Errorf("config directory is empty")
	}
	if !fs.PathExists(configDir) {
		return fmt.Errorf("no such config directory: %s", configDir)
	}

	viper.SetConfigName("spider")  // name of config file (without extension)
	viper.AddConfigPath(configDir) // path to look for the config file in
	err := viper.ReadInConfig()    // Find and read the config file
	if err != nil {                // Handle errors reading the config file
		return fmt.Errorf("Fatal error config file: %s \n", err)
	}

	viperConfig = viper.GetViper()
	if err = validateConfig(viperConfig); err != nil {
		return fmt.Errorf("invalid configuration, error msg: %s", err)
	}
	return nil
}

// GetConfig 获取全局配置
func GetConfig() *viper.Viper {
	return viperConfig
}

func validateConfig(v *viper.Viper) error {
	var (
		urlListFile     = v.GetString("urlListFile")
		outputDirectory = v.GetString("outputDirectory")
		maxDepth        = v.GetInt("maxDepth")
		crawlInterval   = v.GetInt("crawlInterval")
		crawlTimeout    = v.GetInt("crawlTimeout")
		targetUrl       = v.GetString("targetUrl")
		threadCount     = v.GetInt("threadCount")
	)

	if urlListFile == "" {
		return fmt.Errorf("invalid targetUrl: %s, please check configuration", urlListFile)
	}
	if outputDirectory == "" {
		return fmt.Errorf("invalid targetUrl: %s, please check configuration", outputDirectory)
	}
	if maxDepth <= 0 {
		return fmt.Errorf("invalid maxDepth: %d, please check configuration", maxDepth)
	}
	if crawlInterval <= 0 {
		return fmt.Errorf("invalid crawlInterval: %d, please check configuration", crawlInterval)
	}
	if crawlTimeout <= 0 {
		return fmt.Errorf("invalid crawlTimeout: %d, please check configuration", crawlTimeout)
	}
	if targetUrl == "" {
		return fmt.Errorf("invalid targetUrl: %s, please check configuration", targetUrl)
	}
	if threadCount <= 0 {
		return fmt.Errorf("invalid threadCount: %d, please check configuration", threadCount)
	}

	return nil
}
```

