---
categories:
  - code
date: 2014-04-01 14:23:09
tags:
  - scala
  - implicit
  - extensions method
title: scala中的隐式转换
---

#### implicit 
在scala中有一个关键字，implicit，就是我们这篇文章的主题。
先看一个简单的例子：
```scala
def display(input:String) = println(input)
```
我们可以看到display函数的定义只接受String类型的参数，如果调用display("fucktest")不会有任何问题，但如果传参是非字符串的话，编译会通不过。
此时如果我们想让display可以接受其它类型的参数，除非我们重载它，重新定义一个其它类型有入参，此时implicit关键字就派上用场了，我们可以在相同的作用域内用implicit关键字定义一个隐式转换函数。
```scala
object ImplicitTutorial{
	def display(input:String) = println(input)
	implicit def int2String(input:Int) = input.toString
	implicit def boolean2String(input:Boolean) = if(input) "true" else "false"
	def main(args:Array[String]):Unit = {
		def("fucktest") 
		def(2)
		def(true)
	}
}
```
在这个作用域内，我们定义了两个隐式转换函数，这样dispay函数就能接受Int跟Boolean类型的参数了。
得出一个结论：
> 隐式函数是指在同一个作用域下，一个给定参数类型并自动转换指定类型的函数，这个函数跟函数名字无关，跟入参参数名字无关，只和入参参数类型及返回类型相关，注意是同一个作用域。

关于这点，在c#语言中一样存在implicit这个关键字
```scala
class Machine {
  	public int Value { get;set; }
	public static implicit operator ToWidget(Machine m) {
		Widget w = new Widegt();
		w.Value = m.Value * 2;
		return w;
	}
} 
class Widget {
	public int Value { get;set; }
	public static implicit operator ToMachine(Widget w) {
		Machine m = new Machine();
		m.Value = w.Value / 2;
		return m;
	}
}
class Program {
	static void Main() {
		Machine m = new Machine();
		m.Value = 5;
		// Implicit conversion from machine to widget.
		Widget w = m;
		Console.WriteLine(w.Value); // should print 10
		Machine m2 = w
		Console.WriteLine(w.Value); // should print 5
	}
}
```
可以看到两个不同的语言用差不多的方法解决相似的类型转换问题。

#### Type Extension
不过，在c#中，有一个扩展方法(objective-c中也有相关概念，叫做[Category](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/CustomizingExistingClasses/CustomizingExistingClasses.html))，它让我们可以在不用继承的前提下对一个类进行扩展，在这概念出现前，我们一般会写一个工具类来处理，让我们看看是怎么实现的。
```c#
public static class IntExtension{
	public static long plus(this int input,int another) {
		return input + another;
	}
}
```
这样我们就为int这种类型添加了一个plus的方法，因此我们可以这样来调用
```c#
int i = 5;
long result = i.plus(56)
```
如果想在scala中实现相同的方案，我们可以变相的通过implicit来实现，下面是相应的代码实现。
```scala
case class IntExtension(value:Int) {
	def plus(another:Int):Int = value + antoher
}
implicit def int2IntExtension(value:Int) = { IntExtension(value) }
val i = 45
val resutl = i.plus(3)
```
这样我们就实现了c#中的扩展方法。
> 最后总结：
> > 隐式转换在同一个作用域中不能存在参数跟返回值完全相同的两个函数
> > 
> > 隐式转换只在意输入类型，返回类型
> > 
> > 隐式转换是scala语法灵活和简洁的重要组成部分