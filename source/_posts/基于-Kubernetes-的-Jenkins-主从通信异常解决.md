---
title: 基于 Kubernetes 的 Jenkins 主从通信异常解决
date: 2018-10-14 13:24:08
categories: DevOps
tags: Jenkins
---

#### 问题描述
[基于 Kubernetes 部署 Jenkins 动态 slave](https://qhh0205.github.io/2018/10/14/%E5%9F%BA%E4%BA%8E-Kubernetes-%E7%9A%84%E5%8A%A8%E6%80%81-Jenkins-slave-%E9%83%A8%E7%BD%B2/) 后，运行 Jenkins Job 会抛java.nio.channels.ClosedChannelException 异常完整的异常栈如下：
```
FATAL: java.nio.channels.ClosedChannelException
java.nio.channels.ClosedChannelException
Also:   hudson.remoting.Channel$CallSiteStackTrace: Remote call to JNLP4-connect connection from 10.244.8.1/10.244.8.1:55340
		at hudson.remoting.Channel.attachCallSiteStackTrace(Channel.java:1741)
		at hudson.remoting.Request.call(Request.java:202)
		at hudson.remoting.Channel.call(Channel.java:954)
		at hudson.FilePath.act(FilePath.java:1071)
		at hudson.FilePath.act(FilePath.java:1060)
		at hudson.FilePath.mkdirs(FilePath.java:1245)
		at hudson.model.AbstractProject.checkout(AbstractProject.java:1202)
		at hudson.model.AbstractBuild$AbstractBuildExecution.defaultCheckout(AbstractBuild.java:574)
		at jenkins.scm.SCMCheckoutStrategy.checkout(SCMCheckoutStrategy.java:86)
		at hudson.model.AbstractBuild$AbstractBuildExecution.run(AbstractBuild.java:499)
		at hudson.model.Run.execute(Run.java:1819)
		at hudson.model.FreeStyleBuild.run(FreeStyleBuild.java:43)
		at hudson.model.ResourceController.execute(ResourceController.java:97)
		at hudson.model.Executor.run(Executor.java:429)
Caused: hudson.remoting.RequestAbortedException
	at hudson.remoting.Request.abort(Request.java:340)
	at hudson.remoting.Channel.terminate(Channel.java:1038)
	at org.jenkinsci.remoting.protocol.impl.ChannelApplicationLayer.onReadClosed(ChannelApplicationLayer.java:209)
	at org.jenkinsci.remoting.protocol.ApplicationLayer.onRecvClosed(ApplicationLayer.java:222)
	at org.jenkinsci.remoting.protocol.ProtocolStack$Ptr.onRecvClosed(ProtocolStack.java:832)
	at org.jenkinsci.remoting.protocol.FilterLayer.onRecvClosed(FilterLayer.java:287)
	at org.jenkinsci.remoting.protocol.impl.SSLEngineFilterLayer.onRecvClosed(SSLEngineFilterLayer.java:172)
	at org.jenkinsci.remoting.protocol.ProtocolStack$Ptr.onRecvClosed(ProtocolStack.java:832)
	at org.jenkinsci.remoting.protocol.NetworkLayer.onRecvClosed(NetworkLayer.java:154)
	at org.jenkinsci.remoting.protocol.impl.NIONetworkLayer.ready(NIONetworkLayer.java:142)
	at org.jenkinsci.remoting.protocol.IOHub$OnReady.run(IOHub.java:795)
	at jenkins.util.ContextResettingExecutorService$1.run(ContextResettingExecutorService.java:28)
	at jenkins.security.ImpersonatingExecutorService$1.run(ImpersonatingExecutorService.java:59)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
Finished: FAILURE
```

#### 原因及解决方法
##### 原因
抛 java.nio.channels.ClosedChannelException 异常的原因是 Jenkins Slave Pod 在 Jenkins Job 运行时突然挂掉，然后 Master Pod 无法和 Slave Pod 进行通信。那么解决方法就是找到 Slave Pod 经常挂掉的原因，经排查是 Slave Pod 的资源限制不合理，配置的 CPU 和内存太小，导致 Pod 在运行是很容易超出资源限制，然后被 k8s Kill 掉。
##### 解决方法
 打开 Jenkins 设置 Slave Pod 模版的资源限制：
Jenkins->系统管理->系统设置->云->镜像->Kubernetes Pod Template->Container Template->高级，然后根据实际情况调整 CPU 和内存需求。
 
#### 相关文档
https://github.com/GoogleCloudPlatform/continuous-deployment-on-kubernetes/issues/118
https://medium.com/@garunski/closedchannelexception-in-jenkins-with-kubernetes-plugin-a7788f1c62a9
