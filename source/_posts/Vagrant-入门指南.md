---
title: Vagrant 入门指南
date: 2018-04-22 13:06:06
categories: Vagrant
tags: Vagrant
---

### Vagrant 简介
Vagrant 是一个用来构建和管理虚拟机环境的工具。Vagrant 有着易于使用的工作流，并且专注于自动化，降低了开发者搭建环境的时间，提高了生产力。解决了“在我的机器上可以工作”的问题。

Vagrant 是为了方便的实现虚拟化环境而设计的，使用 Ruby 开发，基于 VirtualBox 等虚拟机管理软件的接口，提供了一个可配置、轻量级的便携式虚拟开发环境。使用 Vagrant 可以很方便的就建立起来一个虚拟环境，而且可以模拟多台虚拟机，这样我们平时还可以在开发机模拟分布式系统。

团队新员工加入，常常会遇到花一天甚至更多时间来从头搭建完整的开发环境，而有了Vagrant，只需要直接将已经打包好的 package（里面包括开发工具，代码库，配置好的服务器等）拿过来就可以工作了，这对于提升工作效率非常有帮助。
### 为什么选择 Vagrant
Vagrant 提供了一个易于配置，可重复使用，兼容的环境，通过一个单一的工作流程来控制，帮助你和团队最大化生产力和灵活性。
为了实现 Vagrant 的魔力，Vagrant 站在了巨人的肩膀上。虚拟机的配置基于 VirtualBox，VMware，AWS 或者其他提供商。然后一些配置工具，比如 shell 脚本，Chef 或者 Puppet 可以自动化地在虚拟机安装并配置软件。
#### 对于开发者人员
如果你是一个开发者，Vagrant 将在一个一次性的、一致的环境中隔离依赖项及其配置，而不会影响你习惯使用的任何工具(编辑器、浏览器、调试器等)。一旦你或者其他人创建了一个 Vagrantfile，你只需要执行 `vagrant up` 所有的东西就自动安装和配置了。你团队中的其他成员使用同一个配置文件来创建开发环境，因此不管你工作在 Linux，MacOS X 还是 Windows， 所有团队的成员都可以在统一的环境环境中运行代码，这样就可以避免“在我的机器上可以工作”的问题。
#### 对于运维人员
如果你是一个运维工程师或者 DevOps 工程师，Vagrant 给予你一个一次性的环境来开发和测试基础架构管理脚本。你可以使用本地虚拟机（比如 VirtualBox 或者 VMware）马上测试一些东西，比如 shell 脚本，Chef cookbooks，Puppet 模块等。然后，你可以用同样的配置在远程云上，比如 AWS 或者 RackSpace，来测试这些脚本。抛弃之前自定义脚本来回收 EC2 实例吧，停止使用 SSH 在各种机器之间跳来跳去，请开始使用 Vagrant 来给你的工作带来更多便利。
### Vagrant 和 Terraform 的区别
Vagrant 和 Terraform 都出自同一个公司 [HashiCorp](https://www.hashicorp.com/)，该公司主要做一些开源软件，相关的产品还有 [Packer](https://www.packer.io/)，[Consul](https://www.consul.io/)，[Vault](https://www.vaultproject.io/)，[Nomad](https://www.nomadproject.io/) 等。

Terraform 的主要用途是管理云提供商的远程资源，比如 AWS。Terraform 可以管理横跨多个云提供商巨量的基础设施。而 Vagrant 主要用来管理仅使用少量虚拟机的本地开发环境。

Vagrant 用于开发环境，Terraform 普遍用于基础设施管理。

---
### VirtualBox 安装
VirtualBox 是 Oracle 开源的虚拟化系统，和 VMware 是同类产品，支持多个平台，可以到官方网站：https://www.virtualbox.org/wiki/Downloads 下载适合你平台的 VirtualBox 最新版本并安装。
>**提示：**对于 Mac 用户，如果系统为 OSX 10.13.3(mac OS High Sierra) 或者更高版本，安装过程可能会失败，报错提示`安装失败，安装器遇到了一个错误导致安装失败...`，原因是新版本 Mac 系统的安全机制阻止外部内核扩展安装，导致安装失败。两种解决方法：
1. 进入系统偏好设置>安全性与隐私>通用，然后手动允许；
2. 在终端敲命令禁用此安全特性：`sudo spctl --master-disable`；

### Vagrant 安装
到官方网站下载相应系统平台的安装包：http://www.vagrantup.com/downloads.html  
直接根据向导进行操作即可完成安装，安装完后就可以在终端输入 `vagrant` 命令了。
>**提示：**尽量下载最新的程序，因为VirtualBox经常升级，升级后有些接口会变化，老的Vagrant 可能无法使用。

### Vagrant 启动第一台虚拟机
到此准备工作（VirtualBox 和 Vagrant 安装）基本上做完了，接下来就可以通过 Vagrant 来启动一台虚拟机了。

在启动虚拟机之前先简单介绍下 `Vagrant box`：`box` 是一个打包好的操作系统，是一个后缀名为 `.box` 的文件，其实是一个压缩包，里面包含了 Vagrant 的配置信息和 VirtualBox 的虚拟机镜像文件。`vagrant up` 启动虚拟机是基于 `box` 文件的，因此在启动虚拟机前必须得把 `box` 文件准备好。或者也可以在启动的时候指定远程 `box` 地址，在这里我把 `box` 文件下载下来，然后启动时指定该文件。

我使用网上分享的 `ubuntu-server-16.04` 这个 `box`，由于[vagrant 官方 box](http://www.vagrantbox.es/) 下载速度特别慢，所以在此提供一下该 `box` 的百度网盘下载地址，加速下载：https://pan.baidu.com/s/1wJCeWEyxKQLVPi1IH1IlYg
1. 新建一个目录作为 `Vagrant` 的工程目录
```bash
$haohao cd /Users/haohao
$haohao mkdir vagrant
```

2. 添加前面下载的 `box`
添加 `box` 命令格式：`vagrant box add <本地 box 名称> <box 文件>`
  - 本地 `box` 名称：自定义名称，该名称是本地 `vagrant` 管理时使用的名称；
  - `box` 文件：前面下载的 `vagrant box` 文件或者远程 `box` url 地址；
```bash
$haohao vagrant box add ubuntu-server-16.04 ubuntu-server-16.04-amd64-vagrant.box
==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'ubuntu-server-16.04' (v0) for provider:
    box: Unpacking necessary files from: file:///Users/haohao/vagrant/ubuntu-server-16.04-amd64-vagrant.box
==> box: Successfully added box 'ubuntu-server-16.04' (v0) for 'virtualbox'!
```
3. 查看 `box` 是否添加成功
查看当前 `vagrant` 中有哪些 `box`：`vagrant box list`
```bash
$haohao vagrant box list
ubuntu-server-16.04 (virtualbox, 0)
```
4. 初始化上面添加的 `box`
初始化命令格式：`vagrant init <本地 box 名称>`
本地 `box` 名称：第 2 步中添加的 `box` 名称
这里初始化前面添加的 `box`，初始化后会在当前目录生产一个 `Vagrantfile` 文件，里面包含了虚拟机的各种配置，关于具体每个配置项是什么意思，后面会介绍。
```bash
$haohao vagrant init 'ubuntu-server-16.04'
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```
5. 启动虚拟机
虚拟机启动命令：`vagrant up`
启动虚拟机时会自动将当前目录（即 Vagrantfile 文件所在目录），和虚拟机的 `/vagrant` 目录共享。
```bash
$haohao vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'ubuntu-server-16.04'...
==> default: Matching MAC address for NAT networking...
==> default: Setting the name of the VM: vagrant_default_1524288099752_62326
==> default: Clearing any previously set network interfaces...
.....
==> default: Mounting shared folders...
    default: /vagrant => /Users/haohao/vagrant
```
6. 连接虚拟机
命令格式：`vagrant ssh`
```bash
$haohao vagrant ssh
Welcome to Ubuntu 16.04.3 LTS (GNU/Linux 4.4.0-98-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.

Last login: Sat Apr 21 05:28:37 2018 from 10.0.2.2
ubuntu@ubuntu-xenial:~$
```
7. 查看 Vagrant 共享目录
进入虚拟机后执行 `df -h` 可以看到 Vagrant 默认把宿主机 `Vagrantfile` 所在的目录和虚拟机的 `/vagrant` 目录共享，可以通过 `ls /vagrant/` 查看该目录内容，内容和宿主机对应目录一致。
```bash
ubuntu@ubuntu-xenial:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            490M     0  490M   0% /dev
tmpfs           100M  3.1M   97M   4% /run
/dev/sda1       9.7G  857M  8.8G   9% /
tmpfs           497M     0  497M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           497M     0  497M   0% /sys/fs/cgroup
vagrant         234G   49G  185G  21% /vagrant
tmpfs           100M     0  100M   0% /run/user/1000

# ls 查看该共享目录内容和宿主机内容一致
ubuntu@ubuntu-xenial:~$ ls /vagrant/
ubuntu-xenial-16.04-cloudimg-console.log  Vagrantfile
```

### Vagrant 配置文件浅析
前面我们执行 `vagrant init <本地 box 名称>` 会在当前目录生成 `Vagrantfile` 文件，这个文件是非常重要的，一般给别人共享自己的环境时都是提供一个 `Vagrantfile` 和一个 `box` 文件，这样就可以很轻松地将环境共享给别人，别人能得到一模一样的统一的环境，这就是使用 `Vagrant` 的好处。

`Vagrantfile` 主要包括三个方面的配置，虚拟机的配置、SSH配置、Vagrant 的一些基础配置。Vagrant 是使用 Ruby 开发的，所以它的配置语法也是 Ruby 的，对于没有学过 Ruby 的朋友也没关系，根据例子模仿下就会了。

修改完配置后需要执行 `vagrant reload` 重启 VM 使其配置生效。

以下简单介绍下常用配置的配置项：
1. `box` 名称设置
`config.vm.box = "base"`
	上面这配置展示了 Vagrant 要去启用那个box作为系统，也就是前面我们输入 `vagrant init <本地 box 名称>`时所指定的 `box`，如果沒有输入 `box` 名称的话，那么默认就是 `base`。

2. VM 相关配置
VirtualBox 提供了 VBoxManage 这个命令行工具，可以让我们设定 VM，用`modifyvm`这个命令让我们可以设定 VM 的名称和内存大小等等，这里说的名称指的是在 VirtualBox 中显示的名称，我们也可以在 Vagrantfile 中进行设定，举例如下：
>调用 VBoxManage 的 `modifyvm` 的命令，设置 VM 的名称为 `ubuntu`，内存为 1024 MB。你可以类似的通过定制其它 VM 属性来定制你自己的 VM。
```bash
config.vm.provider "virtualbox" do |v|
  v.customize ["modifyvm", :id, "--name", "ubuntu", "--memory", "1024"]
end
```
3. 网络设置
	Vagrant 有两种方式来进行网络连接，一种是 host-only (主机模式)，这种模式下所有的虚拟系统是可以互相通信的，但虚拟系统和真实的网络是被隔离开的，虚拟机和宿主机是可以互相通信的，相当于两台机器通过双绞线互联。另一种是Bridge(桥接模式)，该模式下的 VM  就像是局域网中的一台独立的主机，可以和局域网中的任何一台机器通信，这种情况下需要手动给 VM 配 IP 地址，子网掩码等。我们一般选择 host-only 模式，配置如下：
```bash
config.vm.network :private_network, ip: "11.11.11.11"
```
4. hostname 设置
	`hostname` 的设置非常简单：
```bash
config.vm.hostname = "kubernetes"
```
5. 目录共享
	我们前面介绍过`/vagrant`目录默认就是当前的开发目录，这是在虚拟机开启的时候默认挂载同步的。我们还可以通过配置来设置额外的同步目录：
```bash
# 第一个参数是主机的目录，第二个参数是虚拟机挂载的目录
config.vm.synced_folder  "/Users/haohao/data", "/vagrant_data"
```
6. 端口转发
对宿主机器上 8080 端口的访问请求 forward 到虚拟机的 80 端口的服务上：
```bash
config.vm.network :forwarded_port, guest: 80, host: 8080
```
### Vagrant 常用命令清单
- `vagrant box add` 添加box
- `vagrant init` 初始化 box
- `vagrant up` 启动虚拟机
- `vagrant ssh` 登录虚拟机
- `vagrant box list` 列出 Vagrant 当前 `box` 列表
- `vagrant box remove` 删除相应的 `box`
- `vagrant destroy` 停止当前正在运行的虚拟机并销毁所有创建的资源
- `vagrant halt` 关机
- `vagrant package` 把当前的运行的虚拟机环境进行打包为 `box` 文件
- `vagrant plugin` 安装卸载插件
- `vagrant reload` 重新启动虚拟机，重新载入配置文件
- `vagrant resume` 恢复被挂起的状态
- `vagrant ssh-config` 输出用于 ssh 连接的一些信息
- `vagrant status`  获取当前虚拟机的状态
- `vagrant suspend` 挂起当前的虚拟机
- `vagrant up` 重启虚拟机

### Vagrant 启动虚拟机集群
前面我们都是通过一个 `Vagrantfile` 配置启动单台机器，如果我们要启动一个集群，那么可以把需要的节点在一个 `Vagrantfile` 写好，然后直接就可以通过 `vagrant up` 同时启动多个 VM 组成一个集群。以下示例配置一个 web 节点和一个 db 节点，两个节点在同一个网段，并且使用同一个 `box` 启动：
```bash
Vagrant.configure("2") do |config|
  config.vm.define :web do |web|
    web.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--name", "web", "--memory", "512"]
    end
    web.vm.box = "ubuntu-server-16.04"
    web.vm.hostname = "web"
    web.vm.network :private_network, ip: "11.11.1.1"
  end

  config.vm.define :db do |db|
    db.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--name", "db", "--memory", "512"]
    end
    db.vm.box = "ubuntu-server-16.04"
    db.vm.hostname = "db"
    db.vm.network :private_network, ip: "11.11.1.2"
  end
end
```
### 相关连接
https://www.vagrantup.com/intro/index.html
https://blog.csdn.net/rickiyeat/article/details/55097687
https://github.com/astaxie/go-best-practice/blob/master/ebook/zh/01.0.md
