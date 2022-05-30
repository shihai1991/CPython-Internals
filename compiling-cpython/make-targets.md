# 4.5 CPython 的 make 目标

如果你使用的是 Linux 或 macOS，你会发现需要自己清理文件、构建或更新配置。

CPython 的 Makefile 中内置了许多有用的 make 目标：

### 4.5.1 构建目标

| 目标          | 目的                                 |
| ------------- | ----------------------------------- |
| all (default) | 构建编译器、库和模块                   |
| profile-opt   | 使用 PGO 优化编译 Python 二进制文件    |
| clinic        | 在所有源文件上运行“Argument Clinic”   |
| sharedmods    | 构建共享模块                         |
| regen-all     | 重新生成所有生成的文件                 |

### 4.5.2 测试目标

| 目标           | 目的                                                |
| ------------- | --------------------------------------------------- |
| test          | 运行一组基本的回归测试                                  |
| testall       | 运行完整的测试套件两次 —— 一次不使用 .pyc 文件，而另一次使用 |
| quicktest     | 运行一组更快的回归测试，其不包括需要很长时间的测试用例        |
| testuniversal | 在 OSX 上的通用构建中运行两种架构的测试套件                |
| coverage      | 使用 gcov 编译和运行测试                                |
| coverage-lcov | 创建覆盖率 HTML 报告                                   |

### 4.5.3 清理目标

主要的清理目标包括 `clean`、`clobber` 和 `distclean`。 `clean` 目标通常用于删除已编译和缓存的库和 pyc 文件。若你发现 `clean` 未起作用，请尝试使用 `clobber`。若要在发行前彻底清理环境，请运行 `distclean` 目标。

| 目标              | 目的                                              |
| --------------- | -------------------------------------------------- |
| check-clean-src | 从源码构建时检查源码是否干净                            |
| cleantest       | 删除之前失败的测试任务的“test\_python\_\*”目录          |
| clean           | 删除 pyc 文件、编译的库和配置文件                       |
| pycremoval      | 删除 pyc 文件                                       |
| docclean        | 删除 Doc/ 中的构建文档                                |
| profile-removal | 删除所有优化配置文件                                   |
| clobber         | 与 `clean` 相同，但同时会删除库、标签、配置和 build 目录    |
| distclean       | 与 `clobber` 相同，但同时会删除从源生成的任何内容，例如 Makefile |

### 4.5.4 安装目标

安装目标分为两类，一类用于安装默认版本，例如 `install`，另一种用于安装 `alt` 版本，例如 `altinstall`。如果你想在自己的计算机上安装编译后的版本，但不希望它作为默认的 Python 3，请使用命令的 `alt` 版本。

使用 `make install` 安装后，命令 `python3` 现在将链接到你编译的二进制文件。

使用 `make altinstall` 只会安装 `python$(VERSION)`，而 `python3` 的现有链接将保持不变。

| 目标            | 目的                                                             |
| ------------- | -------------------------------------------------------------- |
| install       | 安装共享库、二进制文件和文档。将运行 `commoninstall`、`bininstall` 和 `maninstall` |
| bininstall    | 安装所有二进制文件，例如 `python`，`idle`，`2to3`                            |
| altinstall    | 安装带有版本后缀的共享库、二进制文件和文档                                          |
| maninstall    | 安装手册                                                           |
| altmaninstall | 安装带有版本后缀的手册                                                    |
| altbininstall | 安装带有版本后缀 `python` 解释器，例如`python3.9`                            |
| commoninstall | 安装共享库和模块                                                       |
| libinstall    | 安装共享库                                                          |
| sharedinstall | 安装动态加载的模块                                                      |

### 4.5.5 其他目标

| 目标            | 目的                                           |
| ------------- | ----------------------------------------------- |
| python-config | 生成 `python-config` 脚本                        |
| recheck       | 使用与上次运行时相同的选项重新运行 `configure`        |
| autoconf      | 重新生成 `configure` 和 `pyconfig.h.in`           |
| tags          | 为 `vi` 创建一个标签文件                           |
| TAGS          | 为 `emacs` 创建标签文件                            |
| smelly        | 检查导出的符号是否以 `Py` 或 `_Py` 开头（请参阅 PEP7） |
