# 4.4 make 快速入门

作为一名 Python 开发者，你之前可能没接触过 `make`，或者接触过但没花太多时间在它上面。对于 C、C++ 和其他的编译型语言，以正确的顺序加载、链接和编译代码所需执行的命令数量是非常耗人精力的。从源代码编译应用程序时，需要链接任何必须的外部库。期望开发人员知道所有这些库的位置并将它们复制粘贴到命令行中是不现实的，因此在 C/C++ 项目中通常使用 `make` 和 `configure` 来自动创建构建脚本。当执行 `./configure` 时，`autoconf` 会在系统中搜索 CPython 所需的库，并将其路径复制到 `Makefile` 中。

生成的 `Makefile` 类似于 shell 脚本，可以将它分为若干个被称为 **"targets"** 的部分。

以 `docclean` 目标为例。该目标使用 `rm` 命令删除一些生成的文档文件。

```makefile
docclean:
    -rm -rf Doc/build
    -rm -rf Doc/tools/sphinx Doc/tools/pygments Doc/tools/docutils
```

要执行此目标，请运行命令 `make docclean`。`docclean` 是一个简单的目标，因为它只运行两条命令。

执行任何 make 目标的语法规范都是：

```bash
$ make [options] [target]
```

如果你调用 `make` 时没有指定目标，`make` 将运行默认目标，即 `Makefile` 中设定的第一个目标。对于 CPython 而言，默认是一个名为 `all` 的目标，它用于编译 CPython 所有的部分。

Make 有很多选项，下面列出了对你阅读本书有帮助的相关选项：

| 选项                                                    | 功能                                |
| ------------------------------------------------------- | ----------------------------------- |
| -d, --debug\[=FLAGS]                                    | 打印各种类型的调试信息              |
| -e, --environment-overrides                             | 环境变量覆盖 makefile 中的同名变量  |
| -i, --ignore-errors                                     | 忽略执行命令时的错误                |
| -j \[N], --jobs\[=N]                                    | 同时执行 N 任务（否则为无限个任务） |
| -k, --keep-going                                        | 当一些目标无法实现时继续往下执行    |
| <p>-l [N], --load-average[=N],</p><p>--max-load[=N]</p> | 仅在负载 < N 时启动多个任务         |
| -n, --dry-run                                           | 只打印命令而不执行命令              |
| -s, --silent                                            | 不回显命令                          |
| -S, --stop                                              | 关闭选项 -k                         |

在下一节及整本书中，我们将使用以下选项运行 `make`：

```bash
$ make -j2 -s [target]
```

标志 `-j2` 允许 `make` 同时运行 `2` 个任务。如果你有 `4` 个或更多的核，那么可以将 `2` 改为 `4` 或更大，这将使编译更快完成。 标志 `-s` 阻止 `Makefile` 将它运行的每个命令打印到控制台。如果你想查看发生了什么，请删除 `-s` 标志。
