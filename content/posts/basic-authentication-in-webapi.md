---
categories:
  - code
date: 2012-09-05 14:17:59
tags:
  - c#
  - api
title: Asp.net Web Api中的Basic Authentication认证
---

在如今各种OPEN API数不胜数的年代，如何保证调用方是受信任的，考虑到大多数OPEN　API都是基于HTTP协议，此系列打算就以**Basic Authentication**开始介绍。

正如名字所示，basic authentication是http协议的一部分，是非常简单基本的，它的工作原理如下:

1. 客户端向服务端请求资源
2. 服务端发现此资源需要认证并且客户端没提供资料说自己是谁，那么服务端返回401-Unauthorized status code,同时在响应头中添加WWW-Authenticate:Basic这个Key
3. 此时客户端向服务端发送请求头Authorization:Basic ********
4. 如果客户端的credentials经过服务器认证是正确的，那么向客户端返回200这个状态码，如果服务器认证不通过，则重复步骤2

整个流程如下图所示：
![Basic Authentication Flow](/images/basic-authentication.PNG)

Asp.net Web Api中我们可以继承DelegatingHandler并重写相应方法，代码如下:
```c#
public class AuthenticationHandler : DelegatingHandler 
{
    private const string SCHEME = "Basic";
    protected override System.Threading.Tasks.Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, System.Threading.CancellationToken cancellationToken)
    {
        var headers = request.Headers;
        if (headers.Authorization != null && SCHEME.Equals(headers.Authorization.Scheme))
        {
            Encoding encoding = Encoding.GetEncoding("iso-8859-1");
            string credentials = encoding.GetString(Convert.FromBase64String(headers.Authorization.Parameter));
            string[] parts = credentials.Split(':');
            string userId = parts[0].Trim();
            string password = parts[1].Trim();                 

            // TODO - Do authentication of userId and password against your credentials store here
            if (true)
            {                    
                IPrincipal principal = new CustomPrincipal(new CustomIdentity(userId));
                Thread.CurrentPrincipal = principal;
                if (HttpContext.Current != null)
                {
                    HttpContext.Current.User = principal;
                }
            }
        }
        var response = base.SendAsync(request, cancellationToken);
        return response.ContinueWith<HttpResponseMessage>(x =>
        {
            HttpResponseMessage message = x.Result;
            if (message.StatusCode == HttpStatusCode.Unauthorized)
            {
                message.Headers.WwwAuthenticate.Add(new AuthenticationHeaderValue(SCHEME));
            }
            return message;
        });
    }
}
```
CustomPrincipal跟CustomIdentity是IPrincipal及IIdentity的相应实现。
最后我们还需要把我们的这个Handler注册到系统中。
```c#
config.MessageHandlers.Add(new AuthenticationHandler());
```
接下来就是发布了。

最后我们说说如何利用NET最新的HttpClient来访问，这里发下唠叨，话说NET平台下有WebClient,WebRequest，加上这个HttpClient,大家能知道他们的具体区别吗？

HttpClient有两种方式来访问这种类型的服务。

方法一：
```c#
HttpClientHandler handler = new HttpClientHandler();
handler.Credentials = new NetworkCredential("username", "password");
using (HttpClient client = new HttpClient(handler)) {
    client.GetAsync("url").Result;
}
```

方法二：
```c#
using (HttpClient client = new HttpClient(handler)) {
    string creds = String.Format("{0}:{1}", "badrddi", "badrddi");
    byte[] bytes = Encoding.ASCII.GetBytes(creds);
    var header = new AuthenticationHeaderValue("Basic", Convert.ToBase64String(bytes));
    client.DefaultRequestHeaders.Authorization = header;
    client.GetAsync("url").Result;
}
```