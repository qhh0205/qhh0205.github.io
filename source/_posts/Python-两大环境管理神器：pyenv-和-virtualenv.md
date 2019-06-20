---
title: Python 两大环境管理神器：pyenv 和 virtualenv
date: 2018-03-24 20:01:08
categories: Python
tags: Python
---
#### 简介
---
[pyenv](https://github.com/pyenv/pyenv) 是一个开源的 Python 版本管理工具，可以轻松地给系统安装任意 Python 版本，想玩哪个版本，瞬间就可以切换。有了 ``pyenv``，我们不需要再为系统多版本 Python 共存问题而发愁，也不用为手动编译安装其他 Python 版本而浪费时间，只需要执行一条简单的命令就可以切换并使用任何其他版本，该工具真正地做到了开箱即用，简单实用。

[virtualenv](https://github.com/pypa/virtualenv) 是一个用来创建完全隔离的 Python 虚拟环境的工具，可以为每个项目工程创建一套独立的 Python 环境，从而可以解决不同工程对 Python 包，或者版本的依赖问题。假如有 A 和 B 两个工程，A 工程代码要跑起来需要 ``requests 1.18.4``，而 B 工程跑起来需要 ``requests 2.18.4``，这样在一个系统中就无法满足两个工程同时运行问题了。最好的解决办法是用 ``virtualenv`` 给每个工程创建一个完全隔离的 Python 虚拟环境，给每个虚拟环境安装相应版本的包，让程序使用对应的虚拟环境运行即可。这样既不影响系统 Python 环境，也能保证任何版本的 Python 程序可以在同一系统中运行。

**最佳实践：**使用 ``pyenv`` 安装任何版本的 Python，然后用 ``virtualenv`` 创建虚拟环境时指定需要的 Python 版本路径，这样就可以创建任何版本的虚拟环境，这样的实践真是极好的！
#### pyenv 的安装及使用
---
#### 1. 安装
- 将 ``pyenv`` 安装到 ``~/.pyenv`` 目录（当然你可以安装到任意其他路径）
```bash
git clone https://github.com/yyuu/pyenv.git ~/.pyenv
```
- 配置环境变量（我的 Shell 是 zsh，如果是 bash，请添加到 ``~/.bashrc``）
```bash
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
```
- 添加 ``pyenv`` 初始化（我的 Shell 是 zsh，如果是 bash，请添加到 ``~/.bashrc``）
```bash
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
```
- 使当前 Shell 配置生效，完成安装
```bash
exec $SHELL
source ~/.zshrc
```
#### 2. 使用
- 查看有哪些 Python 版本可以安装
```bash
pyenv install --list
```
- 安装某个 Python 版本
```bash
pyenv install -v 3.6.4
```
- 查看当前 Python 版本情况（``*`` 表示系统当前的 Python 版本，``system``表示系统初始版本）
```bash
$ pyenv versions
  system
  2.6.7
* 3.6.4 (set by /Users/haohao/.pyenv/version)
```
- 切换 Python 版本（切换之后查看当前版本）
```bash
$ pyenv global 3.6.4
$ pyenv versions
  system
* 3.6.4 (set by /Users/haohao/.pyenv/version)
$ python -V
Python 3.6.4
```
- 卸载某个 Python 版本
```
pyenv uninstall 3.6.4
```
#### virtualenv 的安装及使用
---
#### 1. 安装
```bash
sudo pip install virtualenv
```
#### 2. 使用
下面我们使用 ``virtualenv`` 创建一个完全隔离的 Python 虚拟环境：
1. 新建一个目录（一般用来用作工程路径）
```bash
$ mkdir myproject
```
2. 进入目录创建一个完全独立干净的虚拟环境
如果 ``virtualenv`` 后面不加任何参数，那么默认创建的虚拟环境的 Python 版本是系统当前版本，如果要创建其他版本，可以使用 ``-p`` 参数指定其他版本的 ``python`` 可执行文件路径。可执行文件可以在上一步安装的 ``pyenv`` 的 ``~/.pyenv/versions`` 路径找到，该路径是 ``pyenv`` 管理的所有 Python 版本路径。
```bash
# 使用系统当前的 Python 版本创建虚拟环境
$ virtualenv venv
New python executable in /Users/haohao/PycharmProjects/myproject/venv/bin/python
# 创建虚拟环境时指定 Python 版本
$ virtualenv -p ~/.pyenv/versions/2.6.7/bin/python venv
Running virtualenv with interpreter /Users/haohao/.pyenv/versions/2.6.7/bin/python
New python executable in /Users/haohao/PycharmProjects/myproject/venv/bin/python
Installing setuptools<37, pip, wheel<0.30...done.
```
3. 激活创建的虚拟环境并使用
可以看出当前虚拟环境版本已经是 Python 2.6.7 了，而且所在路径确实是在上一步创建的虚拟环境路径。接下来使用 ``pip`` 安装的任何包都会安装在虚拟环境目录里面，不会安装在系统标准目录，从而保证当前环境是绝对干净的，对于系统是完全隔离的。
```bash
$ source venv/bin/activate
$ which python
/Users/haohao/PycharmProjects/myproject/venv/bin/python
$ python -V
Python 2.6.7
```
4. 退出虚拟环境，回到系统版本
```bash
$ deactivate
```
