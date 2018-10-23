URLconf路由
===

---

> 一个干净优雅的URL方案是高质量Web应用程序中的一个重要细节。
> 
> Django可以让你设计URL，无论你想要什么，没有框架限制。
> 
> 要为应用程序设计URL，您可以非正式地创建一个名为URLconf（URL配置）的Python模块。
> 
> 这个模块是纯Python代码，是一个简单的Python模式（简单的正则表达式）到Python函数（您的视图）之间的映射。


## 1. Django是如何处理一个请求

当用户从Django供电的站点请求页面时，系统遵循以下算法来确定要执行的Python代码：

1. 首先Django确定要使用的根URLconf模块，通过ROOT_URLCONF来设置，具体在settings.py配置文件中。但是如果传入 HttpRequest对象具有urlconf 属性（由中间件设置），则其值将用于替换ROOT_URLCONF设置。

1. Django加载该Python模块并查找该变量 urlpatterns。这应该是一个Python的django.conf.urls.url()实例列表。

1. **Django按顺序运行每个URL模式，并在匹配所请求的URL的第一个URL中停止。**

1. 一旦正则表达式匹配，Django将导入并调用给定的视图，这是一个简单的Python函数（或基于类的视图）。

1. **如果没有正则表达式匹配，或者在此过程中的任何一点出现异常，Django将调用适当的错误处理视图。**

### 示例

以下是一个URLconf示例：

```python
from django.conf.urls import url

from . import views

urlpatterns = [
    url(r'^articles/2003/$', views.special_case_2003),
    url(r'^articles/([0-9]{4})/$', views.year_archive),
    url(r'^articles/([0-9]{4})/([0-9]{2})/$', views.month_archive),
    url(r'^articles/([0-9]{4})/([0-9]{2})/([0-9]+)/$', views.article_detail),
]
```

#### 说明：

* 要从URL捕获一个值，只需将其括起来。
* 没有必要添加一个主要的斜杠，因为每个URL都有。例如^articles，不是^/articles。
* 正则中的'r'正面的每个正则表达式字符串的中是可选的，但推荐使用。它告诉Python一个字符串是“raw” - 字符串中没有任何内容应该被转义。请参阅Dive Into Python的解释。

#### 示例请求：

* /articles/2005/03/将匹配列表中的第三个条目。Django会调用该函数 。views.month_archive(request, '2005', '03')
* /articles/2005/3/ 不符合任何网址格式，因为列表中的第三个条目需要两个数字的月份。
* /articles/2003/将匹配列表中的第一个模式，而不是第二个模式，因为模式是按顺序测试的，第一个模式是第一个测试通过。随意利用这些命令插入特殊情况。在这里，Django会调用该函数 views.special_case_2003(request)
* /articles/2003 将不匹配任何这些模式，因为每个模式要求URL以斜杠结尾。
* /articles/2003/03/03/将匹配最终模式。Django会调用该函数。views.article_detail(request, '2003', '03', '03')

**注意：每个捕获的参数都作为纯Python字符串发送到视图，无论正则表达式的匹配是什么，即使[0-9]{4}只会匹配整数字符串。**

---

## 2. 命名组

* 上述使用为简单实例，属于正则表达式非命名组（通过括号）捕获URL定位，并将它们作为位置参数传递给视图。

* 在更高级的使用中，我们可以使用正则表达式命名组来捕获URL定位，并将它们作为关键字 参数传递给视图。

* 在Python正则表达式中，正则表达式命名组的语法是(?P<name>pattern)，其中命名组中的命名就是name，并且 pattern是某些匹配的模式。

### 实例：

```python
from django.conf.urls import url

from . import views

urlpatterns = [
    url(r'^articles/2003/$', views.special_case_2003),
    url(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),
    url(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/$', views.month_archive),
    url(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/(?P<day>[0-9]{2})/$', views.article_detail),
]
```

这完成了与上一个例子完全相同的事情，有一个微妙的区别：**捕获的值被传递到视图函数作为关键字参数，而不是位置参数。**

#### 例如：

* 请求/articles/2005/03/调用函数 ，。views.month_archive(request, year='2005', month='03') 而不是 views.month_archive(request, '2005', '03')

* 请求/articles/2003/03/03/调用函数 。views.article_detail(request, year='2003', month='03', day='03')

* 在实践中，这意味着您的URLconfs稍微更明确，更不容易出现参数命令错误 - 您可以重新排序视图的函数定义中的参数。当然，这些好处是以简洁为代价的; 一些开发人员发现命名组语法丑陋而且太冗长。

### URLconf的搜索

URLconf将根据所请求的URL进行搜索，作为普通的Python字符串。这不包括GET或POST参数或域名。


例如：

* 在请求中https://www.example.com/myapp/，URLconf将寻找myapp/。

* 在请求中https://www.example.com/myapp/?page=3，URLconf将会查找myapp/。

URLconf不查看请求方法。换句话说，所有的请求方法(GET,POST,HEAD等)都将被路由到相同的URL功能。

### 指定用于视图参数的默认值

一个方便的技巧是为您的视图参数指定默认参数。下面是一个URLconf和view的例子：

在下面的示例中，**两个URL模式指向相同的视图views.page**- 但是第一个模式不会从URL捕获任何内容。

如果第一个模式匹配，该page()函数将使用它的默认参数num，"1"。

如果第二个模式匹配， page()将使用num正则表达式捕获的任何值。


```python
# URLconf
from django.conf.urls import url

from . import views

urlpatterns = [
    url(r'^blog/$', views.page),
    url(r'^blog/page(?P<num>[0-9]+)/$', views.page),
]

# View (in blog/views.py)
def page(request, num="1"):
    # Output the appropriate page of blog entries, according to num.
    ...
```
---

## 3. 错误处理

> 当Django找不到与请求的URL匹配的正则表达式时，或者异常引发时，Django将调用错误处理视图。
>
> 用于这些情况的视图由四个变量指定。它们的默认值对于大多数项目都是足够的，但通过覆盖其默认值可以进一步定制。
>
> 有关详细信息，请参阅自定义错误视图的文档。
>
> 这样的值可以在你的根URLconf中设置。在任何其他URLconf中设置这些变量将不起作用。
>
> 值必须是可调用的，或者代表视图的完整的Python导入路径的字符串，应该被调用来处理手头的错误条件。

变量是：

* handler400- 见django.conf.urls.handler400。
* handler403- 见django.conf.urls.handler403。
* handler404- 见django.conf.urls.handler404。
* handler500- 见django.conf.urls.handler500。

### 关于404错误

* 404的错误页面，在模板目录中创建一个404.html的页面，
* 在配置文件中 settings.py DEBUG=False
* 在出现404的情况时，自动寻找404页面。
* 也可以在视图函数中 手动报出404错误，带提醒信息


在视图函数中也可以指定返回一个404

```python
注意 Http404需要在django.http的模块中引入
# 响应404
raise Http404('纳尼a')
```

在模板中 404.html

```html
<!DOCTYPE html>
<html>
<head>
    <title>404</title>
</head>
<body>
    <center>
        <h2>404 not found</h2>
        <h3>{ {   exception   } }</h3>
    </center>
</body>
</html>
```

---

## 4. 包括其他的URLconf

> 在任何时候，您urlpatterns都可以“包含”其他URLconf模块。
> 
> 这实质上是将一组网址“植根于”其他网址之下

例如，下面是Django网站本身的URLconf的摘录。它包含许多其他URLconf：

```python
from django.conf.urls import include, url

urlpatterns = [
    # ... snip ...
    url(r'^community/', include('django_website.aggregator.urls')),
    url(r'^contact/', include('django_website.contact.urls')),
    # ... snip ...
]
```

请注意，此示例中的正则表达式没有$（字符串尾匹配字符），但包含尾部斜线。

每当Django遇到`include()`(`django.conf.urls.include()`)时，它会截断与该点匹配的URL的任何部分，并将剩余的字符串发送到包含的URLconf以供进一步处理。

---

## 5. URL的反向解析

如果在视图、模板中使用硬编码的链接，在urlconf发生改变时，维护是一件非常麻烦的事情

* 解决：在做链接时，通过指向urlconf的名称，动态生成链接地址
* 视图：使用django.core.urlresolvers.reverse()函数
* 模板：使用url模板标签
