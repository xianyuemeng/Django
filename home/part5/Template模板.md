Template模板
===

---

> 作为Web 框架，Django 需要一种很便利的方法以动态地生成HTML。最常见的做法是使用模板。
> 
> 模板包含所需HTML 输出的静态部分，以及一些特殊的语法，描述如何将动态内容插入。

## 模板引擎配置

> 模板引擎使用该TEMPLATES设置进行配置。这是一个配置列表，每个引擎一个。
>
>默认值为空。在 settings.py由所产生的startproject命令定义一个更有用的值：

```python
TEMPLATES = [
        {
            'BACKEND': 'django.template.backends.django.DjangoTemplates',
            'DIRS': [os.path.join(BASE_DIR,'templates')],
            'APP_DIRS': True,
            'OPTIONS': {
                'context_processors': [
                    'django.template.context_processors.debug',
                    'django.template.context_processors.request',
                    'django.contrib.auth.context_processors.auth',
                    'django.contrib.messages.context_processors.messages',
                ],
            },
        },
    ]
```
