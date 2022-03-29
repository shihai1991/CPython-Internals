# Python 和 C 模块

其余模块是用 C，或者是 Python 和 C 组合编写的。这些模块源代码的 Python 部分在 Lib 文件夹中，C 的部分在 Modules 文件夹中。不过有两个例外，`sys` 模块在 `Python/sysmodule.c` 中，`__builtins__` 模块在 `Python/bltinmodule.c` 中。

当解释器实例化时，Python 会执行 `import * from __builtins__`，因此所有的内置函数，比如 `print()`、`chr()`、`format()` 等都可以在 `Python/bltinmodule.c` 中找到。

因为 `sys` 模块对 CPython 的解释器和内部实现来说非常特殊，所以 `sys` 模块被放在 `Python` 目录中。可以认为 `sys` 模块是 CPython 的实现细节，而这些实现细节在其他的 Python 实现中可能并不存在。

内置的 `print()` 函数或许是你在 Python 中学会的第一件事。那么当你输入 `print("hello world!")` 后，会发生什么？

1. 参数 `"hello world"` 被编译器从一个字符串常量转换成 `PyUnicodeObject`
2. 这个 `PyUnicodeObject` 作为参数传入 `builtin_print()`，`builtin_print` 的 `kwnames` 为 `NULL`
3. 变量 `file` 被设置成 `PyId_stdout`，也即系统的 `stdout`
4. 每个参数被送往 `file`
5. 一个换行符 `\n` 被送往 `file`

[`Python/bltinmodule.c` 第 1828 行](https://github.com/python/cpython/blob/v3.9.0b1/Python/bltinmodule.c#L1828):

```c
static PyObject *
builtin_print(PyObject *self, PyObject *const *args,
    Py_ssize_t nargs, PyObject *kwnames)
{
    ...
    if (file == NULL || file == Py_None) {
        file = _PySys_GetObjectId(&PyId_stdout);
        ...
    }
    ...
    for (i = 0; i < nargs; i++) {
        if (i > 0) {
            if (sep == NULL)
                err = PyFile_WriteString(" ", file);
            else
                err = PyFile_WriteObject(sep, file,
                                         Py_PRINT_RAW);
        if (err)
            return NULL;
        }
        err = PyFile_WriteObject(args[i], file, Py_PRINT_RAW); if (err)
        return NULL;
    }
    if (end == NULL)
        err = PyFile_WriteString("\n", file);
    else
        err = PyFile_WriteObject(end, file, Py_PRINT_RAW);
    ...
    Py_RETURN_NONE;
}
```

一些用 C 语言编写的模块的需要依赖操作系统提供的接口。并且 CPython 源代码需要编译到 macOS、Windows、Linux 和其他 \*nix 操作系统，所以不得不考虑各种特殊情况。

`time` 模块就是一个很好的例子。Windows 使用和存储时间的方式与 Linux 和 macOS 完全不一样，这也是不同操作系统之间 `clock()` 函数精度不同的原因之一。

在 `Modules/timemodule.c` 中，基于 Unix 的操作系统的时间函数从 `<sys/times.h>` 导入：

```c
#ifdef HAVE_SYS_TIMES_H
#include <sys/times.h>
#endif
...
#ifdef MS_WINDOWS
#define WIN32_LEAN_AND_MEAN
#include <windows.h>
#include "pythread.h"
#endif /* MS_WINDOWS */
...
```

同样在这个文件中，可以看到 `time_process_time_ns()` 只是对 `_PyTime_GetProcessTimeWithInfo()` 的包装:

```c
static PyObject *
time_process_time_ns(PyObject *self, PyObject *unused)
{
    _PyTime_t t;
    if (_PyTime_GetProcessTimeWithInfo(&t, NULL) < 0) {
        return NULL;
    }
    return _PyTime_AsNanosecondsObject(t);
}
```

在 `_PyTime_GetProcessTimeWithInfo()` 函数中可以看到，不同操作系统上实现该函数的方式不同，这些不同的实现方式使用宏来控制，编译时会根据操作系统编译所需要的代码到模块中。Windows 系统将调用 `GetProcessTimes()`，而 Unix 系统将调用 `clock_gettime()`。

除了时间模块，对外提供同一个 API 接口但是有多组底层实现的模块还包括线程模块、文件系统模块和网络模块。由于操作系统的行为不同，CPython 源码中通过使用一致的、抽象的接口，尽可能的向上层调用者暴露出相同的行为。
