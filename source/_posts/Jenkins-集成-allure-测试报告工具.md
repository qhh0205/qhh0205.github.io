---
title: Jenkins 集成 allure 测试报告工具
date: 2019-07-25 11:35:48
categories: DevOps
tags: DevOps
---

[allure](https://docs.qameta.io/allure/) 基于已有的测试报告数据进行进一步的加工，美化等操作，相当于做了一次数据格式转换。allure 支持多种语言的多种测试框架，比如 Java 的 jUnit4、jUnit5、TestNg 等等。

本文主要介绍如何在 Jenkins 中集成 allure 测试报表工具，在每次项目自动化测试完成后，用 allure 生成经过加工后的测试报告。我们以 java 工程的 TestNg 测试为例，处理 TestNg 生成的测试报告。

- Jenkins 安装 allure 插件
全局工具配置：

	![](/images/jenkins-allure4.png)

- Jenkinsfile 添加 allure 代码
```yaml
script {
    allure jdk: '', report: "target/allure-report-unit", results: [[path: "target/surefire-reports"]]
}
```
	`target/allure-report-unit` 参数：allure 报告生成路径；
	`target/surefire-reports` 参数：测试报告原始路径；

- Jenkins 平台查看 allure 报告
在 allure 成功集成到 Jenkins 后，allure 每次处理完成，在 Jenkins job 页面都可以看到 allure 的图标，点击图标即可查看报告详细信息:

	![](/images/jenkins-allure1.png)

	![](/images/jenkins-allure2.png)

### 遇到问题及解决方法
问题：
allure 在 Jenkins pipline 中生成报表时报目录权限问题：java.nio.file.AccessDeniedException

![](/images/jenkins-allure3.png)

原因：
jenkins k8s pod 执行 job 时默认用户为 jenkins，但是 pipeline 中调用的容器生成的文件的属主是 root。
解决方法：
配置 jenkins k8s 插件模版，添加安全配置，运行用户设置为 root
https://groups.google.com/forum/#!topic/jenkinsci-users/GR0n8ZkCJ-E

pod 配置（spec 下）：
```yaml
securityContext:
    runAsUser: 0
    fsGroup: 0
```

### 参考文档
https://docs.qameta.io/allure/
