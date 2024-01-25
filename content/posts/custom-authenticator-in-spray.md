---
categories:
  - code
date: 2015-08-03 20:17:59
tags:
  - scala
  - spray
title: Spray框架之自定义权限验证
---

[Spray](http://spray.io/)内置了一此标准权限验证，像HttpBasicAuthenticator，然而如果我们需要一些其它的验证怎么办？在这篇文章里我们会谈一下如何来自定义一个权限验证基于(ClientId,ClientToken)并添加到项目中。

# Spray权限验证原理
* authenticate directive包含如下两个签名方法
```scala
def authenticate[T](auth:=>Future[Authentication[T]])(implicit executor: ExecutionContext): Directive1[T]
def authenticate[T](auth: ContextAuthenticator[T])(implicit executor: ExecutionContext): Directive1[T]
```
而ContextAuthenticator跟Authentication其实是两个自定义类型
```scala
type Authentication[T] = Either[Rejection, T]
type ContextAuthenticator[T] = RequestContext ＝> Future[Authentication[T]]
```
换句话说，ContextAuthenticator其实是一个函数，需要一个RequestContext(这是一个对收到的请求做包装的类，也就是说，它包含此次请求的所有信息)做为入参，返回一个拒绝或authenticate此次请求。

# ClientId/Token Authenticator
* 我们的目的是创建一个自定义的验证器，通过在请求参数中查找clientId跟token两个参数值同时校验此请求是正确还是错误。
* 首先我们先创建一个类用来存储此次请求的clientId及token,并且一个方法用来从请求中解析出相应参数值
```scala
case class Credentials(clientId: String, token: String)
def extractCredentials(ctx: RequestContext): Option[Credentials] = {
	val queryParams = ctx.request.uri.query.toMap
  	for {
    	    id <- queryParams.get("clientId")
    	    secret <- queryParams.get("token")
  	} yield Credentials(id, secret)
}
```
* 接下来自定义一个我们自己的ContextAuthenticator，记住Authentication[T]其实就是一个Either[Rejection, T]的别名
```scala
val authenticator: ContextAuthenticator[Unit] = { ctx =>
     Future {
       val maybeCredentials = extractCredentials(ctx)
         maybeCredentials.fold[authentication.Authentication[Unit]](
           Left(AuthenticationFailedRejection(CredentialsMissing, List()))
         )( credentials =>
             credentials match {
               case Credentials(clientId, token) => Right({})
               case _ => Left(AuthenticationFailedRejection(CredentialsRejected, List()))
             }
         )
     }
 }
```
 
# 如何使用
* 在你需要验证的route上进行如下包装
```scala
def routes: Route = authenticate(authenticator) { authenticated =>
          uroutes ~ uotherroutes
}
```