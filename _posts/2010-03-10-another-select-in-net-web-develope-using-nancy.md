---
layout: post
title:  "NET平台WEB开发中微软框架外的选择--Nancy"
date:   2010-03-10 14:17:59
categories: 
- code 
tags:
- c#
- api

---

NET平台做B/S开发，常见的比较正统的就是webform跟mvc两个框架，webform把B/S开发本质HTTP协议封装的死死的，将winform那套C/S模式搬到了B/S模式中，以至于好多做了几年B/S开发的伙计不知道http协议,不知道协议中状态码含义,不知道协议中各种响应的类型等等，mvc回归HTTP本质，扔掉了webform中广受诟病的各种控件，又让不少小伙伴很受伤，不知道怎么去开发，会了的又好像很了不起一样。其实不管如何，B/S开发，不管什么语言什么平台，说白了就是服务器端监听着某个端口，然后根据不同请求返回不同字符串，只是这字符串比较特殊，它要能让浏览器认识，仅此而已无它。

言归正传，介绍一款开源的框架，[Nancy](http://nancyfx.org/ "nancy org")，下面我们采用自集成的方式，简单来说就是让你不用布置在IIS上面，了解WCF的小伙伴对自集成的方式应该不会陌生。

我们需要添加如下两个程序集，
<pre>Nancy.dll,Nancy.Hosting.Self.dll</pre>

接下来继承NancyModule做如下自定义Moduld虚类，这个可以具体业务具体分析了，看是建还是不建。
<pre>public abstract class BaseModule:NancyModule{
    public BaseModule():base(){
    
    }
    public BaseModule(string modulePath):base(modulePath){
        
    }
}
</pre>

如果利用过asp.net mvc开发过项目的应该知道，在它的第二版中推出了area的概念，上面带参数的构造作用就在于此。

做个RootModule，当用户访问这个根地址的时候由来处理，代码如下
<pre>public class RootModule:BaseModule{
    public RootModule():base(){
        Get["/"] = parameters => { return "Homepage"; };
    }
}</pre>

这样假如你在本地监听的是http://localhost:8888/nancy/，那么你在浏览器里输入上述地址的时候浏览器中就会显示Homepage的字眼，parameters是一个key-value的字典，代表的是HTTP协议中前端GET，POST，DELETE，PUT过来的数据集。

在做另一个Moduel，
<pre>public class PhotoModule:BaseModule{
    public PhotoModule(): base("/photo"){
        Get["/{slug}"] = parameters => {
            return String.Format("Photo {0}", parameters.slug);
        };
        Post["/{slug}/addcomment"] = parameters => {
            string photoSlug = Convert.ToString(parameters.slug);
            return Response.AsRedirect("/photo/" + photoSlug);
        };
    ｝
｝
</pre>
可以看出，当我们访问http://localhost:8888/nancy/photo/1234的时候，将会在浏览器中打印出Photo 1234的字眼，当我们POST数据到http://localhost:8888/nancy/photo/aaa/addcomment的路径时，将重定向到http://localhost:8888/nancy/photo/aaa，从而显示Photo aaa字眼。

上面都是直接打印出一个字符串，我们可以通过设置Response的ContentType来达到返回json或者其它的功能。
<pre>public class JsonModule:BaseModule{
    public JsonModule():base(){
        Get["/"] = parameters => {   
            Context.Response.ContentType = "application/json";
            JavaScriptSerializer js = new JavaScriptSerializer();
            StringBuilder sb = new StringBuilder();
            js.Serialize(new { id = 1, name = "bob" }, sb);
            return sb;
        };
    }
}
</pre>

当然WEB开发中最主要的还是返回HTML，我们不可能在上面做字符串拼装，不现实，Nancy给我们考虑到了，相对来讲，还支持多种模板，Razor,Spark,DotLiquid,Markdown,Nustache都支持。
<pre>public class HtmlModule:BaseModule{ 
   public JsonModule():base(){
       Get["/testing"] = parameters => {
           return View["staticview", modelname];
       };
   }
}
</pre>
上面第一个参数是模板名称，第二个参数是传给模板的模型。

一般来讲，在做快速HTTP服务的时候，如果是NET平台，可以用其来做一个原型，实在是太方便了，特别是在当今移动终端及各种开放API的情况下，只提供数据，随便你们自己展示去。