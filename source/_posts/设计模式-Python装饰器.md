---
title: 'Python-装饰器模式'
date: 2021-01-28 17:35:16
tags: [设计模式,装饰器,Python]
categories: Python
---



装饰器本质上是一个Python函数，它可以让其他函数在不需要做任何代码变动的前提下增加额外功能，装饰器的返回值也是一个函数对象。它经常用于有切面需求的场景，比如：插入日志、性能测试、事务处理、缓存、权限校验等场景。装饰器是解决这类问题的绝佳设计，有了装饰器，我们就可以抽离出大量与函数功能本身无关的雷同代码并继续重用。

<!--more-->

最简单的装饰器：
```
def debug(func):
    def wrapper():
        print "[DEBUG]: enter {}()".format(func.__name__)
        return func()
    return wrapper

@debug
def say_hello():
    print "hello!"
```

当函数需要接收参数时，装饰器这样写：
```
def debug(func):
    def wrapper(*args, **kwargs):  # 指定宇宙无敌参数
        print "[DEBUG]: enter {}()".format(func.__name__)
        print 'Prepare and say...',
        return func(*args, **kwargs)
    return wrapper  # 返回

@debug
def say(something):
    print "hello {}!".format(something)
```
如果想让装饰器可以定制化，那么装饰器也需要接收参数，比如log装饰器，我们想要传参数来说明日志的等级这个时候需要多一层：
```
def logging(level):
    def wrapper(func):
        def inner_wrapper(*args, **kwargs):
            print "[{level}]: enter function {func}()".format(
                level=level,
                func=func.__name__)
            return func(*args, **kwargs)
        return inner_wrapper
    return wrapper

@logging(level='INFO')
def say(something):
    print "say {}!".format(something)

# 如果没有使用@语法，等同于
# say = logging(level='INFO')(say)

@logging(level='DEBUG')
def do(something):
    print "do {}...".format(something)

if __name__ == '__main__':
    say('hello')
    do("my work")

```
python的变量寻找是依次向上找命名空间的，这个特性也支持了我们装饰器可以嵌套的来写。
