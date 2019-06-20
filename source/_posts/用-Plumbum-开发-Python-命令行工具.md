---
title: 用 Plumbum 开发 Python 命令行工具
date: 2018-04-30 16:44:31
categories: 翻译
tags: Plumbum
---

>  **摘要**：本文翻译自 Python [Plumbum](https://plumbum.readthedocs.io/en/latest/) 开源库的官方文档 [Plumbum CLI](https://plumbum.readthedocs.io/en/latest/cli.html) 部分，主要介绍如何使用 [Plumbum CLI](https://plumbum.readthedocs.io/en/latest/cli.html) 工具包来开发 Python 命令行应用程序，这是一个非常 Pythonic、容易使用、功能强大的工具包，非常值得广大 Python 程序员掌握并使用。

**译文：**

轻松执行程序的另一方面是轻松编写 CLI 程序。Python 脚本一般使用 `optparse` 或者最新的 `argparse` 及其衍生品来开发命令行工具，但是所有这些表现力有限，而且非常不直观（甚至不够 Pythonic）。Plumbum 的 CLI 工具包提供了一个程序化的方法来构建命令行应用程序，不需要创建一个解析器对象，然后填充一系列“选项”，该 CLI 工具包使用内省机制将这些原语转义成 Pythonic 结构。

总体来看，Plumbum CLI 应用程序是一个继承自 `plumbum.cli.Application` 的类。这些类定义了一个 `main()` 方法，并且可选地公开出方法和属性来作为命令行的选项。这些选项可能需要参数，而任何剩余的位置参数会根据 `main` 函数的声明来将其赋予 `main` 方法。一个简单的 CLI 应用程序看起来像如下这样：
```python
from plumbum import cli

class MyApp(cli.Application):
    verbose = cli.Flag(["v", "verbose"], help = "If given, I will be very talkative")

    def main(self, filename):
        print("I will now read {0}".format(filename))
        if self.verbose:
            print("Yadda " * 200)

if __name__ == "__main__":
    MyApp.run()
```
你可以运行该程序：
```bash
$ python example.py foo
I will now read foo

$ python example.py --help
example.py v1.0

Usage: example.py [SWITCHES] filename
Meta-switches:
    -h, --help                 Prints this help message and quits
    --version                  Prints the program's version and quits

Switches:
    -v, --verbose              If given, I will be very talkative
```
到现在为止，你只看到了非常基本的使用。我们现在开始探索该库。

*新版本 1.6.1：* 你可以直接运行应用程序 `MyApp()`，不需要参数，也不需要调用 `.main()`。

### 应用程序
**Application**  类是你的应用程序的“容器”，该“容器”由一个你需要实现的`main()`方法和任何数量公开选项函数和属性。你的应用程序的入口是类方法 `run`，该方法实例化你的类、解析参数、调用所有的选项函数，然后使用给的位置参数来调用`main()`函数。为了从命令行运行你的应用程序，你所要做的是：
```python
if __name__ == "__main__":
    MyApp.run()
```
除了 `run()` 和 `main()`，`Application` 类还公开了两个内置的选项函数：`help()` 和 `version()`，分别用于显示帮助和程序的版本。默认情况下，`--hep` 和 `-h` 会调用 `help()`，`--version` 和 `-v` 会调用 `version()`，这些函数被调用后会显示相应的信息然后退出（没有处理任何其他选项）。

你可以通过定义类属性来自定义 `help()` 和 `version()` 显示的信息，比如 `PROGNAME`, `VERSION` 和 `DESCRIPTION`。举例：
```python
class MyApp(cli.Application):
    PROGNAME = "Foobar"
    VERSION = "7.3"
```

### 颜色
*新版本 1.6*

该库也支持终端字符颜色控制。你可以直接将 `PROGNAME`, `VERSION` 和 `DESCRIPTION` 变为带颜色的字符串。如果你给 `PROGNAME` 设置了颜色，你会得到自定义的程序名字和颜色。使用方法字符串的颜色可以通过设置 `COLOR_USAGE` 来生效，不同选项组的颜色可以通过设置 `COLOR_GROUPS` 字典来生效。

举例如下：
```python
class MyApp(cli.Application):
    PROGNAME = colors.green
    VERSION = colors.blue | "1.0.2"
    COLOR_GROUPS = {"Meta-switches" : colors.bold & colors.yellow}
    opts =  cli.Flag("--ops", help=colors.magenta | "This is help")
```
```bash
SimpleColorCLI.py 1.0.2

Usage:
    SimpleColorCLI.py [SWITCHES]

Meta-switches
    -h, --help         Prints this help message and quits
    --help-all         Print help messages of all subcommands and quit
    -v, --version      Prints the program's version and quits

Switches
    --ops              This is help
```

### 选项函数
`switch` 装饰器是该 CLI 开发工具包的“灵魂”，它会公开你的 CLI 应用程序的方法来作为 CLI 命令行选项，这些方法运行通过命令行来调用。我们测试下如下应用：
```python
class MyApp(cli.Application):
    _allow_root = False       # provide a default

    @cli.switch("--log-to-file", str)
    def log_to_file(self, filename):
        """Sets the file into which logs will be emitted"""
        logger.addHandler(FileHandle(filename))

    @cli.switch(["-r", "--root"])
    def allow_as_root(self):
        """If given, allow running as root"""
        self._allow_root = True

    def main(self):
        if os.geteuid() == 0 and not self._allow_root:
            raise ValueError("cannot run as root")
```
当程序运行时，选项函数通过相应的参数被调用。比如，`$ ./myapp.py --log-to-file=/tmp/log` 将被转化成调用 `app.log_to_file("/tmp/log")`。在选项函数被执行后，程序的控制权会被传递到 `main` 方法。
>**注意**
>方法的文档字符串和参数名字会被用来渲染帮助信息，尽量保持你的代码 [**DRY**](https://www.wikiwand.com/en/Don't_repeat_yourself)。
>
>**autoswitch** 可以从函数名字中推断出选项的名称，举例如下：
>
>```python
@cli.autoswitch(str)
def log_to_file(self, filename):
    pass
```
>这会将选项函数和 `--log-to-file` 绑定。

### 选项参数
如上面例子所示，选项函数可能没有参数（不包括 `self`）或者有一个参数。如果选项函数接受一个参数，必须指明该参数的*类型*。如果你不需要特殊的验证，只需传递 `str`，否则，您可能会传递任何类型（或实际上可调用的任何类型），该类型将接收一个字符串并将其转换为有意义的对象。如果转换是不可行的，那么会抛出 `TypeError` 或者 `ValueError` 异常。

举例：
```python
class MyApp(cli.Application):
    _port = 8080

    @cli.switch(["-p"], int)
    def server_port(self, port):
        self._port = port

    def main(self):
        print(self._port)
```
```bash
$ ./example.py -p 17
17
$ ./example.py -p foo
Argument of -p expected to be <type 'int'>, not 'foo':
    ValueError("invalid literal for int() with base 10: 'foo'",)
```
工具包包含两个额外的"类型"（或者是是验证器）：`Range` 和 `Set`。`Range` 指定一个最小值和最大值，限定一个整数在该范围内（闭区间）。`Set` 指定一组允许的值，并且期望参数匹配这些值中的一个。示例如下：
```python
class MyApp(cli.Application):
    _port = 8080
    _mode = "TCP"

    @cli.switch("-p", cli.Range(1024,65535))
    def server_port(self, port):
        self._port = port

    @cli.switch("-m", cli.Set("TCP", "UDP", case_sensitive = False))
    def server_mode(self, mode):
        self._mode = mode

    def main(self):
        print(self._port, self._mode)
```
```bash
$ ./example.py -p 17
Argument of -p expected to be [1024..65535], not '17':
    ValueError('Not in range [1024..65535]',)
$ ./example.py -m foo
Argument of -m expected to be Set('udp', 'tcp'), not 'foo':
    ValueError("Expected one of ['UDP', 'TCP']",)
```
>**注意**
>工具包中还有其他有用的验证器：*ExistingFile*（确保给定的参数是一个存在的文件），*ExistingDirectory*（确保给定的参数是一个存在的目录），*NonexistentPath*（确保给定的参数是一个不存在的路径）。所有这些将参数转换为[本地路径](https://plumbum.readthedocs.io/en/latest/paths.html#guide-paths)。
>

### 可重复的选项
很多时候，你需要在同一个命令行中多次指定某个选项。比如，在 `gcc` 中，你可能使用 `-I` 参数来引入多个目录。默认情况下，选项只能指定一次，除非你给 `switch` 装饰器传递 ` list = True` 参数。
```python
class MyApp(cli.Application):
    _dirs = []

    @cli.switch("-I", str, list = True)
    def include_dirs(self, dirs):
        self._dirs = dirs

    def main(self):
        print(self._dirs)
```
```bash
$ ./example.py -I/foo/bar -I/usr/include
['/foo/bar', '/usr/include']
```
>**注意**
>选项函数只被调用一次，它的参数将会变成一个列表。

### 强制的选项
如果某个选项是必须的，你可以给 `switch` 装饰器传递 `mandatory = True` 来实现。这样的话，如果用户不指定该选项，那么程序是无法运行的。

### 选项依赖
很多时候，一个选项的出现依赖另一个选项，比如，如果不给定 `-y` 选项，那么 `-x` 选项是无法给定的。这种限制可以通过给 `switch` 装饰器传递 `requires` 参数来实现，该参数是一个当前选项所依赖的选项名称列表。如果不指定某个选项所依赖的其他选项，那么用户是无法运行程序的。
```python
class MyApp(cli.Application):
    @cli.switch("--log-to-file", str)
    def log_to_file(self, filename):
        logger.addHandler(logging.FileHandler(filename))

    @cli.switch("--verbose", requires = ["--log-to-file"])
    def verbose(self):
        logger.setLevel(logging.DEBUG)
```
```bash
$ ./example --verbose
Given --verbose, the following are missing ['log-to-file']
```
>**警告**
>选项函数的调用顺序和命令行指定的选项的顺序是一致的。目前不支持在程序运行时计算选项函数调用的拓扑顺序，但是将来会改进。

### 选项互斥
有些选项依赖其他选项，但是有些选项是和其他选项互斥的。比如，`--verbose` 和 `--terse` 同时存在是不合理的。为此，你可以给 `switch` 装饰器指定 `excludes` 列表来实现。
```python
class MyApp(cli.Application):
    @cli.switch("--log-to-file", str)
    def log_to_file(self, filename):
        logger.addHandler(logging.FileHandler(filename))

    @cli.switch("--verbose", requires = ["--log-to-file"], excludes = ["--terse"])
    def verbose(self):
        logger.setLevel(logging.DEBUG)

    @cli.switch("--terse", requires = ["--log-to-file"], excludes = ["--verbose"])
    def terse(self):
        logger.setLevel(logging.WARNING)
```
```bash
$ ./example --log-to-file=log.txt --verbose --terse
Given --verbose, the following are invalid ['--terse']
```

### 选项分组
如果你希望在帮助信息中将某些选项组合在一起，你可以给 `switch` 装饰器指定 `group = "Group Name"`, `Group Name` 可以是任意字符串。当显示帮助信息的时候，所有属于同一个组的选项会被聚合在一起。注意，分组不影响选项的处理，但是可以增强帮助信息的可读性。

### 选项属性
很多时候只需要将选项的参数存储到类的属性中，或者当某个属性给定后设置一个标志。为此，工具包提供了 `SwitchAttr`，这是一个[数据描述符](https://docs.python.org/3/howto/descriptor.html)，用来存储参数。 该工具包还提供了两个额外的 `SwitchAttr`:`Flag`（如果选项给定后，会给其赋予默认值）和 `CountOf` （某个选项出现的次数）。
```python
class MyApp(cli.Application):
    log_file = cli.SwitchAttr("--log-file", str, default = None)
    enable_logging = cli.Flag("--no-log", default = True)
    verbosity_level = cli.CountOf("-v")

    def main(self):
        print(self.log_file, self.enable_logging, self.verbosity_level)
```
```bash
$ ./example.py -v --log-file=log.txt -v --no-log -vvv
log.txt False 5
```

### 环境变量
*新版本 1.6*

你可以使用 `envname` 参数将环境变量作为 `SwitchAttr` 的输入。举例如下：
```python
class MyApp(cli.Application):
    log_file = cli.SwitchAttr("--log-file", str, envname="MY_LOG_FILE")

    def main(self):
        print(self.log_file)
```
```bash
$ MY_LOG_FILE=this.log ./example.py
this.log
```
在命令行给定变量值会覆盖相同环境变量的值。

### Main
一旦当所有命令行参数被处理后 ，`main()` 方法会获取程序的控制，并且可以有任意数量的位置参数，比如，在 `cp -r /foo /bar` 中,  `/foo` 和 `/bar` 是位置参数。程序接受位置参数的数量依赖于 `main()` 函数的声明：如果 `main` 方法有 5 个参数，2 个是有默认值的，那么用户最少需要提供 3 个位置参数并且总数量不能多于 5 个。如果 `main` 方法的声明中使用的是可变参数（`*args`），那么位置参数的个数是没有限制的。
```python
class MyApp(cli.Application):
    def main(self, src, dst, mode = "normal"):
        print(src, dst, mode)
```
```bash
$ ./example.py /foo /bar
/foo /bar normal
$ ./example.py /foo /bar spam
/foo /bar spam
$ ./example.py /foo
Expected at least 2 positional arguments, got ['/foo']
$ ./example.py /foo /bar spam bacon
Expected at most 3 positional arguments, got ['/foo', '/bar', 'spam', 'bacon']
```
> **注意**
> 该方法的声明也用于生成帮助信息，例如：
> ```bash
Usage:  [SWITCHES] src dst [mode='normal']
```

使用可变参数：
```python
class MyApp(cli.Application):
    def main(self, src, dst, *eggs):
        print(src, dst, eggs)
```
```bash
$ ./example.py a b c d
a b ('c', 'd')
$ ./example.py --help
Usage:  [SWITCHES] src dst eggs...
Meta-switches:
    -h, --help                 Prints this help message and quits
    -v, --version              Prints the program's version and quits
```

### 位置验证
*新版本 1.6*

你可以使用 `cli.positional` 装饰器提供的验证器来验证位置参数。只需在装饰器中传递与 `main` 函数中的相匹配的验证器即可。例如：
```python
class MyApp(cli.Application):
    @cli.positional(cli.ExistingFile, cli.NonexistentPath)
    def main(self, infile, *outfiles):
        "infile is a path, outfiles are a list of paths, proper errors are given"
```
如果你的程序只在 Python 3 中运行，你可以使用注解来指定验证器，例如：
```python
class MyApp(cli.Application):
    def main(self, infile : cli.ExistingFile, *outfiles : cli.NonexistentPath):
    "Identical to above MyApp"
```
如果 `positional` 装饰器存在，那么注解会被忽略。

### 子命令
*新版本 1.1*

随着 CLI 应用程序的扩展，功能变的越来越多，一个通常的做法是将其逻辑分成多个子应用（或者子命令）。一个典型的例子是版本控制系统，比如 [**git**](https://git-scm.com/)，`git` 是根命令，在这之下的子命令比如 `commit` 或者 `push` 是嵌套的。`git` 甚至支持命令别名，这运行用户自己创建一些子命令。Plumbum 写类似这样的程序是很轻松的。

在我们开始了解代码之前，先强调两件事情：
* 在 Plumbum中，每个子命令都是一个完整的 `cli.Application` 应用，你可以单独执行它，或者从所谓的根命令中分离出来。当应用程序单独执行是，它的父属性是 `None`，当以子命令运行时，它的父属性指向父应用程序。同样，当父应用使用子命令执行时，它的内嵌命令被设置成内嵌应用。
* 每个子命令只负责它自己的选项参数（直到下一个子命令）。这允许应用在内嵌应用调用之前来处理它自己的选项和位置参数。例如 `git --foo=bar spam push origin --tags`：根应用 `git` 负责选项 `--foo` 和位置选项 `spam` ，内嵌应用 `push` 负责在它之后的参数。从理论上讲，你可以将多个子应用程序嵌套到另一个应用程序中，但在实践中，通常嵌套层级只有一层。

这是一个模仿版本控制系统的例子 `geet`。我们有一个根应用 `Geet` ，它有两个子命令 `GeetCommit` 和 `GeetPush`：这两个子命令通过 `subcommand` 装饰器来将其附加到根应用。
```python
class Geet(cli.Application):
    """The l33t version control"""
    VERSION = "1.7.2"

    def main(self, *args):
        if args:
            print("Unknown command {0!r}".format(args[0]))
            return 1   # error exit code
        if not self.nested_command:           # will be ``None`` if no sub-command follows
            print("No command given")
            return 1   # error exit code

@Geet.subcommand("commit")                    # attach 'geet commit'
class GeetCommit(cli.Application):
    """creates a new commit in the current branch"""

    auto_add = cli.Flag("-a", help = "automatically add changed files")
    message = cli.SwitchAttr("-m", str, mandatory = True, help = "sets the commit message")

    def main(self):
        print("doing the commit...")

@Geet.subcommand("push")                      # attach 'geet push'
class GeetPush(cli.Application):
    """pushes the current local branch to the remote one"""
    def main(self, remote, branch = None):
        print("doing the push...")

if __name__ == "__main__":
    Geet.run()
```
>**注意**
>* 由于 `GeetCommit` 也是一个 `cli.Application`，因此你可以直接调用 `GeetCommit.run()` （这在应用的上下文是合理的）
>* 你也可以不用装饰器而使用 `subcommand` 方法来附加子命令：`Geet.subcommand("push", GeetPush)`

以下是运行该应用程序的示例：
```bash
$ python geet.py --help
geet v1.7.2
The l33t version control

Usage: geet.py [SWITCHES] [SUBCOMMAND [SWITCHES]] args...
Meta-switches:
    -h, --help                 Prints this help message and quits
    -v, --version              Prints the program's version and quits

Subcommands:
    commit                     creates a new commit in the current branch; see
                               'geet commit --help' for more info
    push                       pushes the current local branch to the remote
                               one; see 'geet push --help' for more info

$ python geet.py commit --help
geet commit v1.7.2
creates a new commit in the current branch

Usage: geet commit [SWITCHES]
Meta-switches:
    -h, --help                 Prints this help message and quits
    -v, --version              Prints the program's version and quits

Switches:
    -a                         automatically add changed files
    -m VALUE:str               sets the commit message; required

$ python geet.py commit -m "foo"
doing the commit...
```

### 配置解析器
应用程序的另一个常见的功能是配置文件解析器，解析后台 INI 配置文件：`Config` （或者 `ConfigINI`）。使用示例：
```python
from plumbum import cli

with cli.Config('~/.myapp_rc') as conf:
    one = conf.get('one', '1')
    two = conf.get('two', '2')
```
如果配置文件不存在，那么将会以当前的 `key` 和默认的 `value` 来创建一个配置文件，在调用 `.get` 方法时会得到默认值，当上下文管理器存在时，文件会被创建。如果配置文件存在，那么该文件将会被读取并且没有任何改变。你也可以使用 `[]` 语法来强制设置一个值或者当变量不存在时获取到一个 `ValueError`。如果你想避免上下文管理器，你也可以使用 `.read` 和 `.write`。

ini 解析器默认使用 `[DEFAULT]` 段，就像 Python 的 ConfigParser。如果你想使用一个不同的段，只需要在 key 中通过 `.` 将段和标题分隔开。比如 `conf['section.item']` 会将 `item` 放置在 `[section]` 下。所有存储在 `ConfigINI` 中的条目会被转化成 `str`，`str` 是经常返回的。

### 终端实用程序
在 `plumbum.cli.terminal` 中有多个终端实用程序，用来帮助制作终端应用程序。

`get_terminal_size(default=(80,25))` 允许跨平台访问终端屏幕大小，返回值是一个元组 `(width, height)`。还有几个方法可以用来询问用户输入，比如 `readline`, `ask`, `choose` 和 `prompt` 都是可用的。

`Progress(iterator)` 可以使你快速地从迭代器来创建一个进度条。简单地打包一个 slow 迭代器并迭代就会生成一个不错的基于用户屏幕宽度的文本进度条，同时会显示剩余时间。如果你想给 fast 迭代器创建一个进度条，并且在循环中包含代码，那么请使用 `Progress.wrap` 或者 `Progress.range`。例如：
```python
for i in Progress.range(10):
    time.sleep(1)
```
如果在终端中有其他输出，但是仍然需要一个进度条，请传递 `has_output=True` 参数来禁止进度条清除掉历史输出。

在 `plumbum.cli.image` 中提供了一个命令行绘图器（`Image`）。它可以绘制一个类似 PIL 的图像：
```python
Image().show_pil(im)
```
Image 构造函数接受一个可选的 size 参数（如果是 None，那么默认是当前终端大小）和一个字符比例，该比例来自当前字符的高度和宽度的度量，默认值是 2.45。如果设置为 None，ratio 将会被忽略，图像不再被限制成比例缩放。要直接绘制一个图像，`show` 需要一个文件名和一对参数。`show_pil` 和 `show_pil_double` 方法直接接受一个 PIL-like 对象。为了从命令行绘制图像，该模块可以直接被运行：`python -m plumbum.cli.image myimage.png`。

要获取帮助列表和更多的信息请参见 [**api docs**](https://plumbum.readthedocs.io/en/latest/api/cli.html#api-cli)。

### 请参阅
* [**filecopy.py**](https://github.com/tomerfiliba/plumbum/blob/master/examples/filecopy.py) 示例
* [**geet.py**](https://github.com/tomerfiliba/plumbum/blob/master/examples/geet.py) - 一个可运行的使用子命令的示例
* [**RPyC**](http://rpyc.readthedocs.io/en/latest/) 已经将基于 bash 的编译脚本换成了 Plumbum CLI。这是多么[**简短和具有可读性**](https://github.com/tomerfiliba/rpyc/blob/c457a28d689df7605838334a437c6b35f9a94618/build.py)
* 一篇[**博客**](http://tomerfiliba.com/blog/Plumbum/)，讲述 CLI 模块的理论
