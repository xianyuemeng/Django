Csrf
===

---

> CSRF（Cross-site request forgery）跨站请求伪造，也被称为“One Click Attack”或者Session Riding，通常缩写为CSRF或者XSRF，是一种对网站的恶意利用。尽管听起来像跨站脚本XSS，但它与XSS非常不同，XSS利用站点内的信任用户，而CSRF则通过伪装来自受信任用户的请求来利用受信任的网站。与XSS攻击相比，CSRF攻击往往不大流行（因此对其进行防范的资源也相当稀少）和难以防范，所以被认为比XSS更具危险性
> 
> **CSRF中间件和模板标签为防止跨站点请求伪造提供了易用的保护。**
> 
> 当恶意网站包含链接，表单按钮或某些旨在在您的网站上执行某些操作的JavaScript时，会使用在浏览器中访问恶意网站的登录用户的凭据进行此类攻击。
> 
> 还介绍了一种相关攻击类型，即“登录CSRF”，攻击网站欺骗用户的浏览器，以便使用其他人的凭证登录到网站。


### csrf的使用

在django的模板中，提供了防止跨站攻击的方法，使用步骤如下：

* step1：在settings.py中启用'django.middleware.csrf.CsrfViewMiddleware'中间件，此项在创建项目时，默认被启用

* step2：在HTML的表单中添加标签

    ```python
    <form>
    { % csrf_token % }
    ...
    </form>
    ```

### 取消保护

如果某些视图不需要保护，可以使用装饰器csrf_exempt，模板中也不需要写标签，修改csrf2的视图如下 from django.views.decorators.csrf import csrf_exempt

```python
@csrf_exempt 
def csrf2(request):
    uname=request.POST['uname'] 
    return render(request,'booktest/csrf2.html',{'uname':uname}) 
    运行上面的两个请求，发现都可以请求
```

### 保护原理

加入标签后，可以查看源代码，发现多了如下代码

```html
<input type='hidden' name='csrfmiddlewaretoken' value='nGjAB3Md9ZSb4NmG1sXDolPmh3bR2g59' />
```

* 在浏览器的调试工具中，通过network标签可以查看cookie信息

* 本站中自动添加了cookie信息，如下图csrf3

* 查看跨站的信息，并没有cookie信息，即使加入上面的隐藏域代码，发现又可以访问了

* 结论：django的csrf不是完全的安全

* 当提交请求时，中间件'django.middleware.csrf.CsrfViewMiddleware'会对提交的cookie及隐藏域的内容进行验证，如果失败则返回403错误


### Ajax CSRF 认证

> GET 请求不需要 CSRF 认证，POST 请求需要正确认证才能得到正确的返回结果。

**如果使用Ajax调用的时候，就要麻烦一些。需要注意以下几点：**

* 在视图中使用**render** （而不要使用 render_to_response）
* 使用 jQuery 的 ajax 或者 post 之前 加入这个 js 代码

    ```js
    jQuery(document).ajaxSend(function(event, xhr, settings) {
        function getCookie(name) {
            var cookieValue = null;
            if (document.cookie && document.cookie != '') {
                var cookies = document.cookie.split(';');
                for (var i = 0; i < cookies.length; i++) {
                    var cookie = jQuery.trim(cookies[i]);
                    // Does this cookie string begin with the name we want?
                    if (cookie.substring(0, name.length + 1) == (name + '=')) {
                        cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                        break;
                    }
                }
            }
            return cookieValue;
        }
        function sameOrigin(url) {
            // url could be relative or scheme relative or absolute
            var host = document.location.host; // host + port
            var protocol = document.location.protocol;
            var sr_origin = '//' + host;
            var origin = protocol + sr_origin;
            // Allow absolute or scheme relative URLs to same origin
            return (url == origin || url.slice(0, origin.length + 1) == origin + '/') ||
                (url == sr_origin || url.slice(0, sr_origin.length + 1) == sr_origin + '/') ||
                // or any other URL that isn't scheme relative or absolute i.e relative.
                !(/^(\/\/|http:|https:).*/.test(url));
        }
        function safeMethod(method) {
            return (/^(GET|HEAD|OPTIONS|TRACE)$/.test(method));
        }

        if (!safeMethod(settings.type) && sameOrigin(settings.url)) {
            xhr.setRequestHeader("X-CSRFToken", getCookie('csrftoken'));
        }
    });
    ```

* 或者 **更为优雅简洁的代码（不能写在 .js 中，要直接写在模板文件中）**：

    ```js
    $.ajaxSetup({
        data: {csrfmiddlewaretoken: '{ { csrf_token } }' },
    });
    ```

> 这样之后，就可以像原来一样的使用 jQuery.ajax() 和 jQuery.post()了