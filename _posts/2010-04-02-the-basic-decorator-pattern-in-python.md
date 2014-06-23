---
layout: post
title:  "Python中的装饰器模式"
date:   2010-04-02 14:17:59
categories: 
- code 
tags:
- python

---
装饰器使得函数和方法的封装(接收一个函数并返回增加版本的一个函数)更加容易和理解。
下面就是一个最简单的装饰器
<pre>def mydecorator(function):
    def _mydecorator(*args,**kw):
        #实际调用前处理
        res = function(*args,**kw) 
        #实际调用后处理
        return res
    return _mydecorator
</pre>
为子函数应用一个诸如_mydecorator之类明确的名称比像wrapper这样的通用名称是一个良好的习惯。

当装饰器需要参数时，必须使用第二级封装，这是因为装饰器在模块第一次被读取时由解释程序装入，所以它们使用受限于总体上可应用的封装器。
<pre>def mydecorator(arg1,arg2):
    def _mydecorator(function):
        def __mydecorator(*args,**kw):
            res = function(*args,**kw)
            return res
        return __mydecorator
    return _mydecorator
</pre>

常见的装饰器模式包括：

+   参数检查
+   缓存
+   代理
+   上下文提供者

其实Python类中所谓的属性，类方法，静态方法都是由装饰器来实现的。

装饰器在其它平台，如NET，JAVA中可以理解成AOP--拦截器。