---
title: Java JSch 远程执行 Shell 命令
date: 2021-04-03 00:38:43
categories: Java
tags: Java
---

### 背景
项目需求，需要远程 ssh 登录到某个节点执行 shell 命令来完成任务。对于这种需求，如果不用 java 程序，直接 linux 的 ssh 命令就可以完成，但是在编码到程序中时需要相关的程序包来完成，本文主要介绍在 java 中如何使用 [JSch](http://www.jcraft.com/jsch/) 包实现 ssh 远程连接并执行命令。

### JSch 简介
[JSch](http://www.jcraft.com/jsch/) 是Java Secure Channel的缩写。JSch是一个SSH2的纯Java实现。它允许你连接到一个SSH服务器，并且可以使用端口转发，X11转发，文件传输等，当然你也可以集成它的功能到你自己的应用程序。框架jsch很老的框架，更新到2016年，现在也不更新了。

### JSch 使用 shell 执行命令，有两种方法
- ChannelExec: 一次执行一条命令，一般我们用这个就够了。
- ChannelShell: 可执行多条命令，平时开发用的不多，根据需要来吧；
```java
ChannelExec channelExec = (ChannelExec) session.openChannel("exec");//只能执行一条指令（也可执行符合指令）
ChannelShell channelShell = (ChannelShell) session.openChannel("shell");//可执行多条指令 不过需要输入输出流
```
#### 1. ChannelExec
- 每个命令之间用 ; 隔开。说明：各命令的执行给果，不会影响其它命令的执行。换句话说，各个命令都会执行，但不保证每个命令都执行成功。
- 每个命令之间用 && 隔开。说明：若前面的命令执行成功，才会去执行后面的命令。这样可以保证所有的命令执行完毕后，执行过程都是成功的。
- 每个命令之间用 || 隔开。说明：|| 是或的意思，只有前面的命令执行失败后才去执行下一条命令，直到执行成功一条命令为止。

#### 2. ChannelShell
对于ChannelShell，以输入流的形式，可执行多条指令，这就像在本地计算机上使用交互式shell（它通常用于：交互式使用）。如要要想停止，有两种方式： 
- 发送一个exit命令，告诉程序本次交互结束；
- 使用字节流中的available方法，来获取数据的总大小，然后循环去读。


### 使用示例
#### 1. 引入 pom 依赖
```
<dependency>
   <groupId>com.jcraft</groupId>
   <artifactId>jsch</artifactId>
   <version>0.1.53</version>
</dependency>
```

#### 2. jsch 使用示例
在此封装了一个 Shell 工具类，用来执行 shell 命令，具体使用细节在代码注释中有说明，可以直接拷贝并使用，代码如下：
```java
package org.example.shell;
/**
 * Created by qianghaohao on 2021/3/28
 */


import com.jcraft.jsch.ChannelExec;
import com.jcraft.jsch.JSch;
import com.jcraft.jsch.Session;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

/**
 * @description:
 * @author: qianghaohao
 * @time: 2021/3/28
 */
public class Shell {
    private String host;
    private String username;
    private String password;
    private int port = 22;
    private int timeout = 60 * 60 * 1000;

    public Shell(String host, String username, String password, int port, int timeout) {
        this.host = host;
        this.username = username;
        this.password = password;
        this.port = port;
        this.timeout = timeout;
    }

    public Shell(String host, String username, String password) {
        this.host = host;
        this.username = username;
        this.password = password;
    }

    public String execCommand(String cmd) {
        JSch jSch = new JSch();
        Session session = null;
        ChannelExec channelExec = null;
        BufferedReader inputStreamReader = null;
        BufferedReader errInputStreamReader = null;
        StringBuilder runLog = new StringBuilder("");
        StringBuilder errLog = new StringBuilder("");
        try {
            // 1. 获取 ssh session
            session = jSch.getSession(username, host, port);
            session.setPassword(password);
            session.setTimeout(timeout);
            session.setConfig("StrictHostKeyChecking", "no");
            session.connect();  // 获取到 ssh session

            // 2. 通过 exec 方式执行 shell 命令
            channelExec = (ChannelExec) session.openChannel("exec");
            channelExec.setCommand(cmd);
            channelExec.connect();  // 执行命令

            // 3. 获取标准输入流
            inputStreamReader = new BufferedReader(new InputStreamReader(channelExec.getInputStream()));
            // 4. 获取标准错误输入流
            errInputStreamReader = new BufferedReader(new InputStreamReader(channelExec.getErrStream()));

            // 5. 记录命令执行 log
            String line = null;
            while ((line = inputStreamReader.readLine()) != null) {
                runLog.append(line).append("\n");
            }

            // 6. 记录命令执行错误 log
            String errLine = null;
            while ((errLine = errInputStreamReader.readLine()) != null) {
                errLog.append(errLine).append("\n");
            }

            // 7. 输出 shell 命令执行日志
            System.out.println("exitStatus=" + channelExec.getExitStatus() + ", openChannel.isClosed="
                    + channelExec.isClosed());
            System.out.println("命令执行完成，执行日志如下:");
            System.out.println(runLog.toString());
            System.out.println("命令执行完成，执行错误日志如下:");
            System.out.println(errLog.toString());
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (inputStreamReader != null) {
                    inputStreamReader.close();
                }
                if (errInputStreamReader != null) {
                    errInputStreamReader.close();
                }

                if (channelExec != null) {
                    channelExec.disconnect();
                }
                if (session != null) {
                    session.disconnect();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        return runLog.toString();
    }
}
```

上述工具类使用：
```java
package org.example;

import org.example.shell.Shell;

/**
 * Hello world!
 *
 */
public class App {
    public static void main( String[] args ) {
        String cmd = "ls -1";
        Shell shell = new Shell("192.168.10.10", "ubuntu", "11111");
        String execLog = shell.execCommand(cmd);
        System.out.println(execLog);
    }
}

```

### 需要注意的点
如果需要后台执行某个命令，不能直接 `<命令> + &` 的方式执行，这样在 JSch 中不生效，需要写成这样的格式：`<命令> > /dev/null 2>&1 &`。比如要后台执行 `sleep 60`，需要写成 `sleep 60 > /dev/null 2>&1`

具体 issue 见这里：[https://stackoverflow.com/questions/37833683/running-programs-using-jsch-in-the-background](https://stackoverflow.com/questions/37833683/running-programs-using-jsch-in-the-background)


### 参考文档
[https://www.cnblogs.com/slankka/p/11988477.html](https://www.cnblogs.com/slankka/p/11988477.html)

[https://blog.csdn.net/sinat_24928447/article/details/83022818](https://blog.csdn.net/sinat_24928447/article/details/83022818)
