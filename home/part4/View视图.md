View视图
===

---

> Django具有“视图”的概念来封装**负责处理用户请求和返回响应的逻辑**
> 
> 视图函数或视图简而言之就**是一个Python函数，它接受一个Web请求并返回一个Web响应**
> 
> 此响应可以是网页的HTML内容，重定向或404错误，XML文档或图像。
> 
> 为了将代码放在某处，惯例是将视图`views.py`放在名为的文件中，放在项目或应用程序目录中


## 1. 一个简单的视图

这是一个返回当前日期和时间的视图，作为HTML文档：
```python
from django.http import HttpResponse
import datetime

def current_datetime(request):
    now = datetime.datetime.now()
    html = "<html><body>It is now %s.</body></html>" % now
    return HttpResponse(html)
```

## 2. 返回错误


```python
# 直接返回一个404,没有去加载404的模板页面
return HttpResponseNotFound('<h1>Page not found</h1>')

# 可以直接返回一个status状态码
return HttpResponse(status=403)

# 返回一个404的错误页面
raise Http404("Poll does not exist")
```


## 3.关于重定向

> 重定向(Redirect)就是通过各种方法将各种网络请求重新定个方向转到其它位置

```python
from django.shortcuts import redirect
from django.core.urlresolvers import reverse

# redirect重定向  reverse反向解析url地址
return redirect(reverse('userindex'))

# 执行一段js代码，用js进行重定向
return HttpResponse('<script>alert("添加成功");location.href = "/userindex"; </script>')

# 加载一个提醒信息的跳转页面
context = {'info':'数据添加成功','u':'/userindex'}
return render(request,'info.html',context)
```
