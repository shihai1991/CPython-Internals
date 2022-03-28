## Python 模块

所有纯 Python 编写的模块都位于源码中的 `Lib` 目录下。一些较大的模块会以文件夹的形式存在，文件下也可能有子模块的文件夹，例如 `email` 模块。
也有一些简单的模块是单个文件，比如 `colorsys` 模块，它只有几百行 Python 代码。大部分人之前可能都没听说过这个模块，`colorsys` 模块提供了在 `RGB` 和其他颜色系统之间的转换函数。

从源代码安装 Python 发行版时，标准库模块会从 `Lib` 文件夹复制到发行版文件夹中。当你启动 Python 时，这个文件夹始终在搜索路径中，因此你可以导入这些标准库模块而不用担心它们的位置。

例如：

```python
>>> import colorsys
>>> colorsys
<module 'colorsys' from '/usr/shared/lib/python3.7/colorsys.py'>
>>> colorsys.rgb_to_hls(255,0,0)
(0.0, 127.5, -1.007905138339921)
```

在 `Lib/colorsys.py` 中我们可以看到 `rgb_to_hls()` 的源码：

```python
# HLS: Hue, Luminance, Saturation
# H: position in the spectrum
# L: color lightness
# S: color saturation

def rgb_to_hls(r, g, b):
    maxc = max(r, g, b)
    minc = min(r, g, b)
    sumc = (maxc+minc)
    rangec = (maxc-minc)
    l = sumc/2.0
    if minc == maxc:
        return 0.0, l, 0.0
    if l <= 0.5:
        s = rangec / sumc
    else:
        s = rangec / (2.0-sumc)
    rc = (maxc-r) / rangec
    gc = (maxc-g) / rangec
    bc = (maxc-b) / rangec
    if r == maxc:
        h = bc-gc
    elif g == maxc:
        h = 2.0+rc-bc
    else:
        h = 4.0+gc-rc
    h = (h/6.0) % 1.0
    return h, l, s
```
这个函数没有什么特别之处，它只是标准的 Python 代码。你会发现所有纯 Python 编写的标准库模块都与它类似。它们只是用纯 Python 编写的，布局美观且易于理解。你甚至可以在这些代码中发现有待改进地方或者bug，然后你可以对其更改并贡献给 Python 社区。在本书的最后会向你介绍如何给 Python 贡献代码。
