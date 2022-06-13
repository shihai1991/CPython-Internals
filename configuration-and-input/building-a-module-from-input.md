# 从输入构建模块

任何代码在被执行之前，必须从输入编译成一个模块。正如之前所讨论的那样，输入的类型可以有多种：

- 本地的文件和包
- 输入/输出流，例如标准输入或内存管道 
- 字符串

输入的内容被读取之后会被传递给解析器，再之后是编译器：

![图6.3.1 输入的传递过程](<../.gitbook/assets/图6.3.1 输入的传递过程.png>)

正是由于输入类型的灵活性，因此有很大一部分的 CPython 源码专门用于处理 CPython 解析器的输入。 

## 相关的源文件

用于处理命令行界面的主要有两个文件：

| 文件               | 功能                                                         |
| ------------------ | ------------------------------------------------------------ |
| Lib/runpy.py       | 标准库模块，用于导入 Python 模块并执行                       |
| Modules/main.c     | 为外部代码执行进行函数包装，外部代码来源例如文件、模块或输入流 |
| Programs/python.c  | python 可执行文件的入口，适用于 Windows，Linux 和 macOS 系统，仅用作 Modules/main.c 的装饰器 |
| Python/pythonrun.c | 对内部 C API 进行函数包装以处理来自命令行的输入              |



## 读取文件/输入

一旦 CPython 有了运行时配置和命令行参数，它就可以加载它所需要执行的代码，这个任务由 Modules/main.c 文件中的 [pymain_main()](https://github.com/python/cpython/blob/v3.9.0b1/Modules/main.c#L651) 函数来完成。
基于新创建的 PyConfig 实例，CPython 就可以执行通过多种方式所提供的代码了。

## 命令行输入字符串

CPython 可以通过指定 `-c` 选项，从而通过命令行模式来执行一个小的 Python 应用，例如想要执行 `print(2 ** 2)` 可以通过如下方式：

```bash
$ ./python -c "print(2 ** 2)"
4
```

命令行中通过 `-c` 参数传入的内容会被传递给 Modules/main.c 文件中 [pymain_run_command()](https://github.com/python/cpython/blob/v3.9.0b1/Modules/main.c#L226) 函数的 `wchar_t*` 类型的参数。

{% hint style="info" %}
**Note**

`wchar_t* ` 类型通常作为 CPython 中 Unicode 数据的低级存储类型，因为这种类型的大小可以存储 UTF8 字符。

将 `wchar_t` 类型转换成 Python 字符串时，Objects/unicodeobject.c 文件中有一个辅助函数叫做 [PyUnicode_FromWideChar()](https://github.com/python/cpython/blob/v3.9.0b1/Objects/unicodeobject.c#L2183) 会返回Unicode字符串，之后由 `PyUnicode_AsUTF8String()` 完成 UTF8 编码。
Python 的 Unicode 字符串在《Unicode 字符串类型》章节和《对象与类型》章节有深入的介绍。

{% endhint %}

完成此操作后，[pymain_run_command()](https://github.com/python/cpython/blob/v3.9.0b1/Modules/main.c#L226) 将会把 Python 字节对象传递给 [PyRun_SimpleStringFlags()](https://github.com/python/cpython/blob/v3.9.0b1/Python/pythonrun.c#L463) 用于执行。
[PyRun_SimpleStringFlags()](https://github.com/python/cpython/blob/v3.9.0b1/Python/pythonrun.c#L463) 函数是 Python/pythonrun.c 文件的一个部分，它的目的是将一个字符串转换成 Python 模块，之后把它送去执行。每个 Python 模块都需要有一个入口点 （[__main__](https://realpython.com/python-main-function/)），才能作为独立模块执行。`PyRun_SimpleStringFlags()` 函数将会隐式地创建入口点。
一旦 [PyRun_SimpleStringFlags()](https://github.com/python/cpython/blob/v3.9.0b1/Python/pythonrun.c#L463)  创建了模块和字典之后就会调用 [PyRun_StringFlags()](https://github.com/python/cpython/blob/v3.9.0b1/Python/pythonrun.c#L1054) 。[PyRun_SimpleStringFlags()](https://github.com/python/cpython/blob/v3.9.0b1/Python/pythonrun.c#L463) 创建了一个假的文件名，之后调用 Python 解析器从字符串创建一个 AST （抽象语法树）并且返回一个模块。

{% hint style="info" %}
**Note**

Python 模块是用于将解析后的代码交给编译器的数据结构。Python 模块的 C 数据结构名为 `mod_ty`，在 Include/Python-ast.h 文件中定义

{% endhint %}

## 本地模块输入

执行 Python 命令的另一种方式是通过 `-m` 选项指定模块名，一个典型的例子是通过 `python -m unittest` 在标准库中运行 unittest 模块。
以脚本的形式来执行模块这个想法最初是在 [PEP338](https://peps.python.org/pep-0338/) 中被提出的，明确的相对导入的标准是在 [PEP366](https://peps.python.org/pep-0366/) 中定义的。
 `-m` 标志的使用意味着在您想要执行模块包中入口点（[__main__](https://realpython.com/python-main-function/)）内的所有内容，它也是表示您要在 `sys.path` 中搜索命名模块。
正式由于导入库（`importlib`）中的这种搜索机制，因此我们不需要记住 `unittest` 模块在文件系统中所存储的位置。
CPython 导入一个标准库模块 `runpy` 并通过 [PyObject_Call()](https://github.com/python/cpython/blob/v3.9.0b1/Objects/call.c#L289) 来执行它，导入的过程由一个名为 [PyImport_ImportModule()](https://github.com/python/cpython/blob/v3.9.0b1/Python/import.c#L1477) 的 C API 函数完成，该函数在 Python/import.c 文件中定义。

{% hint style="info" %}
**Note**

在 Python 中，如果你有一个对象并且想要获取其属性，可以调用 `getattr()` 。而在 C API 中，需要调用的是 [PyObject_GetAttrString()](https://github.com/python/cpython/blob/v3.9.0b1/Objects/object.c#L786)，该方法在 Objects/object.c 文件中定义。如果你想运行一个可调用的方法，你可以给它加上括号，或者你可以在任何 Python 对象上调用  __call__()  属性。__call__() 方法在 Objects/object.c 文件中实现：

```bash
>>> my_str = "hello world!"
>>> my_str.upper()
'HELLO WORLD!'
>>> my_str.upper.__call__()
'HELLO WORLD!'
```

{% endhint %}

`runpy` 模块在 Lib/runpy.py 文件中定义，是用纯 Python 编写的。
执行 `python -m <module>` 相当于运行 `python -m runpy <module>`。 创建 `runpy` 模块是为了将操作系统上定位和执行模块的过程抽象出来。
为了运行目标模块，`runpy` 做了以下这些工作：

- 为指定的模块名调用 __import__() 方法
- 将 __name__（模块名称）设置到名为 __main__ 的命名空间
- 在 __main__ 命名空间中执行模块

`runpy` 模块还支持执行目录和 zip 文件。

## 来自脚本文件或标准输入的输入

如果 python 执行时的第一个参数是一个文件名，例如 `python test.py`，CPython 将会打开一个文件句柄并将句柄传递给 [PyRun_SimpleFileExFlags()](https://github.com/python/cpython/blob/v3.9.0b1/Python/pythonrun.c#L382)，该方法在 Python/pythonrun.c 文件中定义。
这个方法可以处理三种类型的文件路径：

1. 如果文件路径是 `.pyc` 文件，将会调用 [run_pyc_file()](https://github.com/python/cpython/blob/v3.9.0b1/Python/pythonrun.c#L1205)；
2. 如果文件路径是脚本文件（`.py`），将会调用 [PyRun_FileExFlags()](https://github.com/python/cpython/blob/v3.9.0b1/Python/pythonrun.c#L1085)；
3. 如果文件路径是 `stdin`，例如用户执行了 `command | python`，那么会将 `stdin` 作为一个文件句柄并调用 [PyRun_FileExFlags()](https://github.com/python/cpython/blob/v3.9.0b1/Python/pythonrun.c#L1085)。

对于 `stdin` 和基本的脚本文件，CPython 会将文件句柄传递给 Python/pythonrun.c 文件中定义的 [PyRun_FileExFlags()](https://github.com/python/cpython/blob/v3.9.0b1/Python/pythonrun.c#L1085) 函数。

[PyRun_FileExFlags()](https://github.com/python/cpython/blob/v3.9.0b1/Python/pythonrun.c#L1085) 的目的类似于 [PyRun_SimpleStringFlags()](https://github.com/python/cpython/blob/v3.9.0b1/Python/pythonrun.c#L463)。CPython 会将文件句柄加载到[PyParser_ASTFromFileObject()](https://github.com/python/cpython/blob/v3.9.0b1/Python/pythonrun.c#L1442) 中。

与 [PyRun_SimpleStringFlags()](https://github.com/python/cpython/blob/v3.9.0b1/Python/pythonrun.c#L463) 相同，一旦 [PyRun_FileExFlags()](https://github.com/python/cpython/blob/v3.9.0b1/Python/pythonrun.c#L1085) 从文件中创建了一个 Python 模块，就会将它发给 [run_mod()](https://github.com/python/cpython/blob/v3.9.0b1/Python/pythonrun.c#L1186) 去执行。

## 从编译好的字节码输入

如果用户为 python 可执行程序传递了一个 `.pyc` 文件，那么 CPython 不会将文件作为纯文本文件加载并解析它，而是假定 `.pyc` 文件包含一个写入磁盘的 `code object`。
在 [PyRun_SimpleFileExFlags()](https://github.com/python/cpython/blob/v3.9.0b1/Python/pythonrun.c#L382) 中，有一个子句用于用户提供 `.pyc` 文件的文件路径。
Python/pythonrun.c 文件中的 [run_pyc_file()](https://github.com/python/cpython/blob/v3.9.0b1/Python/pythonrun.c#L1205) 函数使用文件句柄从 `.pyc` 文件中 marshals `code object`，磁盘上的 `code object` 数据结构是 CPython 编译器缓存已编译的代码的方式，这样它就不需要在脚本每次被调用时都去解析一次。

{% hint style="info" %}
**Note**

**Marshaling** 的意思是将一个文件的内容复制到内存并将它们转换为特定的数据结构。

{% endhint %}

一旦 `code object` 被 marshal 到内存中，它就会被送到 [run_eval_code_obj()](https://github.com/python/cpython/blob/v3.9.0b1/Python/pythonrun.c#L1155)，该函数接着会调用 Python/ceval.c 来执行代码。