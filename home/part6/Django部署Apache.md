Django部署(Apache)
===

---

> 在前面的章节中我们使用python3 manage.py runserver来运行服务器。这只适用测试环境中使用。
> 
> 正式发布的服务，我们需要一个可以稳定而持续的服务器，比如Apache, Nginx, IIS等，本文将以 Apache为例。
> 
> 使用Apache和mod_wsgi部署Django 是一种久经考验的将Django投入生产的方法。
> 
> mod_wsgi是一个Apache模块，可以托管任何Python WSGI应用程序，包括Django。
> 
> Django将使用任何支持mod_wsgi的Apache版本。


### 测试环境

* Ubuntu 16.04
* Python 3.5.2
* Django 1.11.7
* Apache 2.4

---

## 配置步骤

* ### 1.Apache2安装

    ```bash
    Apache2安装
    sudo apt-get install apache2

    查看版本

    apachectl -v

    Server version: Apache/2.4.18 (Ubuntu)

    Server built: 2017-09-18T15:09:02
    ```

* ### 2.确保有127.0.0.1 localhost，没有就加上。

    ```bash
    sudo vim /etc/hosts

    127.0.0.1       localhost
    127.0.0.1       www.pyweb.cn
    ```

* ### 3.打开浏览器 输入 127.0.0.1或localhost

    ```bash
    出现 Apache2 Ubuntu Default Page

    或It works!

    则成功
    ```

* ### 4.安装apache2解析python的包 wsgi程序包

    ```vim
    sudo apt-get install libapache2-mod-wsgi-py3

    安装完成后 进入 /usr/lib/apache2/modules 目录
    cd /usr/lib/apache2/modules
    查看是否存在mod_wsgi.so-3.5
    ```

* ### 5.配置使apache2加载mod-wsgi包

    ```vim
    编辑配置文件
    sudo vim /etc/apache2/apache2.conf

    在文件的最后 添加

    LoadModule wsgi_module /usr/lib/apache2/modules/mod_wsgi.so-3.5
    ```

* ### 6.创建网站配置文件

    ```vim
    编辑网站配置文件
    sudo vim /etc/apache2/sites-available/myproject.conf

    配置内容:
    <VirtualHost *:80>
    ServerName www.pyweb.cn
    ServerAdmin py@163.cn
    #wsgi文件目录
    WSGIDaemonProcess python-path=/var/www/myproject
    WSGIScriptAlias / /var/www/myproject/myproject/wsgi.py
    <Directory /var/www/myproject/myproject>
        <Files wsgi.py>
            Require all granted
        </Files>
    </Directory>
    #项目文件目录
    DocumentRoot /var/www/myproject
    <Directory /var/www/myproject>
        Require all granted
    </Directory>
    #静态文件目录
    Alias /static/ /var/www/myproject/static/
    <Directory /var/www/myproject/static/>
        Require all granted
    </Directory>
    #错误日志
    ErrorLog ${APACHE_LOG_DIR}/django-myproject-error.log
    CustomLog ${APACHE_LOG_DIR}/myproject-django.log combined
    </VirtualHost>
    ```

* ### 7.将当前的配置文件创建一个软连接到/etc/apache2/sites-enabled

    ```vim
    cd /etc/apache2/sites-enabled

    sudo ln -s ../sites-available/myproject.conf ./
    ```

* ### 8.执行命令 生效当前配置

    ```vim
    
    ```

* ### 8.执行命令 生效当前配置

    ```vim
    sudo a2ensite myproject.conf

    如果需要让这个配置失效,可以执行 sudo a2dissite myproject.conf
    ```

* ### 9.配置Django项目目录及修改seeting.py文件，

    ```vim
    首先把myproject项目目录拷贝至 /var/www/目录下

    在将其ALLOWED_HOSTS=[]改为

    ALLOWED_HOSTS=['www.pyweb.cn']，多个域名可以通过逗号隔开。
    ```

* ### 10.修改Django的wsgi.py文件

    ```python
    import os
    os.environ["DJANGO_SETTINGS_MODULE"] = "myproject.settings"
    #os.environ.setdefault("DJANGO_SETTINGS_MODULE", "pyjfive.settings")

    from os.path import join,dirname,abspath
    PROJECT_DIR = dirname(dirname(abspath(__file__)))

    import sys # 4
    sys.path.insert(0,PROJECT_DIR)


    from django.core.wsgi import get_wsgi_application
    application = get_wsgi_application()
    ```

* ### 最后:重启apache2

    ```vim
    sudo service apache2 restart
    ```

* ### 浏览器访问

    ```vim
    http://www.pyweb.cn/
    ```

* ### 浏览器错误 500

    ```vim
    Internal Server Error
    The server encountered an internal error or misconfiguration and was unable to complete your request.
    Please contact the server administrator at [no address given] to inform them of the time this error occurred,
    and the actions you performed just before this error.
    More information about this error may be available in the server error log.
    Apache/2.4.18 (Ubuntu) Server at www.py6web.com Port 80
    ```

* #### 1.查看apache2的错误日志

    ```vim
    cd /var/log/apache2/

    File "/var/www/myproject-test/myproject/wsgi.py", line 17, in <module>, referer: http://www.pyweb.com/
    from django.core.wsgi import get_wsgi_application, referer: http://www.pyweb.com/
    ImportError: No module named 'django', referer: http://www.pyweb.com/
    ```

* #### 2.问题分析

    ```vim
    进入项目目录，使用命令 pip3 show Djando 查看当前是否已经安装django
    ---
    Metadata-Version: 1.1
    Name: Django
    Version: 1.11.8

    切换至root用户 sudo su 

    进入python3的shell模式
    python3
    #加载django模块
    import django
    #错误：No module named 'django'
    ```

* #### 3.解决方案

    ```vim
    在当前root用户下 安装django
    sudo su
    pip3 install django==1.11
    ```

* ### 文件上传错误:

    * #### 给static文件夹或web项目目录文件递归追加一个www-data用户权限
    ```vim
    sudo setfacl -R -m u:www-data:rwx web/
    ```