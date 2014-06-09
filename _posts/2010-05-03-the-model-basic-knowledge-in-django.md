---
layout: post
title:  "Django的Model基础知识"
date:   2010-05-03 14:17:59
categories: 
- code 
tags:
- python
- django

---
Django中的Model层，负责的是跟数据库表间的映射，同是也是一个ORM。数据库表间关系无非三种，Django中都做了很好的支持。下面是三个模型，我们从这入手。

<pre>class Publisher(models.Model):
    name = models.CharField(max_length=30)
    website = models.URLField()

    def __str__(self):
        return self.name

class Author(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=40)
    email = models.EmailField()

    def __str__(self):
        return u'%s %s' % (self.first_name, self.last_name)

class Book(models.Model):
    title = models.CharField(max_length=100)
    authors = models.ManyToManyField(Author)
    publisher = models.ForeignKey(Publisher)
    publication_date = models.DateField()

    def __str__(self):
        return self.title

b1 = Book.objects.filter(publication_date__gte=datetime(2006, 1, 1))
b2 = b1
 .filter(publisher__name__exact="Cleveland",publisher__website__exact="Ohio")
</pre> 

QuerySet是惰性的，这一点非常不错。这意味着只在对数据库进行求值之后才会对它们执行查询，这会比立即执行查询的速度更快。

当Model创建好后，我们需要同步到数据库中，不过这时我们首先要做的是把这个app注册到Django的配置文件中，然后执行python manage.py syncdb命令。如果我们想看看根据这些模型什么样的SQL语句会发送到数据库，那么我们执行这个命令python manage.py sql appname。不过syncdb命令在我们更改了模型后不会对表结构进行修改，除非我们手动把表给Drop表，再运行。但这样一来如果表里有数据就会比较麻烦，因此推荐[south](http://south.aeracode.org/ "south org")这个第三方app来解决这一问题。

ModelAdmin
---
光有Model还不行，Django最值得称道的就是它的后台管理，如果我们想要这些Model出现在后台管理中，我们需要在我们的app中进行注册，添加一个admin.py，写上如下代码：

<pre>class BookAdmin(admin.ModelAdmin):
    ordering = ["-publication_date"]
    list_display = ("title", "publisher","publication_date",)
    search_fields = ("title",)
    list_filter = ("publisher",)

admin.site.register(Publisher)
admin.site.register(Author)
admin.site.register(Book,BookAdmin)
</pre>

我们需要继承admin.ModelAdmin，然后在里面写上一些代码，这样就可以在Django自带的后台操作了。

list_display : 主要用于列表显示的时候显示哪些列。

list_filter : 在一对多的时候,可以显示指定1时候多的那一头数据。

Model Manager
---
管理器是Django查询数据库时会用到的一个特别的对象，在Book.objects.all()的语法中，objects就是一个管理器，每个Model至少有一个管理器，你也可以创建自己的管理器来定义数据库访问操作。

<pre>class BookManager(models.Manager):
    def title_count(self, keyword):
        return self.filter(title__icontains=keyword).count()

class Book(models.Model):
    objects = BookManager()

#来返回书名中含有python字眼的记录数。
Book.objects.title_count('python') 
</pre>

类管理器后可以更好的进行功能封装及代码重用。

Model 方法 
---
<pre>class Author(models.Model):
    def get_full_name(self):
        return u'%s %s' % (self.first_name, self.last_name)
</pre>
同管理器相比，Model方法更像是基于记录级别的方法，像上面，当我们得到一个Author的记录后，就可以调用get_full_name来返回全名。

<pre>p = Author.objects.get(first_name='Barack', last_name='Obama')
p.get_full_name()
</pre>

执行自宝义的SQL语句
---
<pre>class PythonBookManager(models.Manager):
    def books_titles_after_publication(self, date_string):
        cursor = connection.cursor()
        cursor.execute("""
            SELECT title
            FROM books_book
            WHERE publication_date > %s""", [date_string])
        #fetchall()返回的是元组的列表
        return [row[0] for row in cursor.fetchall()]
</pre>

可以使用SQL对数据库中所有的表进行操作，而不是引用特定的Model类，这样操作比较自由，比较好的一个做法是把它放在一个自定义的管理器中。

<pre>class Book(models.Model):
    objects = BookManager()
	customer_objects = PythonBookManager()

Book.customer_objects.books_titles_after_publication('2012-05-22')
</pre>






