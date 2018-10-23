Virtualenv
===

---

VirtualEnv用于在一台机器上创建多个独立的Python虚拟运行环境，多个Python环境相互独立，互不影响，它能够：

1. 在没有权限的情况下安装新套件
1. 不同应用可以使用不同的套件版本
1. 套件升级不影响其他应用


## ubuntu16.04安装:

```bash
$ [sudo] pip3 install virtualenv
```

## 创建虚拟环境

```bash
$ virtualenv venv
```

## 激活虚拟环境

```bash
$ source venv/bin/activate
```

当虚拟环境被激活了，Python解释器的位置会被添加到PATH中，但是这个改动并不是永久的；它只影响当前命令会话。提醒一下，你激活了虚拟环境，该激活命令会将环境的名称包含在命令提示符里面：

```bash
(venv) $
```

## 停止虚拟环境

当你在虚拟环境中完成工作并想回到全局Python解释器，在命令提示符中输入`deactivate`就可以了。

```bash
$ deactivate
```

---

## 使用pip安装python包

大多数的Python包是通过`pip`程序安装的，在创建虚拟环境的时候virtualenv会自动添加进去。当一个虚拟环境被激活后，pip程序的位置会被添加到`PATH`中。


> 注：如果你使用pyvenv创建虚拟环境在Python 3.3中，则必须手动安装pip。安装指令在pip网站上可以找到。在Python 3.4下，pyvenv会自动安装pip。

比如，安装Flask到虚拟环境中，使用下面的命令：

```bash
(venv)$ python
>>> import flask
>>>
```

如果需要安装的包比较多的时候，这样做会比较繁琐，我们还有一键安装的方法。首先新建一个文本文件，如：requirements.txt，然后将你需要安装的包名保存到该文件中(根据自己的需要)，如下：

```bash
Babel==1.3
Flask==0.10.1
Flask-Login==0.2.7
Flask-SQLAlchemy==1.0
Flask-WTF==0.9.3
Jinja2==2.7.1
SQLAlchemy==0.8.2
WTForms==1.0.5
Werkzeug==0.9.4
psycopg2==2.5.1
...
```

最后你只需要输入以下命令，所有需要的包就可以全部安装好了：

```bash
(venv)$ pip install -r requirements.txt
```

如果没有出现错误，祝贺你：安装成功了。

若要查看当前环境安装了哪些包，可以使用下面的命令：

```bash
(venv)$ pip freeze
```

还可以直接导出到文件中

```bash
(venv)$ pip freeze > requirements.txt
```


### 移除环境


删除虚拟环境只需通过停用虚拟环境并删除环境文件夹及其所有内容即可完成：

```bash
(ENV)$ deactivate
$ rm -r /path/to/ENV
```

