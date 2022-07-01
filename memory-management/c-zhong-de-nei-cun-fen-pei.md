---
description: 简单了解 C 语言中内存分配方式。
---

# C 中的内存分配

在 C 语言中，在使用变量前首先需要从操作系统为它们分配内存。

在 C 中有三种内存分配的机制：

1. 静态内存分配，这部分内存在编译阶段就要计算出大小，在可执行文件运行时真正被分配；
2. 自动内存分配，当一个帧开始时，会在调用栈中分配此作用域（例如函数）需要的内存，在帧执行完成后这部分内存被释放；
3. 动态内存分配，可以在运行时通过调用内存分配的 API 动态的分配内存。

## 静态内存分配

C 中的类型所占的内存大小是固定的。对于全局和静态变量，编译器可以计算出它们需要的内存大小并将这些大小信息编译到应用程序中。

举个例子：

```c
static int number = 0;
```

你可以通过 `sizeof()` 函数去查看 C 中类型所占的内存大小。就我的操作系统来说，在 64 位 macOS 上通过 GCC 编译器得到的 `int` 类型大小是 4 bytes。根据操作系统架构和编译器的类型，C 中的基础类型可能有不同的大小。

对于静态定义的数组类型，这里以一个包含了 10 个整型的数组作为例子：

```c
static int numbers[10] = {0,1,2,3,4,5,6,7,8,9};
```

C 编译器会为这段代码分配 `sizeof(int) * 10 bytes` 大小的内存空间。

C 编译器使用系统调用去分配内存。这些系统调用依赖于操作系统架构，是内核中更底层的函数，用于从系统内存页分配内存。

## 自动内存分配

类似于静态内存分配，自动内存分配将在编译期间计算需要分配的内存大小。下面给出了一个计算 100 华氏度的例子：

```c
#include <stdio.h>
static const double five_ninths = 5.0/9.0;

double celsius(double fahrenheit) {
    double c = (fahrenheit - 32) * five_ninths;
    return c;
}

int main() {
    double f = 100;
    printf("%f F is %f Cn", f, celsius(f));
    return 0;
}
```

这个例子使用了静态和动态内存分配技术：

* `five_ninths` 这个变量由于带有 `static` 关键字所以使用了静态内存分配；
* 当函数 `celsius()` 调用时，函数中的变量 `c` 使用了自动内存分配，为其分配的内存在函数结束后释放；
* 当函数 `main()` 调用时，函数中的变量 `f` 使用了自动内存分配，为其分配的内存在函数结束后释放；
* `celsius(f)` 输出的结果隐式的自动分配了内存；
* 所有自动分配的内存在 `main()` 函数结束后被释放。

## 动态内存分配

许多情况下，静态内存分配和自动分配内存两者都不能满足我们的需求。举个例子，编译阶段有时无法知道我们究竟需要多大的内存，因为这些数据可能是用户输入的。

这时候就需要使用动态内存分配。动态内存分配通过调用 C 语言的内存分配 API 来工作。

操作系统保留了一段内存专门用于动态内存分配，这部分内存也称为堆。

下面给出一个例子，你将使用动态内存分配去为记录了华氏度和摄氏度的数组分配内存。

```c
#include <stdio.h>
#include <stdlib.h>

static const double five_ninths = 5.0/9.0;

double celsius(double fahrenheit) {
    double c = (fahrenheit - 32) * five_ninths;
    return c;
}

int main(int argc, char** argv) {
    if (argc != 2)
        return -1;
    int number = atoi(argv[1]);
    double* c_values = (double*)calloc(number, sizeof(double));
    double* f_values = (double*)calloc(number, sizeof(double));
    for (int i = 0 ; i < number ; i++ ){
        f_values[i] = (i + 10) * 10.0 ;
        c_values[i] = celsius((double)f_values[i]);
    }
    for (int i = 0 ; i < number ; i++ ){
        printf("%f F is %f Cn", f_values[i], c_values[i]);
    }
    free(c_values);
    free(f_values);
    return 0;
}
```

传入参数 4 去执行这段代码，你将会得到以下结果：

```c
100.000000 F is 37.777778 C
110.000000 F is 43.333334 C
120.000000 F is 48.888888 C
130.000000 F is 54.444444 C
```

这个例子就是使用了堆中的内存块去进行动态内存分配，而在释放内存后，这部分内存的控制权又交回给堆。如果存在动态分配的内存没有被释放，就会造成内存泄漏。
