# 翻译规范

## 翻译要求
* 在使用直译方式尊重原作者表达意图的基础上再结合意译方式消除中西语言导致的思维逻辑偏差。
* 具体函数翻译为中文需要强调`函数`时，将函数跟在实际函数后面，如:`in print()`，可以翻译成在 `print()` 函数中。



## 表格规范

文中出现的三线表统一使用 Markdown 内置的表格，例如：

| 文件 | 功能    |
| ---- | ------- |
| a.py | demo a. |
| b.py | demo b. |



## 符号规范

### 基础规范

* 文章中的符号规范遵从 [中文文案排版指北](https://github.com/sparanoid/chinese-copywriting-guidelines) ；
* 文章中的粗体或斜体字需要使用添加``进行突出说明。


### 有序列表和无序列表

有序列表和无序列表未结束时以 `; ` 结尾，最后一项以 `。`结尾。

例如：

> - Start；
> - Do something；
> - End；



### 换行

小章节内容结束后空一行，再添加新的标题及内容，例如：

> ## 1. Chapter 1
>
> Do something.
>
> 
>
> ## 2. Chapter 2
>
> Do something.
>
> 



### 图题

* 图片基于 `gitbook` 平台上传，默认存放在 `.gitbook/assets` 下;
* 建议图片文件的命名格式为 `图$(章节名).$(图片序号) 图片说明`，例如：`图9.1 The Evaluation Loop` ;
* 文章中的图题与文件名保存一致。



### Note

使用 `gitbook` 自带的富文本功能，语法格式如下：

```
{% hint style="info" %} Note

Content.

{% endhint %}
```



# 术语规范

## 不需要翻译的术语
`CPython`内有严格定义的数据结构尊重原始定义不做二次翻译，如: PyObject、TypeType、PyFrameObject、Threading等。



## 需要翻译的术语
暂无
