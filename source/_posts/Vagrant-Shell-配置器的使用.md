---
title: Vagrant Shell 配置器的使用
date: 2018-05-06 22:26:27
categories: Vagrant
tags: Vagrant
---

## Vagrant Shell 配置器的使用
> **摘要：** 本文翻译自 [Vagrant](https://www.vagrantup.com/docs/) 官方文档 [Shell Provisioner](https://www.vagrantup.com/docs/provisioning/file.html#destination) 部分，主要介绍了 Vagrant Shell 配置器的使用，如何使用内联脚本代码和外部脚本文件来配置虚拟机。

**Provisioner name:** "shell"

Vagrant Shell 配置器允许你向虚拟机上传脚本并执行。

对于想要快速启动和运行 Vagrant 的新手来说，Shell 配置器非常理想，并为不熟悉 `Chef` 或 `Puppet` 等配置管理系统的用户提供了选择。

对于类 POSIX 机器，shell 配置器使用 SSH 执行脚本。对于 Windows 机器，使用 WinRM 来执行脚本，shell 配置器通过 WinRM 执行 PowerShell 和 Batch （**译者注：批处理**） 脚本。

### 选项
shell 配置器有很多选项，其中 `inline` 或者 `path` 选项是必须的：
* [inline](https://www.vagrantup.com/docs/provisioning/shell.html#inline) (string) - 指定在远程机器执行的 shell 内联命令。更多信息见下面 [inline scripts](https://www.vagrantup.com/docs/provisioning/shell.html#inline-scripts) 章节。
* [path](https://www.vagrantup.com/docs/provisioning/shell.html#path) (string) - 要上传并执行的 shell 脚本路径。该脚本可以是一个相对于 Vagrantfile 工程的文件或者是一个远程脚本（比如 [gist](https://gist.github.com/)）。

剩下的这些选项是可选的：
* [args](https://www.vagrantup.com/docs/provisioning/shell.html#args) (string or array) - 当以单一字符串执行脚本时，传递给脚本的参数。这些参数必须写得好像它们直接在命令行上键入一样，所以在需要时一定要避免字符、引号等。你也可以使用数组来传递参数。在这种情况下，Vagrant 将会为你处理引用。
* [env](https://www.vagrantup.com/docs/provisioning/shell.html#env) (string) - 传递给脚本作为环境变量的键值对。Vagrant 会处理环境变量值的引用，但是键保持不变。
* [binary](https://www.vagrantup.com/docs/provisioning/shell.html#binary) (boolean) - Vagrant 会自动用 Unix 换行符来代替 Windows 换行符。如果设置为 false，不会替换。默认值是 "false"。如果 shell 配置器通过 WinRM 来交互，那么默认值是 "true"。
* [privileged](https://www.vagrantup.com/docs/provisioning/shell.html#privileged) (boolean) - 指定是否以超级用户执行脚本。默认是 "true"。Windows guest 虚拟机使用计划任务作为真正的管理员运行，而不受WinRM限
* [upload_path](https://www.vagrantup.com/docs/provisioning/shell.html#upload_path) (string) - 上传脚本的远程路径。脚本将会通过 SCP 上传到 SSH 用户，因此该路径对于 SSH 用户必须是可写的。默认路径是 `/tmp/vagrant-shell`。在 Windows 下，这个默认路径是 `C:\tmp\vagrant-shell`。
* [keep_color](https://www.vagrantup.com/docs/provisioning/shell.html#keep_color) (boolean) -  Vagrant 根据终端输出是否是 stdout 或 stderr 自动地以绿色或者红色输出。如果设置为 true，Vagrant 不会输出颜色，而使用脚本的原生色彩输出。
* [name](https://www.vagrantup.com/docs/provisioning/shell.html#name) (string) - 该值将显示在终端输出中，以便在许多 shell 配置器存在时让用户识别更容易。
* [powershell_args](https://www.vagrantup.com/docs/provisioning/shell.html#powershell_args) (string) - 传递给 `PowerShell` 的额外参数，如果你在 Windows 使用 PowerShell。
* [powershell_elevated_interactive](https://www.vagrantup.com/docs/provisioning/shell.html#powershell_elevated_interactive) (boolean) - 在 Windows 交互式运行脚本。默认是 "false"。也必须享有特权。一定要启用 Windows 的自动登录，因为用户必须登录才能使用交互模式。
* [md5](https://www.vagrantup.com/docs/provisioning/shell.html#md5) (string) - 用于验证远程下载的 shell 文件的 MD5 值。
* [sha1](https://www.vagrantup.com/docs/provisioning/shell.html#sha1) (string) - 用于验证远程下载的 shell 文件的 SHA1 值。
* [sensitive](https://www.vagrantup.com/docs/provisioning/shell.html#sensitive) (boolean) - 将 `env` 选项中的 Hash 值标记为敏感数据，并将其从输出中隐藏起来。默认值是 "false"。

### 内联脚本
也许最简单的入门方式是使用内联脚本。内联脚本是直接在 Vagrantfile 文件中给定的脚本代码。例如：
```ruby
Vagrant.configure("2") do |config|
  config.vm.provision "shell",
    inline: "echo Hello, World"
end
```
当配置器运行时，会在虚拟机中运行 `echo Hello, World`。

结合少量 Ruby 代码，很容易将 shell 脚本直接嵌入到 Vagrantfile 文件。举例如下：
```ruby
$script = <<-SCRIPT
echo I am provisioning...
date > /etc/vagrant_provisioned_at
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.provision "shell", inline: $script
end
```
我知道如果你对 Ruby 不熟悉，那么上面的配置可能会看起来非常高级，但是不用害怕，它做的很简单：将 shell 脚本被赋 `$shell` 变量。这个全局变量包含一个字符串，然后作为内联脚本传递给 Vagrant 配置文件。

当然，如果在 Vagrantfile 中还有除了基本变量赋值之外的其他 Ruby 代码使你感到不舒服，那么您可以使用一个实际的脚本文件，在下一节中将对此进行说明。

对于 Windows 虚拟机，内联脚本必须是是 PowerShell。Batch（**译者注：批处理**） 脚本不能作为内联脚本。

### 外部脚本
shell 配置器还可以指定本地主机上的脚本的路径。Vagrant 会将该脚本上传到虚拟机并执行。例如：
```ruby
Vagrant.configure("2") do |config|
  config.vm.provision "shell", path: "script.sh"
end
```
上面的路径是相对于中 Vagrantfile 的路径。也可以使用绝对路径，以及 `~` （家目录）和 `..`  等快捷方式。

如果你使用远程脚本作为配置器，你也可以将远程脚本的 URL 作为 `path` 的参数：
```ruby
Vagrant.configure("2") do |config|
  config.vm.provision "shell", path: "https://example.com/provisioner.sh"
end
```
如果你在 Windows 上运行 Batch（**译者注：批处理**） 或者 PowerShell 脚本，请确保外部路径具有适当的扩展名（".bat" 或者 ".ps1"），由于 Windows 使用扩展名来决定所执行文件的类型。如果没有扩展名，那么脚本可能无法使用。

如果运行一个已经在虚拟机存在的脚本文件，你可以使用内联脚本来调用该远程虚拟机脚本：
```ruby
Vagrant.configure("2") do |config|
  config.vm.provision "shell",
    inline: "/bin/sh /path/to/the/script/already/on/the/guest.sh"
end
```

### 脚本参数
您可以像任何普通的shell脚本一样参数化脚本。这些参数可以指定给 shell 配置器。应该将它们指定为字符串，因为它们将作为命令行的输入，因此确保正确地转义任何字符：
```ruby
Vagrant.configure("2") do |config|
  config.vm.provision "shell" do |s|
    s.inline = "echo $1"
    s.args   = "'hello, world!'"
  end
end
```
如果您不想担心引用，则还可以将参数指定为数组：
```ruby
Vagrant.configure("2") do |config|
  config.vm.provision "shell" do |s|
    s.inline = "echo $1"
    s.args   = ["hello, world!"]
  end
end
```
