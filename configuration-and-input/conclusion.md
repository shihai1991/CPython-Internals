# 总结

在这个章节中，你已经了解了 Python 的诸多配置选项是如何加载的，以及代码是如何输入到解释器中的。Python 的输入灵活性使其成为支撑一系列应用程序的出色工具，例如：

- 命令行实用程序
- 长时间运行的网络应用程序，例如Web服务器
- 简短、可组合的脚本

Python 设置配置属性的方式是多样的，从而也导致了 Python 语言的复杂性。例如，如果您基于 Python3.8 测试一个 Python 应用程序，它执行正确，但是在一个不同的环境下，它又执行失败了，您必须要清楚在这个环境中有哪些设置跟 Python3.8 不同。这意味着您需要检查环境变量、运行时标志，甚至是系统配置属性。在系统配置中找到的编译时属性在各个 Python 的发行版中可能不同。例如，从 python.org 下载的适用于 macOS 的 Python3.8 的默认值与 Homebrew 上的 Python3.8 发行版或 Anaconda 发行版上的默认值不同。

所有这些输入方法都会输出一个 Python 模块。在下一章，您将了解如何根据输入的内容创建模块。