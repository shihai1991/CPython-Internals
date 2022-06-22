---
description: Python 内存管理机制概述
---

# Python 内存管理系统设计

由于 CPython 是基于 C 构建的，它也受到 C 中静态内存分配、自动内存分配和动态内存分配的约束。而 Python 语言的一些特性设计使得我们面临更多的挑战：

1. Python 是一门动态类型的语言，变量的大小不能在编译阶段得到；
2. 大多数 Python 的核心类型大小都是动态的，例如 `list` 类型可以是任意长度，`dict` 类型可以有任意数量的键，甚至连 `int` 大小也不是固定的。用户永远不需要去定义这些类型的大小；
3. Python 内的变量名可以复用于任意类型，例如：

```python
>>> a_value = 1
>>> a_value = "Now I'm a string"
>>> a_value = ["Now" , "I'm" "a", "list"]
```

为了解决这些问题，CPython 十分依赖动态内存分配，同时借助垃圾回收（GC）和引用计数算法去保证分配的内存可以自动释放。

Python 对象的内存是通过一个统一 API 自动分配得到的，并不需要 Python 开发者自己去分配内存。这种设计也意味着 CPython 的标准库和核心模块都要使用该 API 去分配内存。

## 内存分配作用域

CPython 可以在 3 个层次的作用域上进行动态内存分配：

1. 原始域：用于从系统的堆上分配内存，用于大块内存或非对象的内存分配；
2. 对象域：用于所有 Python 对象的内存分配；
3. PyMem 域：与 `PYMEM_DOMAIN_OBJ` 功能一致，用于支持旧版本 API。

每一个域都会实现相同函数接口：

* `_Alloc(size_t size)` - 分配指定大小的内存，返回指向这块内存的指针；
* `_Calloc(size_t nelem, size_t elsize)` - 分配 `nelem` 个长度为 `size` 的连续内存空间，并返回指向第一个内存块的指针；
* `_Realloc(void *ptr, size_t new_size)` - 将指针指向的内存重新分配大小；
* `_Free(void *ptr)` - 将指针指向的内存释放回堆中。

枚举变量 `PyMemAllocatorDomain` 将 CPython 中的 3 个作用域分别表示为 `PYMEM_DOMAIN_RAW`, `PYMEM_DOMAIN_OBJ` 和`PYMEM_DOMAIN_MEM` 。

## 内存分配器

CPython 中使用了两种内存分配器：

1. 操作系统层面的内存分配器 `malloc` ，主要用于原始内存作用域;
2. CPython 层面的内存分配器 `pymalloc` ，主要用于 PyMem 和对象内存作用域。

{% hint style="info" %}
**Note**

默认情况下，会将 CPython 的内存分配器`pymalloc`编译到 CPython 的可执行文件中。你可以通过将 pyconfig.h 中的`WITH_PYMALLOC`设置为 0，再重新编译 CPython 去禁用它。禁用之后，在为 PyMem 和对象域分配内存时将使用系统自带的内存分配 APIs。
{% endhint %}

如果你使用 debug 模式去编译 CPython （在 macOS/Linux 下添加 `--with-pydebug` 编译选项，或者在 Windows 下以 `Debug` 模式编译），则所有的内存分配函数都会切换到 Debug 模式下的实现。例如，如果你启用了 Debug 模式，当触发内存分配时调用的将会是 `_PyMem_DebugAlloc()` 而不是`_PyMem_Alloc()` 。
