# 为什么 CPython 是用 C 语言而不是用Python语言来实现

CPython 中的 `C` 来源于 C 编程语言，这意味着 Python 发行版本是用 C 语言写的。

这个结论基本是正确的：CPython 中的编译器是用纯 C 语言写的。但是许多的标准库模块是用纯 Python 语言或者是 C 及 Python 语言混写而成。

#### 所以为什么 CPython 编译器是用 C 语言而不是 Python语言来实现？

这个问题的答案在于编译器是如何工作的。主要有两种类型的编译器：

1. 自编译编译器：是用他们要编译的语言所写成的编译器，比如：Go 编译器。这个过程是由一个称为“自举”的过程所实现；
2. 源到源编译器：是用另一种已经有编译器的语言实现的编译器。

如果你想从零开始写一门新的编程语言，你需要一个可执行应用来编译你的编译器！你需要一个编译器能执行任何事情，所以当一门新语言被开发出来时，他们通常需要用一种更老、更稳定的语言来编写。

还有一些工具可以使用语言规范并且创建解析器（这个主题将会在此章节中详细讲到）。主流的编译器生成器有：GNU Bison，Yacc 和 ANTLR。

{% hint style="info" %}
如果你想了解更多关于解析器的信息，可以下载查看 [lark](https://github.com/lark-parser/lark) 项目。lark 是一个用 Python 编写的上下文无关文法的解析器。
{% endhint %}

一个编译器自举的优秀案例是 Go 编程语言。第一个 Go 编译器是由 C 语言写的，然后一旦 Go 可以被编译，Go 编译器就会被 Go 语言重写。

CPython 保留了 C 语言的实现；许多标准库模块，如：ssl 模块或者 sockets 模块，都是用 C 语言编写的以访问底层操作系统 API。Windows 和 Linux 内核中用来创建网络套接字，使用文件系统或者和显示器交互的API都是用 C 语言编写的。

Python 的扩展层聚集在 C 语言是合理的。在本书的下半部分，你可以了解到Python标准库和 C 模块。

还有一个用 Python 语言实现的 Python 编译器叫做：[PyPy](https://pypy.org)。PyPy 的logo是一条[携尾蛇](https://en.wikipedia.org/wiki/Ouroboros)，用这个logo来表达本身的自编译特性。

Python 交叉编译器的另一个例子是 [Jython](https://www.jython.org)。Jython 是用 Java 语言编写的并且可以将 Python 源码编译为 Java 字节码。CPython 使其导入 C 库以及在Python语言使用它们更加简单，而 Jython 则是让它能更简单的引用 Java 模块以及相关类。

创建编译器的第一步是要定义语言。如下示例所示不是一个有效的 Python 代码：

```python
def my_example() <str>:
{
    void* result = ;
}
```

编译器在尝试执行代码之前需要对目标语言有严格的语法结构规则。

{% hint style="info" %}
对于本章节的剩余部分，./python 回引用到 CPython 的编译版本。但实际的命令依赖于你的操作系统。

对于 Windows：

\\> python.exe

对于 Linux：

$ ./python

对于 MacOS：

$ ./python.exe
{% endhint %}
