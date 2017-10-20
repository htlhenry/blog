---
title: Python元类FAQ
date: 2017-10-20 17:44:18
tags: 
- 元类
categories:
- Python
---



> 此处的问题以及解答摘自《Python学习手册 第4版》

#### 1.什么是元类？

元类是用来创建一个类的类。常规的类默认的是type类的实例。元类通常是type类的子类，它重新定义了类创建协议方法，以便定制在一条class语句的末尾发布的类创建调用(创建类的\_\_call\_\_()函数)；它通常会重新定义\_\_new\_\_和\_\_init\_\_方法以接入创建类的协议，元类也可以以其他的方式编码—例如，作为简单函数，但是他们负责为新类创建和返回一个对象。

一句话版本：

> 元类是创建类的类，元类可以为它所有的实例类统一添加方法或执行代码.

#### 2.如何声明一个类的元类？

在Python3.0 及以后的版本中，使用一个关键字参数: class(metaclass=M)。在Python2.x中，使用类属性: \_\_metaclass\_\_ =  M 。

#### 3.在管理类方面，类装饰器如何与元类重叠？

由于二者都是在一条class语句的末尾自动触发的，因此类装饰器和元类都可以用来管理类。装饰器**把一个类名重新绑定到一个可调用对象的结果**，而元类**把类创建指向一个可调用对象**，但它们都是可以用做相同目的的钩子。要管理类，装饰器直接扩展并返回最初的类对象。元类在创建一个类之后扩展它。

类装饰器:

```
def tracer(cls):
	class Wrapper(object):
		def __init__(self, *args):
			self.wrapped = cls(*args)
		def __getattr__(self, name):
			print("Getting the {} of {}".format(name,  self.wrapped))
			return getattr(self.wrapped, name)
    return Wrapper

@tracer
class Spam(object):
	pass
```

元类:

```
class Meta(type):
	def __new__(meta, classname, supers, classdict):
	    do_something()
	    classdict[some_attrs] = attr
		return type.__new__(classname, supers, classdict)

class Spam(object):
    __metaclass__ = Meta
    pass
```



#### 4.在管理实例方面，类装饰器如何与元类重叠？

由于二者都是在一条class语句的末尾自动触发的，因此类装饰器和元类都可以用来管理类实例，通过插入一个包装器对象来捕获实例创建调用。装饰器把类名重新绑定到一个可调用对象，而该可调用对象在实例创建时运行以保持最初的类对象。元类可以做同样的事情，但是，它们必须也创建类对象，因此，它们用在这一角色中更复杂一些。