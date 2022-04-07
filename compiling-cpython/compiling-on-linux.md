# 在 Linux 上编译 CPython

要在 Linux 上编译 CPython，首先需要下载并安装 make、gcc、configure 和 pkgconfig。

对于 Fedora Core、RHEL、CentOS 或其他基于 `yum` 命令的系统：

```bash
$ sudo yum install yum-utils
```

对于 Debian、Ubuntu 或其他基于 `apt` 命令的系统：

```bash
$ sudo apt install build-essential
```

接下来安装一些所需的其他包。

对于 Fedora Core、RHEL、CentOS 或其他基于 `yum` 命令的系统：

```bash
$ sudo yum-builddep python3
```

对于 Debian、Ubuntu 或其他基于 `apt` 命令的系统：

```bash
$ sudo apt install libssl-dev zlib1g-dev libncurses5-dev \
libncursesw5-dev libreadline-dev libsqlite3-dev libgdbm-dev \
libdb5.3-dev libbz2-dev libexpat1-dev liblzma-dev libffi-dev
```

安装了依赖项后，你可以运行配置脚本了，选项 `--with-pydebug` 能帮助启用调试钩子：

```bash
$ ./configure --with-pydebug
```

接下来，你可以通过运行上一步生成的 Makefile 来构建 CPython 二进制文件：

```bash
$ make -j2 -s
```

{% hint style="info" %}
**See Also**

有关 `make` 选项的的详细介绍，请参考章节编译命令快速入门。
{% endhint %}

查看输出二进制文件中的 `_ssl` 模块是否有问题。 若有问题，请检查你的发行版以获取有关安装 OpenSSL 头文件的说明。

在构建过程中你可能会收到一些错误提示。若有的包未成功构建，`make` 会在构建摘要中通知你。如果你的开发计划中不涉及这些包，那也没关系。但如果需要用到这些包，请查看这些包的详细信息。

构建过程会持续几分钟时间，并生成一个二进制文件 `python`。这是 CPython 的调试二进制文件。执行 `./python` 可以查看运行中的 REPL：

```bash
$ ./python
Python 3.9.0b1 (tags/v3.9.0b1:97fe9cf, May 19 2020, 10:00:00)
[Clang 10.0.1 (clang-1001.0.46.4)] on Linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```
