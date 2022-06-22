# 在 macOS 上编译 CPython

在 macOS 上编译 CPython 需要一些额外的应用程序和库。首先需要安装基本的 C 编译器工具包。“命令行开发工具”是一个可以在 macOS 的 App Store 中进行更新的应用程序。你需要在终端上进行一些初始安装。

{% hint style="info" %}
**Note**

依次点击 Applications -> Other -> Terminal 可在 macOS 中打开终端。若想在将这个应用保存在 dock 栏中，请右击图标并选择 Keep in Dock。
{% endhint %}

在终端中，通过运行以下命令安装 C 编译器和工具包：

```bash
$ xcode-select --install
```

此命令将弹出下载和安装一组工具的提示，包括 Git、Make 和 GNU C 编译器。

从网站 [PyPi.org](https://pypi.org) 上获取包时需要用到 [OpenSSL](https://www.openssl.org) 。如果你后面计划基于这次构建的版本去安装其他包，那么 SSL 认证是必须的。

在 macOS 上安装 OpenSSL 最简单的方式是使用 [Homebrew ](https://brew.sh)。

{% hint style="info" %}
**Note**

如果你没有 Homebrew，你可以使用如下命令直接从 GitHub 下载并安装 Homebrew：

```bash
$ /usr/bin/ruby -e "$(curl -fsSL \
https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
{% endhint %}

安装了 Homebrew 之后，可以使用 `brew install` 命令来安装 CPython 的依赖项。

```bash
$ brew install openssl xz zlib gdbm sqlite
```

现在你已经安装了依赖项，接下来可以运行配置脚本了。Homebrew 的命令 `brew --prefix [package]` 能返回指定包的安装目录。你可以在编译时通过使用 Homebrew 提供的路径来启用对 SSL 的支持。

若你出于开发或测试的目的需要进行调试，请添加选项 `--with-pydebug`。调试 CPython 的内容将在调试一章中详细介绍。

配置阶段只需要运行一次，请在运行配置命令时指定 zlib 包的位置：

```bash
$ CPPFLAGS="-I$(brew --prefix zlib)/include" \
LDFLAGS="-L$(brew --prefix zlib)/lib" \
./configure --with-openssl=$(brew --prefix openssl) --with-pydebug
```

配置命令运行后会在 cpython 目录中生成一个 Makefile，你可以使用该 Makefile 来自动化构建过程。

现在，你可以通过运行以下命令来构建 CPython 二进制文件：

```bash
make -j2 -s
```

{% hint style="info" %}
**See Also**

有关 `make` 选项的的详细介绍，请参考章节编译命令快速入门。
{% endhint %}

在构建过程中你可能会收到一些错误提示。若有的包未成功构建，`make` 会在构建摘要中通知你。例如，ossaudiodev、spwd 和 \_tkinter 将无法使用这组指令构建。如果你的开发计划中不涉及这些包，那也没关系。但如果需要用到这些包，请查看[官方开发指南](https://devguide.python.org)网站了解更多信息。

构建过程会持续几分钟时间，并生成一个二进制文件 `python.exe`。每次更改源代码后，你都需要使用相同的选项重新运行 `make`。二进制文件 `python.exe` 是 CPython 的调试二进制文件。执行 `python.exe` 可以查看运行中的 REPL：

```bash
$ ./python.exe
Python 3.9.0b1 (tags/v3.9.0b1:97fe9cf, May 19 2020, 10:00:00)
[Clang 10.0.1 (clang-1001.0.46.4)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

{% hint style="info" %}
**Important**

是的，没错，macOS 构建的结果会有 .exe 的文件扩展名。出现这个扩展名不是因为它是 Windows 二进制文件！而是由于 macOS 的文件系统不区分大小写，开发人员不希望人们在运行二进制文件时意外地引用了目录 Python/，因此附加了 .exe 以避免歧义。如果你稍后运行 make install 或 make altinstall，你会发现文件在安装到系统之前被重命名回 python。
{% endhint %}
