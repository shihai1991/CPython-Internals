---
description: CPython 如何解析并执行 Code Object。
---

# 九、求值循环

到本章为止，你可能已经了解了如何将 Python 代码解析为抽象语法树并将其编译成 code objects，这些 code objects 包含了以字节码形式表示的一系列操作。但想要真正的执行 code objects 还缺少一样关键的东西！那就是输入。在 Python 中，输入可能以局部变量或全局变量的形式出现。在本章中，你将会接触到一个名叫 `Value Stack` 的概念，code object 中的字节码将创建、修改和使用 `Value Stack` 中的变量。

CPython 中执行代码的动作发生在一个核心循环中，这个循环又被称为 "evaluation loop (求值循环)"。CPython 解释器将在这解析并执行由序列化的 `.pyc` 文件或由编译器得到的 code object：

![图9.1 The Evaluation Loop](<../.gitbook/assets/图9.1 The Evaluation Loop.png>)

在 evaluation loop 中，每一个字节码指令都是基于系统的 “[栈帧](http://www.cs.uwm.edu/classes/cs315/Bacon/Lecture/HTML/ch10s07.html)” 获取和执行的。

{% hint style="info" %}
**Note**

不止在 Python 中，在许多 runtimes 中都使用了 **栈帧** 这种数据结构。栈帧保证了可以在函数中调用函数并获取返回值，它同样包含了参数、局部变量和其他一些状态信息。

每次调用函数时都会创建一个栈帧，这些栈帧按调用顺序堆叠在一起。如果抛出了未处理的异常，你就可以看到 CPython 的栈帧信息：

```python
Traceback (most recent call last):
    File "example_stack.py", line 8, in <module> <--- Frame
        function1()
    File "example_stack.py", line 5, in function1 <--- Frame
        function2()
    File "example_stack.py", line 2, in function2 <--- Frame
        raise RuntimeError
    RuntimeError
```
{% endhint %}

当探索 CPython 编译器时，你已经理解了 [run\_eval\_code\_obj()](https://github.com/python/cpython/blob/v3.9.0b1/Python/pythonrun.c#L1155) 调用之前的内容。在本章，你将会进一步探索解释器相关的 API：

![图9.2 run\_eval\_code](<../.gitbook/assets/图9.2 run\_eval\_code.png>)

### 相关的源文件

| 文件                 | 功能                       |
| ------------------ | ------------------------ |
| Python/ceval.c     | 实现 evaluation loop 的核心代码 |
| Python/ceval-gil.h | GIL的定义和控制算法              |

### 需要关注的内容

* evaluation loop 将会获取一个 **code object** 并将其转换为一系列的 **frame object**；
* 解释器至少需要一个**线程**；
* 每个线程都有自己的**线程状态**；
* 在栈中执行 **frame objects**，又被称为栈帧；
* 变量的引用存放在 **value stack** 中。
