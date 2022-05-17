# 构建配置

就像 Include/cpython/initconfig.h 中的运行时配置一样，还有一个配置叫做构建配置。构建配置位于根目录的 pyconfig.h 文件中，这个文件是在执行 `./configure` （macOS/Linux）或者 `build.bat` （Windows）过程中动态创建的。

通过执行以下命令即可看到构建配置： 

```bash
$./python -m sysconfig

Platform: "macosx-10.15-x86_64"
Python version: "3.9"
Current installation scheme: "posix_prefix"

Paths:
data = "/usr/local"
include = "/Users/anthonyshaw/CLionProjects/cpython/Include"
platinclude = "/Users/anthonyshaw/CLionProjects/cpython"
...
```
构建配置属性属于编译时值，用于选择要链接到二进制文件中的附加模块。例如，调试器、检测库和内存分配器都是在编译时进行设置的。

通过以上三个配置阶段（预初始化配置、运行时配置、构建配置），现在 CPython 解释器就可以开始接受输入并将文本转换为可执行代码了。