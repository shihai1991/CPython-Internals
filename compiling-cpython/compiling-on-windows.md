# 4.6 在 Windows 上编译 CPython

有两种方法可以从 Windows 编译 CPython 二进制文件和库。

第一种从命令行编译，这需要用到 Visual Studio 自带的 Microsoft Visual C++ 编译器。第二种是通过 Visual Studio 打开 `PCBuild/pcbuild.sln` 直接进行构建。

### 安装依赖项

不论使用命令行还是 Visual Studio 进行编译，你都需要安装一些外部工具、库和 C 头文件。

在 `PCBuild` 文件夹中有一个 `.bat` 文件，可以为你自动执行上述安装操作。

在 `PCBuild` 中打开命令行并执行 `get_externals.bat`：

```
> get_externals.bat
Using py -3.7 (found 3.7 with py.exe)
Fetching external libraries...
Fetching bzip2-1.0.6...
Fetching sqlite-3.28.0.0...
Fetching xz-5.2.2...
Fetching zlib-1.2.11...
Fetching external binaries...
Fetching openssl-bin-1.1.1d...
Fetching tcltk-8.6.9.0...
Finished.
```

现在你可以从命令行或 Visual Studio 进行编译了。

### 从命令行编译

从命令行进行编译时，需要选择要编译的 CPU 架构。默认值为 win32，但你可能需要 64 位 (amd64) 二进制文件。

如果你进行任何调试，调试版本提供了在源代码中添加断点的能力。要构建调试版本，请添加 `-c Debug` 以指定调试配置。

默认情况下，`build.bat` 将获取外部依赖项，但由于我们已经完成了该步骤，它会打印一条跳过下载的消息：

```
> build.bat -p x64 -c Debug
```

此命令将生成 Python 二进制文件 `PCbuild/amd64/python_d.exe`。该二进制文件可以直接从命令行启动：

```
> amd64\python_d.exe

Python 3.9.0b1 (tags/v3.9.0b1:97fe9cf, May 19 2020, 10:00:00)
[MSC v.1922 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

现在已经进入了你编译的 CPython 二进制文件的 REPL 中。

要编译二进制文件发行版：

```
> build.bat -p x64 -c Release
```

上述命令将生成二进制文件 `PCbuild/amd64/python.exe`。

{% hint style="info" %}
后缀 `_d` 表明 CPython 是基于 `Debug` 配置构建的。

python.org 上发布的二进制文件在基于 Profile-Guided-Optimization (PGO) 配置编译的。有关 PGO 的更多详细信息，请参阅本章末尾的 Profile-Guided-Optimization (PGO) 部分。
{% endhint %}

#### 参数

build.bat 支持以下参数：

| 标记 | 作用             | 可选值                                               |
| -- | -------------- | ------------------------------------------------- |
| -p | 指定构建平台的 CPU 架构 | `x64`，`Win32` (默认值)，`ARM`，`ARM64`                 |
| -c | 指定构建配置         | `Release` (默认值)，`Debug`，`PGInstrument`，`PGUpdate` |
| -t | 指定构建目标         | `Build` (默认值)，`Rebuild`，`Clean`，`CleanAll`        |

#### 标记

下面是一些可用于 `build.bat` 的可选标志。如需完整列表，请运行 `build.bat -h`。

| 标记      | 作用                         |
| ------- | -------------------------- |
| -v      | 详细模式。在构建期间显示信息性消息          |
| -vv     | 非常详细的模式。在构建期间显示详细消息        |
| -q      | 安静模式。仅在构建期间显示警告和错误         |
| -e      | 下载并安装外部依赖项（默认）             |
| -E      | **不**下载和安装外部依赖项            |
| --pgo   | 编译时开启 PGO                  |
| --regen | 当更新语言时，使用该选项重新生成新语法和 token |

### 从 Visio Studio 编译
