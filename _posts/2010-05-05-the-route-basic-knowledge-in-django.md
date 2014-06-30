---
layout: post
title:  "Django路由配置基本知识"
date:   2010-05-05 14:17:59
categories: 
- code 
tags:
- python
- django

---

命名路由参数 vs. 非命名路由参数
----
![clear cache](/media/pic/name-url-django.PNG)

1. 如果匹配到命名路由，则优先选用，忽略掉非命名路由.
2. 把所有参数都传递到非命名路由中.

所有参数捕获的值都是字符串

给VIEW函数添加参数默认值
----
<pre>urlpatterns = patterns('',
    url(r'^blog/$', 'blog.views.page'),
    url(r'^blog/page(?P<num>\d+)/$', 'blog.views.page'),
)
def page(request, num="1"):
</pre>

视图处理函数前缀
----
<pre>urlpatterns = patterns('',
    url(r'^articles/(\d{4})/(\d{2})/$', 'news.views.month_archive'),
    url(r'^articles/(\d{4})/(\d{2})/(\d+)/$', 'news.views.article_detail'),
)
</pre>
我们可以改成
<pre>urlpatterns = patterns('news.views',
    url(r'^articles/(\d{4})/(\d{2})/$', 'month_archive'),
    url(r'^articles/(\d{4})/(\d{2})/(\d+)/$', 'article_detail'),
)
</pre>
这样我们就可以少打几个字了，需要注意的是，"."我们不需要手动加了，Django框架会自动加上。
如果有很多这种情况，那我们可以分开来写例子如下：
<pre>urlpatterns = patterns('',
    url(r'^$', 'myapp.views.app_index'),
    url(r'^(?P<year>\d{4})/(?P<month>[a-z]{3})/$', 'myapp.views.month_display'),
    url(r'^tag/(?P<tag>\w+)/$', 'weblog.views.tag'),
)</pre>
改成
<pre>urlpatterns = patterns('myapp.views',
    url(r'^$', 'app_index'),
    url(r'^(?P<year>\d{4})/(?P<month>[a-z]{3})/$','month_display'),
)
urlpatterns += patterns('weblog.views',
    url(r'^tag/(?P<tag>\w+)/$', 'tag'),
)</pre>
如果路由前面大部分一样，如下所示：
<pre>urlpatterns = patterns('wiki.views',
    url(r'^(?P<page_slug>\w+)-(?P<page_id>\w+)/history/$', 'history'),
    url(r'^(?P<page_slug>\w+)-(?P<page_id>\w+)/edit/$', 'edit'),
    url(r'^(?P<page_slug>\w+)-(?P<page_id>\w+)/discuss/$', 'discuss'),
    url(r'^(?P<page_slug>\w+)-(?P<page_id>\w+)/permissions/$', 'permissions'),
)</pre>那我们也可修改成如下样子：
<pre>urlpatterns = patterns('',
    url(r'^(?P<page_slug>\w+)-(?P<page_id>\w+)/', include(patterns('wiki.views',
    	url(r'^history/$', 'history'),
    	url(r'^edit/$', 'edit'),
    	url(r'^discuss/$', 'discuss'),
    	url(r'^permissions/$', 'permissions'),
	))
    ),
)</pre>
前面所讲的处理函数都是以字符串形式，我们也可以不用字符串形式
<pre>from mysite import views
urlpatterns = patterns('',
    url(r'^archive/$', views.archive),
    url(r'^about/$', views.about),
)</pre>

反转URL--避免硬编码
----
+ 在模板中使用url标签
+ 在Python的VIEW处理函数中，用django.core.urlresolvers.reverse()

假设有如下URL配置
<pre>urlpatterns = patterns('',
    url(r'^articles/(\d{4})/$', 'news.views.year_archive'),
)</pre>
在模板中，假设我们有个链接<pre>src='articles/2012'</pre>如果这样写的就是硬编码了，如果后期我们更改了，就会牵连到很多地方，那我们怎么避免呢？
![clear cache](/media/pic/url-in-template.PNG)
这样我们就避免了，路由不管如何修改，只要处理函数不变就行。

View处理函数中我们可以这样写：
<pre>from django.core.urlresolvers import reverse
from django.http import HttpResponseRedirect
def redirect_to_year(request):
    year = 2006
    url = reverse('news.views.year_archive', args=(year,))
    return HttpResponseRedirect(url)
</pre>

命名URL
----
很多情况下，不同的URL地址可能是相同的函数处理，如下所示，/archive跟/archive-summary都由archive这个函数处理。
<pre>urlpatterns = patterns('',
    url(r'^archive/(\d{4})/$', archive),
    url(r'^archive-summary/(\d{4})/$', archive, {'summary': True}),
)</pre>这个时候如果我们也用reverse('archive')或者url标签在模板中，就会造成困惑，因为不知道跳转到哪个url地址。此时我们可以给这些url命个名字，如下：
<pre>urlpatterns = patterns('',
    url(r'^archive/(\d{4})/$', archive, name="full-archive"),
    url(r'^archive-summary/(\d{4})/$', archive, {'summary': True}, name="arch-summary"),
)</pre>这样使用的时候，就变成了<pre>reverse('full-archive')</pre>框架也知道是指向archive/这个地址。