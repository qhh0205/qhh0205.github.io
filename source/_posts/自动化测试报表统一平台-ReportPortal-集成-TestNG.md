---
title: 自动化测试报表统一平台 ReportPortal 集成 TestNG
date: 2019-07-10 21:39:41
categories: DevOps
tags: DevOps
---

本文主要介绍 [ReportPortal](https://reportportal.io/) 如何集成 TestNG 测试框架，用到的工具链有：ReportPortal + TestNG + log4j。ReportPortal 集成 TestNG 的主要原理是通过给 TestNG 配置 ReportPortal 的 listener，在测试开始时该监听器将测试信息实时上报给 ReportPortal 平台。另外我们通过给 log4j 配置 ReportPortal appender，将测试过程中代码日志也上报到 ReportPortal 平台。然后 ReportPortal 将收到的数据进行整合、分析，形成平台数据统一展示。
#### ReportPortal 简介
ReportPortal 是一个统一的自动化测试报告收集、分析、可视化平台，可以集成多种测试框架，比如 TestNG、Selenium 等等。ReportPortal 的主要特性有：
- 能轻易和多种测试框架集成；
![Alt text](/images/rp_feature1.png)
- 实时展示测试情况；
- 所有的自动化测试结果在一个地方统一查看；
- 保留历史测试信息；
- 能和 bug 跟踪系统集成，比如 Jira；

#### ReportPortal 解决了什么问题
个人认为 ReportPortal 最大的价值在于报表的统一收集、查看、分析。假如没有 ReportPortal 工具，我们可能需要自己写脚本，或者 Jenkins 插件针对不同的测试框架装不同的插件，然后展示测试报告，但是 Jenkins 收集的测试报告只能在 Jenkins 平台查看。微服务拆分细、导致 Jenkins job 数量比较多，要看每次测试的报告要逐个点开进去查看，没有一个全局的地方查看。另外 Jenkins 本身的插件生态提供的测试报告收集不支持对历史测试报告的统一查询，如果有这种需求，基本不能满足。

ReportPortal 基本是全测试框架支持的统一报表收集、分析、可视化平台，能轻松解决上存在的痛点。


#### ReportPortal 在 CI/CD 中扮演了什么角色
CI/CD 我们已经很熟悉了，但是如何将 CI/CD 与 CT 无缝整合，也许 ReportPortal 在 CI/CD 与 CT 的整合中扮演了重要角色。DevOps 的关键在于自动化统一标准、流程，根据 ReportPortal 的特性及本人的试用，发现 ReportPortal 真是对 CI/CD 完美的补充，整个交付流水线更加统一、规范、简洁、无缝衔接。

![Alt text](/images/rp_role_cicd.png)


### ReportPortal + TestNG + log4j 集成详细步骤
以一个基于 TestNG 测试框架的 java 工程为例说明，配置前 java 工程目录结构：
```
.
├── pom.xml
├── README.md
├── run.sh
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   └── resources
    │       ├── config.properties
    │       ├── dev.yml
    │       ├── log4j.properties
    │       ├── log4testng.properties
    │       ├── production.yml
    │       ├── stage.yml
    │       ├── test.yml
    │       └── web.yml
    └── test
        ├── java
        │   └── com
        └── resources
            └── testng.xml
```
#### 1. 配置工程 pom 文件
##### 1.1 配置 ReportPortal 相关依赖远程仓库
```xml
<repositories>
        <repository>
            <id>bintray</id>
            <url>http://dl.bintray.com/epam/reportportal</url>
        </repository>
        <repository>
            <id>jitpack.io</id>
            <url>https://jitpack.io</url>
        </repository>
</repositories>
```
##### 1.2 添加一些依赖配置
**testng 依赖**
```xml
<dependency>
    <groupId>org.testng</groupId>
    <artifactId>testng</artifactId>
    <version>6.11</version>
</dependency>
```
**ReportPortal agent 的 testng 实现**
```
<dependency>
    <groupId>com.epam.reportportal</groupId>
    <artifactId>agent-java-testng</artifactId>
    <version>4.2.1</version>
</dependency>
```
**添加 Rport Portal 的 log 包装以及 log4j 本身的配置**
```xml
<dependency>
    <groupId>com.epam.reportportal</groupId>
    <artifactId>logger-java-log4j</artifactId>
    <version>4.0.1</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.26</version>
</dependency>
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```
##### 1.3 maven-surefire-plugin 插件配置
说明：
src/test/resources/testng.xml：tesng 执行测试用例时读取的文件，该文件指定执行哪些测试用例等信息；
```xml
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-surefire-plugin</artifactId>
<version>2.22.0</version>
<configuration>
    <testFailureIgnore>true</testFailureIgnore>
    <suiteXmlFiles>
      <suiteXmlFilexmlFile>src/test/resources/testng.xml</suiteXmlFilexmlFile>
    </suiteXmlFiles>
    <properties>
      <property>
        <name>usedefaultlisteners</name> <!-- disabling default listeners is optional -->
        <value>false</value>
      </property>
    </properties>
</configuration>
</plugin>
```
##### 1.4 maven-compiler-plugin 插件配置
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.0</version>
    <configuration>
       <source>1.8</source>
       <target>1.8</target>
    </configuration>
</plugin>
```
### 2. TestNG 配置 RportPortal listener
testng  testng.xml  文件添加 RportPortal 的 listener，文件：src/test/resources/testng.xml
```xml
<listeners>
       <listener class-name="com.epam.reportportal.testng.ReportPortalTestNGListener"/>
</listeners>
```
### 3. 工程添加 ReportPortal resource
src/test/resources/ 目录添加 ReportPortal 配置文件：reportportal.properties
**获取 ReportPortal 配置：**
访问 ReportPortal UI--->点击右上角图标--->点击 Profile--->拷贝右下角框框中 REQUERED 配置。

**将上面获取到的 ReportPortal 配置放到 src/test/resources/ 目录下reportportal.properties 文件：**
示例文件内容：
```
rp.endpoint = http://reportIp:8080
rp.uuid = xxxxx
rp.launch = superadmin_TEST_EXAMPLE
rp.project = superadmin_personal
```

### 4. 配置 log4j 的 ReportPortal appender
src/test/resources/ 目录添加 log4j.xml 文件，主要是配置 log4j 的 ReportPortal appender，内容如下:
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
<log4j:configuration debug="true"
                     xmlns:log4j='http://jakarta.apache.org/log4j/'>

    <appender name="ReportPortalAppender" class="com.epam.ta.reportportal.log4j.appender.ReportPortalAppender">
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="[%d{HH:mm:ss}] %-5p (%F:%L) - %m%n"/>
        </layout>
    </appender>
    <logger name="com.epam.reportportal.apache">
        <level value="OFF"/>
    </logger>
    <root>
        <level value="info"/>
        <appender-ref ref="ReportPortalAppender"/>
    </root>
</log4j:configuration>
```
至此相关配置已完成，工程目录结构此时为：
```
.
├── pom.xml
├── README.md
├── run.sh
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   └── resources
    │       ├── config.properties
    │       ├── dev.yml
    │       ├── log4j.properties
    │       ├── log4testng.properties
    │       ├── production.yml
    │       ├── stage.yml
    │       ├── test.yml
    │       └── web.yml
    └── test
        ├── java
        │   └── com
        └── resources
            ├── log4j.xml
            ├── reportportal.properties
            └── testng.xml
```
### 5. 执行 `mvn clean test` 测试
### 6. 到 ReportPortal 控制台观察新的 Launches 是否启动
每次测试会在 ReportPortal 平台对应触发一个 Launche，包含本次构建相关信息。
![Alt text](/images/rp_launche.png)

### 7. 创建 ReportPortal Dashboard，可视化测试报告
创建 ReportPortal 的 Dashboard 很简单，也很灵活，主要思想是 RP 提供了多种图表，然后每个图表配置条件，筛选出想要的 Launches 展示。
![Alt text](/images/rp_dashboard.png)

### 相关文档
https://reportportal.io/docs
https://github.com/reportportal/example-java-TestNG
