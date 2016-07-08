---
layout: post
title:  "如何在Django1.6结合Python3.4版本中使用MySql"
date:   2014-05-14 20:17:59
categories: 
- code 
tags:
- python
- mysql

---
最近赶下新潮，用起了Python3.4跟Django1.6，数据库依然是MySql。

悲剧的是在Python2.7时代连接MySql的MySQLdb还不支持Python3.4，还好，苦苦追问G哥，终于找到一款代替品，而且效果还不错，它就是[pymysql]()

下载之，安装。

关于在Diango1.6中DATABASES的设置也是一样不用做任何改动，跟以前使用MySQLdb的时候一样，如下：
<pre>DATABASES = {
	'default' : {
		'ENGINE': 'django.db.backends.mysql',
		'NAME': 'test',
		'USER': 'root',
		'PASSWORD': 'root',
		'HOST': '',
		'PORT': '', 
		'OPTIONS': {
			'autocommit': True,
		},
	}
}
</pre>

最关键的一点，在做完上述配置后，在站点的__init__.py文件中，我们需要添加如下代码：
<pre>import pymysql
pymysql.install_as_MySQLdb()</pre>

上面都做完好，就可以在django中访问mysql了。