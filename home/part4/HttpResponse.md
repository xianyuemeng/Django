HttpResponse对象
===

---

* 在django.http模块中定义了HttpResponse对象的API
* HttpRequest对象由Django自动创建，HttpResponse对象由程序员创建
* 在每一个视图函数中必须返回一个HttpResponse对象,当然也可以是HttpResponse子对象

## 1.不用模板,直接返回数据

```python
from django.http import HttpResponse

def index(request):
    return HttpResponse('你好')
```

## 2.调用模板返回数据

```python
from django.http import HttpResponse

# 返回模板
return render(request,'user/edit.html',{'info':'你好'})
```

## 3.子类HttpResponseRedirect

> 重定向，服务器端跳转
> 构造函数的第一个参数用来指定重定向的地址
> 可以简写为 redirect

```python
from django.shortcuts import redirect
from django.core.urlresolvers import reverse

def index(request):
    return redirect(reverse('myindex')
```

## 4.子类JsonResponse

> 返回json数据，一般用于异步请求
> 帮助用户创建JSON编码的响应
> JsonResponse的默认Content-Type为application/json

```python
from django.http import JsonResponse

def index2(requeset):
    return JsonResponse({'list': 'abc'})
```

## 5.set_cookie方法

> Cookie 是由 Web 服务器保存在用户浏览器（客户端）上的小文本文件，它可以包含有关用户的信息。
>
> 服务器可以利用Cookies包含信息的任意性来筛选并经常性维护这些信息，以判断在HTTP传输中的状态。
>
> Cookies最典型的应用是判定注册用户是否已经登录网站

* ### 设置cookie

    ```python
     # 获取当前的 响应对象
    response = HttpResponse('cookie的设置')

    # 使用响应对象进行cookie的设置
    response.set_cookie('a','abc')

    # 返回响应对象
    return response
    ```

* ### 获取cookie

    ```python
    # 读取
    a = request.COOKIES.get('a',None)

    if a:
        return HttpResponse('cookie的读取：'+a)
    else:
        return HttpResponse('cookie不存在')
    ```
