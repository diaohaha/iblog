---
title: python-with as语句
date: 2021-01-28 17:38:13
tags: [Python]
categories: Python
---

#### With语句是什么？
有一些任务，可能事先需要设置，事后做清理工作。对于这种场景，Python的with语句提供了一种非常方便的处理方式。一个很好的例子是文件处理，你需要获取一个文件句柄，从文件中读取数据，然后关闭文件句柄。但是切莫只局限于使用with去处理文件操作，其他类似的场景也可以使用。

<!--more-->

```
with open("/tmp/foo.txt") as file:
    data = file.read()
```

#### with如何工作？

Python对with的处理还很聪明。基本思想是with所求值的对象必须有一个__enter__()方法，一个__exit__()方法。紧跟with后面的语句被求值后，返回对象的__enter__()方法被调用，这个方法的返回值将被赋值给as后面的变量。当with后面的代码块全部被执行完之后，将调用前面返回对象的__exit__()方法。

```
class ThriftClient(object):
    """ client """
    def __init__(self, module, host, port, timeout=2000):
        tsocket = TSocket.TSocket(host, port)
        tsocket.setTimeout(timeout)
        self.transport = TTransport.TBufferedTransport(tsocket)
        self.client = module.Client(TBinaryProtocol.TBinaryProtocol(self.transport))

    def __enter__(self):
        self.transport.open()
        return self.client

    def __exit__(self, value_type, type, trace):
        self.transport.close()
```
实际上，在with后面的代码块抛出任何异常时，__exit__()方法被执行。正如例子所示，异常抛出时，与之关联的type，value和stack trace传给__exit__()方法，因此抛出的ZeroDivisionError异常被打印出来了。开发库时，清理资源，关闭文件等等操作，都可以放在__exit__方法当中。
因此，Python的with语句是提供一个有效的机制，让代码更简练，同时在异常产生时，清理工作更简单。
