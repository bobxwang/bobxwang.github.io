---
categories:
  - code
date: 2010-04-03 16:17:59
tags:
  - python
title: 如何在web.py处理请求前后添加自己的逻辑
---

web.py在python百花齐放的web开发框架中，可谓是一个简单却功能强大的轻量级框架，麻雀虽小，五脏俱全。

一般在web应用中常见的可能就是判断用户是否已登陆，然后做不同的权限处理。在Asp.Net平台里我们用HttpModuel来完成类似功能，在Django中我们利用Middleware来完成，那么在web.py中呢，我们如何来处理呢。

web.py中完成类似功能的东西有两种，一种叫做应用处理器--Application processors，另一种叫做钩子。

**应用处理器**

在具体处理请求前后，添加相应的逻辑
```python
def my_processor(handler):
    print 'before handling'
    result = handler() 
    print 'after handling'
    return result

app.add_processor(my_processor)
```

**加载钩子(loadhook)或卸载钩子(unloadhook)**

除了处理器外，钩子也可以完成上述功能
```python
def my_loadhook():
    print "my load hook"

defmy_unloadhook():
    print "my unload hook"

app.add_processor(web.loadhook(my_loadhook))
app.add_processor(web.unloadhook(my_unloadhook))
```
可以看到，调用的都是add_processor这个方法，在钩子处，处理前后给独立了出来，推荐采用。