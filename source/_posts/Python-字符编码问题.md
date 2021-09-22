---
title: Python-字符编码问题
date: 2021-01-28 17:37:45
tags: [字符编码,Python]
categories: Python
---


字符编码问题很简单，当你可以区分以下几种概念之后：
1. 字节编码与字符编码
2. 字节串与字符串
3. 文件编码、IDE编码、其他I/O编码
4. __str__()与__repr__()

<!--more-->

---
#### 字节编码与字符编码
ascii unicode是字符编码，编码后的字符编成一个段落是字节编码，utf-8 gbk是字节编码。
#### 字节串与字符串
+ str是字节串，由unicode经过编码(encode)后的字节组成的。使用len来求长度获取的是字节长度。
+ unicode才是严格意义上的字符串，如u'中文'。使用len来求长度时是字符个数。
```
>>> len("中文")
6
>>> len(u"中文")
2
```
在程序中处理字符串之前，最好先确定是哪种类型。
```shell
>>> isinstance(u'中文', unicode)
True
>>> isinstance(u'中文', str)
False
>>> isinstance('中文', str)
True
```
通常的处理过程是识别输入编码，然后解码成unicode，在最后输出之前在进行编码，分别使用python内建函数encode和decode，如果确定一个字符串不是unicode类型，这时候需要进行解码，可以使用chardet识别str的编码。
```
>>> from chardet import detect
>>> a = "中文"
>>> detect(a)
{'confidence': 0.682639754276994, 'encoding': 'KOI8-R'}
```
#### 文件编码、IDE编码、其他I/O编码

+ Python程序文件的编码 (默认编码ascii 只在文件中有编码时有影响)
+ Python程序运行时环境（IDE）的编码（影响输入和输出）
+ Python程序读取外部文件、网页的编码 （影响输入与输出）

当我们发现乱码时，就要区分是第二还是第三种情形，终端打印是IDE相关，数据库属于其他I/O相关，之所以出现乱码就是编码不匹配，我们需要确定的是不匹配的双方和双方所对应的编码。

对于系统的编码可以使用sys来获取，这影响终端显示。
```
>>> import sys
>>> sys.getdefaultencoding()
'ascii'
```
#### __str__()与__repr__()

__repr__和__str__这两个方法都是用于显示的，__str__是面向用户的，而__repr__面向程序员。print首先尝试的是__str__函数，这就是我们unicode字符串打印出来和字节串一样的原因。
