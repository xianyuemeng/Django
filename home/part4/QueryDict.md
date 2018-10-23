QueryDict对象
===

---

* 定义在django.http.QueryDict

* request对象的属性GET、POST都是QueryDict类型的对象

* 与python字典不同，QueryDict类型的对象用来处理同一个键带有多个值的情况

* 方法get()：根据键获取值

    * 只能获取键的一个值
    * 如果一个键同时拥有多个值，获取最后一个值

        ```python
        dict.get('键',default)
        或简写为
        dict['键']
        ```

* 方法getlist()：根据键获取值

    * 将键的值以列表返回，可以获取一个键的多个值

        ```python
        dict.getlist('键',default)
        ```

---

GET && POST
===

> 一键一值

```python
http://127.0.0.1/get?a=1&b=2&c=3

request.GET['name']
request.GET.get('name',None)
```


> 一键多值

```python
http://127.0.0.1/get?a=1&a=2&b=3

request.GET.getlist('name',None)

request.POST.getlist('name',None)
```

