# 配置状态

在执行 Python 代码之前，CPython 首先会对运行时及用户选项进行配置。
运行时的配置存在于以下三个数据结构中，数据结构在 [PEP587](https://peps.python.org/pep-0587/) 中定义：

1.  [PyPreConfig](https://github.com/python/cpython/blob/v3.9.0b1/Include/cpython/initconfig.h#L125)，用于预初始化配置
2.  [PyConfig](https://github.com/python/cpython/blob/v3.9.0b1/Include/cpython/initconfig.h#L416)，用于运行时配置
3.  CPython 解释器编译选项的配置

上述的数据结构都在 Include/cpython/initconfig.h 文件中定义。

## 预初始化配置

预初始化配置与运行时配置是分开的，因为它的属性与操作系统或用户环境相关。

`PyPreConfig` 最主要的三个职能如下：

- 设置 Python 内存分配器
- 配置 `LC_CTYPE` 语言环境
- 设置 UTF-8 模式（[PEP540](https://www.python.org/dev/peps/pep-0540/)）

`PyPreConfig` 结构体包含以下属性，都是 `int` 类型：

| 名称                       | 作用                                                         |
| :------------------------- | ------------------------------------------------------------ |
| allocator                  | 内存分配器的名称（例如：`PYMEM_ALLOCATOR_MALLOC`）。运行 `./configure --help` 可获取有关内存分配器的详细信息 |
| configure_locale           | 将 `LC_CTYPE` 语言环境设置为用户首选语言环境。如果等于 0，则将 `coerce_c_locale` 和 `coerce_c_locale_warn` 设置为 0 |
| coerce_c_locale            | 如果等于 2，则强制使用 C 语言环境； 如果等于 1，则读取 `LC_CTYPE` 的设置来决定是否强制使用 C 语言环境。 |
| coerce_c_locale_warn       | 如果非零，在强制使用 C 语言环境时会发出告警。                |
| dev_mode                   | 参见 `PyConfig.dev_mode`。                                     |
| isolated                   | 启用隔离模式：`sys.path` 不包含脚本的目录或用户的 site-packages 目录 |
| legacy_windows_fs_encoding | （只在 windows 下使用）如果非零禁用 UTF-8 模式，设置 Python 文件系统编码模式为 **mbcs**。 |
| parse_argv                 | 如果非零，`Py_PreInitializeFromArgs()` 和 `Py_PreInitializeFromBytesArgs()` 将会解析从命令行输入的参数。 |
| use_environment            | 参见 `PyConfig.use_environment`。                              |
| utf8_mode                  | 如果非零，启用 UTF-8 模式。                                  |



## 相关的源文件

与 `PyPreConfig` 相关的源文件如下：

| 文件目录                     | 作用                                                     |
| ---------------------------- | -------------------------------------------------------- |
| Python/initconfig.c          | 从系统环境加载配置，并将其与命令行中输入的信息进行合并。 |
| Include/cpython/initconfig.h | 定义了初始化配置的数据结构。                             |



## 运行时配置数据结构

第二阶段配置是运行时配置。[PyConfig](https://github.com/python/cpython/blob/v3.9.0b1/Include/cpython/initconfig.h#L416) 中的运行时配置数据结构包含如下值：

- 调试和优化等模式的运行时标志
- 执行模式，例如脚本文件，标准输入或者一个模块
- 扩展选项，通过 `-X <选项值>` 的方式来指定
- 运行时设置的环境变量

CPython 运行时使用配置数据来启用和禁用某项功能。

## 通过命令行设置运行时配置

Python 还附带了几个[命令行接口选项](https://docs.python.org/3/using/cmdline.html)。
例如，CPython 有个名为 **verbose** 的模式，这个模式主要面向开发者，用于调试 CPython。
在 Python 中可以通过 `-v` 标志启用 verbose 模式，在这种模式下，模块被加载时会有信息打印在屏幕上：

```bash
$ ./python -v -c "print('hello world')"
# installing zipimport hook
import zipimport # builtin
# installed zipimport hook
...
```

可以看到在屏幕上会打印出上百条甚至更多的模块加载信息，包含用户 site-packages 和其他在系统环境中的模块。
由于运行时配置可以通过多种方式进行设置，因此各个配置设置之间是有优先顺序的。verbose 模式的优先顺序如下：

1. `config->verbose` 的默认值是 -1，硬编码在源码中；
2. 环境变量 PYTHONVERBOSE 用来设置 `config->verbose` 的值；
3. 如果环境变量不存在，将会保留默认值 -1；
4. 在 Python/initconfig.c 文件的 [config_parse_cmdline()](https://github.com/python/cpython/blob/v3.9.0b1/Python/initconfig.c#L1875) 方法中，如果有提供命令行标志的话也会被用来设置值；
5. 该值将会被 [_Py_GetGlobalVariablesAsDict()](https://github.com/python/cpython/blob/v3.9.0b1/Python/initconfig.c#L167) 方法拷贝到一个全局变量 `Py_VerboseFlag` 中。

所有的 `PyConfig` 值都遵循相同的排列和优先顺序：

![图6.1.1 运行时配置顺序](<../.gitbook/assets/图6.1.1 运行时配置顺序.png>)

## 查看运行时配置

CPython 解释器有一组运行时标志，这些标志是用于切换 CPython 特定行为的高级功能。 在 Python 会话中，你可以使用名为 `sys.flags` 的 tuple 访问运行时标志，例如 verbose 模式和 quiet 模式。所有通过 `-X` 指定的标志都会在 `sys._xoptions` 字典中生效：

```bash
$ ./python -X dev -q
>>> import sys
>>> sys.flags
sys.flags(debug=0, inspect=0, interactive=0, optimize=0,
dont_write_bytecode=0, no_user_site=0, no_site=0,
ignore_environment=0, verbose=0, bytes_warning=0,
quiet=1, hash_randomization=1, isolated=0,
dev_mode=True, utf8_mode=0)
>>> sys._xoptions
{'dev': True}
```
