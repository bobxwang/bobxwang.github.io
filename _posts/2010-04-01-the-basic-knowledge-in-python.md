---
layout: post
title:  "Python基础知识"
date:   2010-04-01 14:17:59
categories: 
- code 
tags:
- python

---
下面罗列一些Python的基础知识。

<pre># -- coding: utf-8 --
people = 30
cars = 40
buses = 15
print 'cars > buses' if (cars > buses) else ('cars < buses' if cars < buses else '''we can't decide''')   ##相当于c中的？：运算符

if 'yes' in ('y','ye','yes'): #判断'yes'是不是在后者
    print 'ok'

'''列表'''
alist = ['a',12,'aheo',{'id':1,'name':'bob'}]
for item in alist:
    print item

'''字典/hash'''
adic = {'name':'bob','age':28,'address':[1,2,3]}

#使用元组最重要的一点是，一旦生成就无法改变
'''元组tuple'''
atuple = (0,'hah',['a','b']) 
print atuple[2]

#无序不重复的元素集,假设有a,b两个set,则a-b表示:差集,a|b表示:并集,a&b表示:交集,a^b表示:(并集跟交集的差集)
'''set''' 
basket = ['apple','orange','apple','pear','apple','banana']
#可以统计apple在这列表中出现的次数
basket.count('apple') 
aset = set(basket)
print aset

#不定长参数，*表示接受一个元组，**表示接受一个字典
def parms(*args,**dic): 
    for arg in args:
        print arg
    for k,v in dic.iteritems():
        print k,':',v

def choose(bool,a,b):
    return (bool and [a] or [b])[0]

def operation():
    u'''操作列表，元组，字典的例子'''
    vec = [2, 4, 6]
    newVec = [2*x for x in vec if x > 3] #列表推导
    lst2 = [4, 3, -9]
    lst3 = [x*y for x in vec for y in lst2]  #vec列表中每个元素跟lst2列表中每一个元素相乘
    #取获（0-10)之间每个数的平方，[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
    lst4 = [x**2 for x in range(10)] 

#filter(function,sequence)返回序列,为原序列中能使function返回true的值
fb = filter(lambda x:x>2,[1,2,3,4]) 
print fb

#map(function,sequence,[sequence...])返回序列,为对原序列每个元素分别调用function获得的值.
#可以传入多个序列,但function也要有相应多的参数,如
mb = map(lambda x,y,z:x+y+z,range(1,3),range(3,5),range(5,7))
print mb
#map函数原型
def map_(function,alist):    
    result = []
    for item in alist:
        result.append(function(item))
    return result

#reduce(function,sequence,[init]),返回一个单值，其计算步骤为：
#a,第1个结果=function(sequence[0],sequence[1])
#b,第2个结果=function(第1个结果,sequence[2])
#...
#返回最后一个计算得值
#如果有init,则先调用function(init,sequence[0])
#sequence只有一个元素时,返回该元素,为空时抛出异常.
#如reduce(lambda x,y:x+y,range(3),99)的计算步骤为：
#99+0=99 => 99+1=100 => 100+2=102，返回值为102

questions=['name','quest','favorite color']
answers=['lancelot','the holy grail','blue']
for q,a in zip(questions,answers):    #zip用于多个sequence的循环
    print 'What is your %s ? It is %s.'%(q,a)
#输出:
#What is your name ? It is lancelot.
#What is your quest ? It is the holy grail.
#What is your favorite color ? It is blue.

#When testing for nulls, always use if foo is None rather than if !foo so foo == 0 does not cause bugs.

#reversed反向循环
for i in reversed(range(1,4)):   
    print i  #输出：3，2，1

# f=open('/tmp/hello','w') 'w'读写模式有，r(只读),r+(读写),w(新建，会覆盖原有文件),a(追加),b(二进制文件)，模式可以组合使用

class Person(object):
    def __int__(self,name):
        self.name = name
        self.pet = None

class Employee(Person):
    def __int__(self,name,salary):
        super(Employee,self).__init__(name)
        self.salary = salary

def print_two(*args):
    arg1,arg2 = args
    print  "arg1:  %r,  arg2:  %r"  %  (arg1,  arg2)

class ParserError(Exception):
    pass

def convert_number(s):
    try:
        raise ParserError("Expected a verb next.")
        return int(s)
    except ParserError:
        return None

if __name__ == '__main__':
    pass

</pre>

重点关注Python中的列表解析，函数式编程，可变参数






