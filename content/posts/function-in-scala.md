---
categories:
  - code
date: 2017-06-05 20:17:59
tags:
  - scala
  - fp
  - map
  - case class
title: Monid/Functor的一点认识
---

一个单子(Monad)说白了不过就是自函子范畴上的一个幺半群, 此话出自Haskell大神 Philip Wadler. 在自学scala的过程中, 好几次尝试着去学习Monad这些概念, 每次都半途而废, 这次算是记录下过程, 虽然中间还是有些糊涂. 从编程角度来讲, 感觉只要把它理解成一种带上下文的顺序执行结构就差不多了

### 术语
* 单子(Monad)
* 自函子(Endo-Functor)
* 幺半群(Monoid)
* 范畴(category)

#### 范畴
我们可以简单的理解为函数 `f:Int=>String = ...`
* 一组对象
* 一组态射(morphisms),态射会绑定两个对象,如f是对象A到B的态射,记作 `f:A->B`
* 组合, 假设h是态射f跟g的组合,记作 `h = g o f`

#### 函子
本质上范畴间的转换,如对范围 C, D, 函子F:C=>D, 将C中任意对象A转为D的F(A)

#### 自函子
是一种将范畴映射到自身的函子 

#### 群
表示一个拥有封闭性,结合律,单位元,有逆元的二远运算代数结构 
* 半群  只满足封闭性和结合律的群
* 幺半群  在满足半群的同时还有一个单位元

### Code In Scala

#### 半群的代码表示
```scala
trait Group[A] {
  def op(a1:A, a2:A): A
}
```

#### 幺半群的代码表示
```scala
trait Monoid[A] extends Group[A]{
  def zero:A
}
```

#### Functor的代码表示
[其实就是一个可以对包装过的值做处理的函数](https://github.com/bobxwang/scala-tour/blob/master/src/main/scala/com/bob/scalatour/fp/ffunc.scala)
```scala
trait Functor[F[_]] {
  def map[A,B](a:F[A])(f:A=>B):F[B]
}
```

#### Monad的代码表示 
```scala
trait Monad[M[_]] {
 def unit[A](a: A): M[A]
 def flatMap[A, B](fa: M[A])(f: A => M[B]): M[B]
}
```