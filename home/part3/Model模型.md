Model模型
===

---

> 模型是你的数据的唯一的、权威的信息源。它包含你所储存数据的必要字段和行为。
> 
> 通常，每个模型对应数据库中唯一的一张表。

* 每个模型都是django.db.models.Model的一个Python 子类。
* 模型的每个属性都表示为数据库中的一个字段。
* Django 提供一套自动生成的用于数据库访问的API；
* 这极大的减轻了开发人员的工作量，不需要面对因数据库变更而导致的无效劳

## 模型与数据库的关系

> 模型(Model)负责业务对象和数据库的关系映射(ORM)

ORM是“对象-关系-映射”的简称，主要任务是：

* 根据对象的类型生成表结构
* 将对象、列表的操作，转换为sql语句
* 将sql查询到的结果转换为对象、列表

## 为什么要用模型?

Model是MVC框架中重要的一部分,主要负责程序中用于处理数据逻辑的部分。通常模型对象负责在数据库中存取数据

它实现了数据模型与数据库的解耦，即数据模型的设计不需要依赖于特定的数据库，通过简单的配置就可以轻松更换数据库

## 配置Mysql数据库

* 在当前环境中安装mysql

    ```bash
    sudo apt-get install mysql-server

    sudo apt install mysql-client

    sudo apt install libmysqlclient-dev
    ```

* 在当前python环境中安装 pymysql

    ```bash
    pip3 install pymysql
    ```

* **在mysql中创建数据库**

    ```bash
    create databases mydb default charset=utf8
    ```

* **在Django项目中配置数据库**

    修改settings.py文件中的DATABASE配置项

    ```python
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': 'mydb',#选择数据库的名,请确认你的mysql中有这个库
            'USER': 'root',
            'PASSWORD': '123456',
            'HOST': 'localhost',
            'PORT': '3306',
        }
    }
    ```

* **告诉Django在接下来的mysql操作中使用pymysql**

    打开``mysite/__init__.py``，写入以下代码导入pymysql：

    ```python
    import pymysql
    pymysql.install_as_MySQLdb()
    ```

## 开发流程

* 在models.py中定义模型类，要求继承自models.Model

    ```python
    from django.db import models

    # Create your models here.

    #用户信息模型
    class Users(models.Model):
        username = models.CharField(max_length=32)
        password = models.CharField(max_length=32)
        email = models.CharField(max_length=50)

    # class Meta:
    #    db_table = "polls_users"  # 指定表名
    ```

* 把应用加入settings.py文件的installed_app项

    编辑mysite/settings.py文件，并将项目应用文件名添加到该INSTALLED_APPS设置。

    ```python
    INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'myweb',
    ]
    ```

* 生成迁移文件

    ```bash
    python3 manage.py makemigrations
    ```

* 执行迁移

    ```bash
    python3 manage.py migrate
    ```

* 使用模型类进行crud操作