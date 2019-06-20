---
title: Vagrant 文件配置器的使用
date: 2018-05-06 14:13:46
categories: Vagrant
tags: Vagrant
---

> **摘要：** 本文翻译自 [Vagrant](https://www.vagrantup.com/docs/) 官方文档 [File Provisioner](https://www.vagrantup.com/docs/provisioning/file.html#destination) 部分，主要介绍了文件配置器的使用，如何使用文件配置器将本地文件上拷贝到 Vagrant 所管理的虚拟机上面。

**Provisioner name:** "file"

Vagrant 文件配置器可以将本地主机的文件或者目录上传到虚拟机。

文件配置器可以轻松地将本地 `~/.gitconfig` 复制到虚拟机的 `vagrant` 用户家目录，这样的话就不需要在每次配置一个新的虚拟机时运行 `git config --global` 。
```ruby
Vagrant.configure("2") do |config|
  # ... other configuration

  config.vm.provision "file", source: "~/.gitconfig", destination: ".gitconfig"
end
```
如果你想给虚拟机上传一个目录，可以使用下面的文件配置器来完成。复制时，本地文件夹会被复制到虚拟机的 `newfolder` 文件夹。注意，如果想在虚拟机保持和本机同样的文件夹名称，请确保目的路径名称和本地路径名称一致。
```ruby
Vagrant.configure("2") do |config|
  # ... other configuration

  config.vm.provision "file", source: "~/path/to/host/folder", destination: "$HOME/remote/newfolder"
end
```
首先将 `~/path/to/host/folder` 拷贝到虚拟机:
```bash
folder
    ├── script.sh
    ├── otherfolder
    │   └── hello.sh
    ├── goodbye.sh
    ├── hello.sh
    └── woot.sh

    1 directory, 5 files
```
然后将 `~/path/to/host/folder` 拷贝到虚拟机的 `$HOME/remote/newfolder`:
```bash
newfolder
    ├── script.sh
    ├── otherfolder
    │   └── hello.sh
    ├── goodbye.sh
    ├── hello.sh
    └── woot.sh

    1 directory, 5 files
```
注意，文件上传不像文件目录同步，上传的文件或者文件夹不是保持同步的。还是以上面的例子来说，如果你更改了本地的 `~/.gitconfig`，那么虚拟机上的同样的文件并不会更改。

文件配置器的文件上传是通过 `SSH 或 PowerShell` 用户来上传。通常情况下用户的权限是有限的，如果你想将文件上传到需要更高权限的位置，我们推荐将它们上传到临时位置，然后使用 [shell 配置器](https://www.vagrantup.com/docs/provisioning/shell.html)将它们移动到目标的位置。

### 选项
文件配置器只有两个选项，并且是必须的：
* [source](https://www.vagrantup.com/docs/provisioning/file.html#source)(string) - 要上传的本地文件或者文件夹。
* [destination](https://www.vagrantup.com/docs/provisioning/file.html#destination)(string) - 远程虚拟机路径。文件或者文件夹通过 `SCP` 来上传，因此该目的路径需要对 `SSH` 用户可写。`SSH` 用户可以通过 `ssh-config` 来查看，默认是 `vagrant`。

### 注意事项
尽管文件配置器支持尾部斜杠或 "globing"，但这会导致在本地和虚拟机之间复制文件时产生一些令人困惑的结果。例如，如下源路径没有尾部斜杠，而目的路径有尾部斜杠：
```ruby
config.vm.provision "file", source: "~/pathfolder", destination: "/remote/newlocation/"
```
这意味着 `vagrant` 会将 `~/pathfolder` 上传到远程目录 `/remote/newlocation` 底下，结果看起来会像下面这样：
```bash
newlocation
    ├── pathfolder
    │   └── file.sh

    1 directory, 2 files
```
同样的行为也可以通过如下配置来实现：
```ruby
config.vm.provision "file", source: "~/pathfolder", destination: "/remote/newlocation/pathfolder"
```
另一个例子是使用 `.` 号，拷贝本地机器目录里面的文件，不包括上一层目录：
```ruby
config.vm.provision "file", source: "~/otherfolder/.", destination: "/remote/otherlocation"
```
以上配置将 `~/otherfolder` 目录下的所有文件拷贝到新的路径 `/remote/otherlocation`。这个也可以通过指定一个和源文件夹不同的远程文件夹来实现：
```ruby
config.vm.provision "file", source: "/otherfolder", destination: "/remote/otherlocation"
```
