---
layout: post
title:  uWSGI,nginx部署基于Django的cms开源项目mezzanine
tags: [Mezzanine]
---

很早就当搭建个人的cms系统，趁着休假期间时间充裕前后后用了几天时间终于勉强可以访问.  
vps使用的linode，10$每月，也曾经考虑DigtalOcean,不过网上评价不高,也不想冒险.  
  cms系统的选择问题，之前自己使用github提供的静态页面blog，想现看看基于Django的开源搭建，发现这样的项目不多而且使用的人也不少，直接导致了后面的一路坑.(会不会后悔没有使用wordpress : \ )  
  Mezzanine is a powerful, consistent, and flexible content management platform. Built using the Django framework, Mezzanine provides a simple yet highly extensible architecture that encourages diving in and hacking on the code.  

Django建议使用apache,mod_python部署，这里用ngnix,uwsgi,数据库选择postgres.  
vps系统为ubuntu14.04lts.  
## Mezzanine安装

{% highlight bash %}
$ pip install mezzaniz
{% endhighlight %}
或源码安装
{% highlight bash %}
$ python setup.py install
{% endhighlight %}
创建一个新的mezzanine项目
{% highlight bash %}
$ mezzanine-project project_name
$ cd project_name
{% endhighlight %}

创建和配置对应的数据库,执行下面的步骤之前要安装和配置好所使用的数据库，这里使用的是postgres
{% highlight bash %}
$ python manage.py createdb --noinput
{% endhighlight %}

运行development mode，模式访问http://127.0.0.1:8000，用于调试开发
{% highlight bash %}
$ python manage.py runserver
{% endhighlight %}

## postgres 安装配置
{% highlight bash %}
$ apt-get install postgresql-9.4
{% endhighlight %}
根据django文档提示还需要安装Psycopg.根据django文档提示还需要安装Psycopg
{% highlight bash %}
$ sudo port install py27-psycopg2
{% endhighlight %}

配置cms系统需要的数据库
修改mezzanine创建项目中的local_setting.py 文件对应的配置修改mezzanine创建项目中的$ 
{% highlight text %}
DATABASES = {
    "default": {
            # Add "postgresql_psycopg2", "mysql", "sqlite3" or "oracle".
            "ENGINE": "django.db.backends.postgresql_psycopg2",
            # DB name or path to database file if using sqlite3.  文本信息储存的的数据库，需要自己创建
            "NAME": "blog_db",
            # Not used with sqlite3. 链接用户名
            "USER": "postgres",  
            # Not used with sqlite3. 链接密码
            "PASSWORD": "*****",
            # Set to empty string for localhost. Not used with sqlite3.
            "HOST": "127.0.0.1",
            # Set to empty string for default. Not used with sqlite3.
            "PORT": "5432",
        }
}
{% endhighlight %}
>这里需要注意的是如果不想使用mezzanine默认的数据库sqlite3，需要自己创建先配置好local_setting.py文件再运行如下命令
{% highlight bash%}
$ python manage.py createdb --noinput
{% endhighlight %}


## nginx配置
{% highlight bash%}
sudo apt-get install nginx
sudo /etc/init.d/nginx start    # start nginx
{% endhighlight %}
配置部署站点nginx config文件文件
/etc/nginx/sites-enabled/blog_nignx.conf
{% highlight text %}
upstream mezzanine_uwsgi{
    server unix:////home/Bran/mysite.sock; # for a file socket
    #server 127.0.0.1:8001; # for a web port socket (we'll use this first)
}
server {
    listen 80 ;
    server_name 45.56.94.238; # substitute your machine's IP address or FQDN
    charset     utf-8;


    location / {
            root /home/Bran/bran_blog/;
            uwsgi_pass mezzanine_uwsgi;
            include /home/Bran/bran_blog/uwsgi_params;
        }

    location /static {
            autoindex on;
            alias /home/Bran/bran_blog/static;
            access_log off;
            log_not_found off;
        }


    location /media {
            alias /home/Bran/bran_blog/static/media;
        }

    location /robots.txt {
            alias /home/Bran/bran_blog/static;
            access_log off;
            log_not_found off;
        }

    location /favicon.ico {
            alias /home/Bran/bran_blog/static/img;
            access_log off;
            log_not_found off;
        }
}
{% endhighlight %}

配置ngnix访问静态文件的路径.
其中static media robots.txt 均为项目mezzanine项目创建时默认生成的，只需要配置正确路径即可

在运行ngnix之前先需要收集django项目的静态文件到配置对应的路径(static/), 首先需要编辑mysite/setting.py
添加收集的路径地址
{% highlight text %}
STATIC_ROOT = os.path.join(BASE_DIR, "static/")
{% endhighlight %}
接着再运行
{% highlight bash%}
python manage.py collectstatic
{% endhighlight %}

### 测试nginx
restart nginx 
{% highlight bash%}
sudo /etc/init.d/nginx restart
{% endhighlight %}
把某个含有内容的文件放到media目录下面, 命名为text.txt,对应我的路径为/home/Bran/bran_blog/static/media/text.txt.

尝试访问http://example.com/media/text.txt,如果能访问的文本中的文件就说明nginx 配置没有问题,可以正常进行服务

## uWSGI
The uWSGI project aims at developing a full stack for building hosting services
安装uWSGI
{% highlight bash%}
pip install uwsgi
{% endhighlight %}
测试脚本
{% highlight python%}
# test.py
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return [b"Hello World"] # python3
    #return ["Hello World"] # python2
{% endhighlight %}
运行uwsgi
{% highlight bash%}
uwsgi --http :8000 --wsgi-file test.py
{% endhighlight %}

测试是否成功
{% highlight bash%}
http://example.com:8000
the web client <-> uWSGI <-> Python
{% endhighlight %}

uwsgi测试我们的django项目.
本地测试
{% highlight bash%}
python manage.py runserver 0.0.0.0:8000
uwsgi --http :8000 --module bran_blog.wsgi
{% endhighlight %}
bran_blog.wsgi 表示以bran_blog命名的wsgi模块(load the specified wsgi module), 在当前项目中是bran_blog.wsgi
>这里需要注意的是在项目上级目录中运行，不要在项目当前目录运行，否则会包错误找不到bran_blog module.
the web client <-> uWSGI <-> Django

### 配置uwsgi_params
you will need the uwsgi_params file, which is available in the nginx directory of the uWSGI distribution, or from https://github.com/nginx/nginx/blob/master/conf/uwsgi_params
uwsgi_param 要与nginx.conf文件中的路径相对应

## nginx and uWSGI 

通过nginx访问test.py
{% highlight bash%}
uwsgi --socket :8001 --wsgi-file test.py
{% endhighlight %}
这里的8001是同bran_blog.conf 中的127.0.0.1:8001是对应的端口,nginx通过8001端口与uwsgi模块通信.
访问http://example.com:80
the web client <-> the web server <-> the socket <-> uWSGI <-> Python
最后浏览器呈现出来的信息是通过8001端口提供给数据

### 使用unix socket代替ports
只需要编辑配置文件 blog_nginx.conf
{% highlight text%}
server unix:///path/to/your/mysite/mysite.sock; # for a file socket
# server 127.0.0.1:8001; # for a web port socket (we'll use this first)
{% endhighlight %}
这里需要重启nginx

运行uWSGI
uwsgi --socket mysite.sock --wsgi-file test.py --chmod-socket=666
参数中的sock与配置文件中的sock一个是相对路径一个是绝对路径
>这里请注意系统创建sock的权限问题

### run .ini file 
uWSGI也支持ini文件配置运行
{% highlight text%}
# mysite_uwsgi.ini file
[uwsgi]

# Django-related settings
# the base directory (full path)
chdir           = /home/Bran
# Django's wsgi file
module          = bran_blog.wsgi
# the virtualenv (full path)
#home            = /home/Bran

# process-related settings
# master
master          = true
# maximum number of worker processes
processes       = 10
# the socket (use the full path to be safe
socket          = /home/Bran/mezzanine.sock
# ... with appropriate permissions - may be needed
chmod-socket    = 666
# clear environment on exit
vacuum          = true
{% endhighlight %}

运行
{% highlight bash%}
uwsgi --ini mysite_uwsgi.ini # the --ini option is used to specify a file
{% endhighlight %}

可以配置rc.local  添加到系统默认启动过程中

>这里遇到一个问题，processes设置大于1时候会有session问还没解决


**大概的配置流程就是如上，更详细的信息可以参考uWSGI官网的说明,说明的步骤比较详细对于我这个不懂django的新手,都可以完全配置成功.**<br>
[Setting up Django and your web server with uWSGI and nginx](http://uwsgi-docs.readthedocs.org/en/latest/tutorials/Django_and_nginx.html)

## 遇到的问题
由于自己对django不了解遇到一些问题，还有待解决，再有就是不太适应使用html格式来写文章  
1.uwsgi中processes配置为大于1运行之后，会有seesion问题导致多次重复登陆.  
2.更换themes不成功，mezzanine不多的免费[themes](https://github.com/renyi/mezzanine-themes)还没有配置成功  

涉及完成文章参考web url:<br>
[PythonBlogSoftware](https://wiki.python.org/moin/PythonBlogSoftware)<br>
[Setting up Django and your web server with uWSGI and nginx](http://uwsgi-docs.readthedocs.org/en/latest/tutorials/Django_and_nginx.html)<br>
[uwsgi](http://uwsgi-docs.readthedocs.org/en/latest/#)<br>
[uwsgi_params](https://github.com/nginx/nginx/blob/master/conf/uwsgi_params)<br>
[mezzaine-themes](https://github.com/renyi/mezzanine-themes)<br>
[customizing-mezzanine](http://rosslaird.com/blog/customizing-mezzanine/)<br>
[Mezzanine Django Linux Nginx快速python CMS部署](http://www.phodal.com/blog/Mezzanine-Django-Linux-Nginx/)<br>
[Psycopg](http://initd.org/psycopg/)<br>
[PostgreSQL](http://www.postgresql.org/)<br>
[Django + uWSGI + Nginx](https://liangsun.org/posts/django-plus-uwsgi-plus-nginx/)<br>
[Wordpress vs. Mezzanine](http://grokbase.com/t/gg/mezzanine-users/12byf39adw/wordpress-vs-mezzanine)<br>


[How To Deploy: Installing Mezzanine Theme](http://stackoverflow.com/questions/28024595/how-to-deploy-installing-mezzanine-theme) <br>
登陆问题<br>
[Django: Redirect to previous page after login](http://stackoverflow.com/questions/806835/django-redirect-to-previous-page-after-login?rq=1)<br>
[Why second user login redirects me to /accounts/profile/ url?](http://stackoverflow.com/questions/13186693/why-second-user-login-redirects-me-to-accounts-profile-url)<br>
[Django - Authentication - redirect](http://stackoverflow.com/questions/10244331/django-authentication-redirect)<br>
[Django admin authentication failure](http://stackoverflow.com/questions/2176172/django-admin-authentication-failure)<br>
[uwsgi-processes-lose-django-sessions](http://stackoverflow.com/questions/17725616/uwsgi-processes-lose-django-sessions)<br>
[Problem with session in Django after login](http://stackoverflow.com/questions/7560237/problem-with-session-in-django-after-login?rq=1)<br>
[Django: Redirect to previous page after login](http://stackoverflow.com/questions/806835/django-redirect-to-previous-page-after-login)<br>
[Not understanding Django admin login](http://stackoverflow.com/questions/16199463/not-understanding-django-admin-login)









