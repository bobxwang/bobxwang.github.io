---
layout: post
title:  "利用scala的宏达到map跟case class的互换"
date:   2017-04-09 20:17:59
categories: 
- code 
tags:
- scala
- macros
- map
- case class
---
前段时间接触了scala的宏，相对C语言的**＃define**可以将其理解成一个更先进的版本，没多少实战经验的可以将其理解成一个复杂的查找替换工具，在每次编译前运行。从另一方面来讲，scala的宏可以给我们带来代码生成，字符串静态类型安全检测甚至允许创建流畅的DSL接口。

#### 问题描述

对于我来讲，最有用的地方就是代码生成了。假设一种场景，将一个case class转成一个map，map的keys就是case class的参数，或者将一个map转成一个case class，map的keys就是case class的参数。这里还要同时感谢**implicit macros**。我们定义如下的一个特质来进行类型安全转换。

<pre>trait Mappable[T] {
  def toMap(t:T) : Map[String, Any]
  def fromMap(map:Map[String,Any]) : T
}
</pre>

这样任何一个实现了`Mappable[T]`的都可以方便地在T跟Map间进行转换，举个例子如下：

<pre>case class People (name:String, age:Int) 
val PeopleMapper = new Mappable[People] {
  def toMap(p:People) = Map ( "name" -> p.name, "age" -> p.age)
  def fromMap(map:Map[String,Any]) = People(
  	map("name").asInstanceOf[String],
  	map("age").asInstanceOf[Int])
}</pre>

这个时候当case class发生了变化，我们也必须重写这个Mappable类，要不然编译器会捕捉到错误在fromMap这个方法中，但却不能捕捉到toMap。

#### 引入宏

很明显每次针对不同的case class都要定义不同的Mappable让我们的代码看起来不是那么的**DRY**，理想的情况下，Mappable应该能感知到case class中定义的每个field而不是要我们显示在方法中列出来。事实证明宏可以让你做到这一点，让我们来定义一个基本的宏伴随这个Mappable特质。

<pre>import scala.reflect.macros.whitebox.Context
object Mappable {
  implicit def materializeMappable[T]: Mappable[T] = macro materializeMappableImpl[T]
  def materializeMappableImpl[T: c.WeakTypeTag](c: Context): c.Expr[Mappable[T]] = {
    import c.universe._
    val tpe = weakTypeOf[T]
    val companion = tpe.typeSymbol.companion
    c.Expr[Mappable[T]] { q"""
      new Mappable[$tpe] {
        def toMap(t: $tpe) = ???
        def fromMap(map: Map[String, Any]):$tpe = ???
      }
    """ }
  }
}
</pre>

上面可以理解成一个模板，那我们就下来要做的就是如何用代码替换掉上面的???

#### 宏实现

* 获取字段

  <pre>val fields = tpe.decls.collectFirst {
    case m:MethodSymbol if m.isPrimaryConstructor => m
  }.get.paramLists.head
  </pre>

* toMap/fromMap

  <pre>val (toMapParams,fromMapParams) = fields.map { field =>
    val name = field.name.toTermName
    val decoded = name.decodedName.toString
    val returnType = tpe.decl(name).typeSignature
    (q"$decoded -> t.$name", q"map($decoded).asInstanceOf[$returnType]")
  }.unzip
  </pre>

* ??? 代替

  <pre>c.Expr[Mappable[T]] { q"""
    new Mappable[$tpe] {
      def toMap(t: $tpe) = Map(..$toMapParams)
      def fromMap(map: Map[String, Any]):$tpe = $companion(..$fromMapParams)
    }
  """ }
 </pre>

我们可以看到，其实宏就是在编译的时候把你返回的这个字段当成一段代码去编译，不管你想做什么，我们只要将其翻译成相应的字段并用q"""进行包装进行返回。

#### Demo展示

<pre>val mapper = Mappable.materializeMappable[People]
val p = People("bob.wang",12)
val map = mapper.toMap(p)
val anotherP = mapper.fromMap(map)
</pre>

这样不管你这个case class怎么变，上面的代码都不需要再去做额外处理了。
