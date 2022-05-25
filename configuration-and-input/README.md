# 配置和输入

现在你已经了解了 Python 的语法，是时候探索一下代码是如何将输入的内容转换成可执行状态的了。

在 CPython 中运行 Python 代码的方式有很多，下面是一些最常用的方法：

1. 通过 `python -c` 命令加一个 Python 字符串来运行；
2. 通过 `python -m` 命令加一个模块名称来运行；
3. 通过 `python [file]` 运行，其中 `[file]` 为文件的具体路径，并且文件中包含 Python 代码；
4. 通过标准输入将 Python 代码通过管道传输到可执行文件中，例如 `cat [file] | python`；
5. 通过启动REPL（Read Eval Print Loop:交互式解释器），一次执行一个命令；
6. 使用 C API，并用 Python 作为嵌入式环境。

{% hint style="info" %}
**See Also**

Python 有这么多执行脚本的方法，信息量比较大可能一时难以消化。如果你想要学习更多的细节，可以参考 Darren Jones [在 realpython.com 上整理的一个关于运行 Python 脚本的很棒的课程](https://realpython.com/courses/running-python-scripts/)。
{% endhint %}

在执行任何的 python 代码之前，解释器都需要：

- 一个待执行的模块
- 保存变量等信息的状态
- 配置，例如开启了哪些选项

有了这三个组件，解释器就可以执行代码并且输出 ：

![图6.1 CPython解释器三个模块](<../.gitbook/assets/图6.1 CPython解释器三个模块.png>)

{% hint style="info" %}
**Note**

与 Python 的编码规范 [PEP8](https://realpython.com/courses/writing-beautiful-python-code-pep-8/) 类似，CPython 中 C 代码的编码规范叫做 [PEP7](https://peps.python.org/pep-0007/)。以下是一些针对 C 源码的命名标准：

- 公共函数使用 `Py` 前缀，静态函数不使用 `Py` 前缀。`Py_` 前缀是为全局服务例行程序保留的，如 `Py_FatalError`。特定的例行程序组（如特定对象类型的 API）使用更长的前缀，例如字符串函数会使用 `PyString_` 前缀。
- 公共函数和变量的命名规则是大小写混合并带有下划线，例如：`PyObject_GetAttr()`、`Py_BuildValue()`、`PyExc_TypeError()` 。
- 有些情况下，“内部”函数必须对加载器是可视的，可以使用 `_Py` 作为前缀，例如`_PyObject_Dump()`。
- 宏的前缀应是大小写混合的，之后的部分全部使用大写字母，例如`PyString_AS_STRING`，`Py_PRINT_RAW `。

与 PEP8 不同的是，用于检查代码规则是否符合 PEP7 的工具很少。因此这部分合规性的检查任务就需要通过核心开发者参与到代码检视中来完成。与其他任何人工操作的流程一样，代码也不会是完美的，总会有不符合 PEP7 规则的地方。

唯一用来检查 PEP7 合规性的工具是一个名为 `smelly.py` 的脚本，你可以在 Linux 或 macOS 上通过 `make smelly` 执行，或通过以下命令行执行：

```bash
$ ./python Tools/scripts/smelly.py
```

一旦发现任何属于 `libpython` （共享CPython库）的不以 `Py` 或 `_Py` 开头的符号，脚本就会报错。

{% endhint %}