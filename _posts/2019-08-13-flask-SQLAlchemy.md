---
title: sqlalchemy MySQL server has gone away
date: 2019-08-13 14:05:00 +0800
categories: [Question]
tags: [Flask,MySQL]
comments: true
toc: true
---

最近做的监控项目使用到了flask_sqlalchemy，因为大部分接口都是读取DB或ES返回数据，想来用ORM的方式还是比较方便的，
不过使用的过程中也遇到了一些坑。

初期调试命令行启动app查询数据一切正常，后续使用gunicorn部署后对接口进行测试，发
现会出现概率包错，错误信息如下:

```bash 
OperationalError: (OperationalError) (2006, 'MySQL server has gone away')
```

初步分析感觉是当父进程创建了子进程之后，子进程会调用父进程的mysql链接；当这个
子进程退出时，连接被关闭；当父进程创建新的子进程时，这个子进程同样调用了父进程
的mysql连接，而此时mysql连接已经被上一个进程关闭了，但当前子进程并不知道.
由此思路尝试了解决方法.


1. 每次请求查询完成之后关闭session

```python
@app.after_request
def after_app_request(response):
    db.session.close()
    return response
```
或者
```python
@app.teardown_appcontext
def shutdown_session(exception=None):
    db_session.remove()
```

测试后发现无效

2. 每次请求前重新初始化sqlalchemy 链接
```python 
db = SQLAlchemy(app)
```
测试方法有效，有些粗暴，影响接口效率


方案2虽然可以成功不在暴露2006的问题，但是个人觉得实现的方式并不优美。于是又查了
一些方案，

3. 设置SQLALCHEMY_POOL_RECYCLE 数值小一点
```python
app.config['SQLALCHEMY_POOL_RECYCLE'] = 10
```
测试之后发现有效。线上查看mysql配置
```bash
show global variables like '%timeout%';
```
发现是由于wait_timeout值(12)设置的过小。wait_timeout数据库的默认值一般是
28800，即8小时，配置含义是服务器关闭非交互连接之前等待活动的秒数。而sqlalchemy默认设置是7200。
所以问题是由于wait_timeout设置的过小，mysql服务端自动断开连接，api中的session已经失效，导致api请求报错。
如果有数据库操作权限的话也可以修改wait_timeout配置大一些.

还有其他一些解决方法，此处先记录一下

4. 出错后需要rollback，为了后续程序能运行，给每个涉及sql语句的函数用了装饰器

```python 
def auto_rollback(func):
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except Exception as err:
            db.session.rollback()
            log.error(err)
            raise err

    return wrapper
```

5. SQlAlchemy官方：设置pool_pre_ping=True

官方文档其实对于myslq断链的问题，官方除了设置pool_recycle之外还建议在创建Engine的时候设置pool_pre_ping=True也就是在每一次使用Session之前都会进行简单的查询检查，判断是Session是否过期。
```python
engine = create_engine("mysql+pymysql://user:pw@host/db", pool_pre_ping=True)
```

6. 多进程相关配置,Pool使用事件来检测自身，以便在子进程中自动使连接无


```python
from sqlalchemy import event
from sqlalchemy import exc
import os

engine = create_engine("...")

@event.listens_for(engine, "connect")
def connect(dbapi_connection, connection_record):
    connection_record.info['pid'] = os.getpid()

@event.listens_for(engine, "checkout")
def checkout(dbapi_connection, connection_record, connection_proxy):
    pid = os.getpid()
    if connection_record.info['pid'] != pid:
        connection_record.connection = connection_proxy.connection = None
        raise exc.DisconnectionError(
                "Connection record belongs to pid %s, "
                "attempting to check out in pid %s" %
                (connection_record.info['pid'], pid)
        )
```

### 参考方案如下

[https://stackoverflow.com/questions/18054224/python-sqlalchemy-mysql-server-has-gone-away](https://stackoverflow.com/questions/18054224/python-sqlalchemy-mysql-server-has-gone-away)

[https://www.jianshu.com/p/b4c59e2f1399](https://www.jianshu.com/p/b4c59e2f1399)

[http://cocode.cc/t/heroku-flask-python-2006-mysql-server-has-gone-away/6213](http://cocode.cc/t/heroku-flask-python-2006-mysql-server-has-gone-away/6213)

[https://www.cnblogs.com/lesliexong/p/8654615.html](https://www.cnblogs.com/lesliexong/p/8654615.html)

[http://librelist.com/browser//flask/2014/4/8/flask-sqlalchemy-mysql-server-has-gone-away/#d4cf769b0c17ec8116a2ab6a7b766d79](http://librelist.com/browser//flask/2014/4/8/flask-sqlalchemy-mysql-server-has-gone-away/#d4cf769b0c17ec8116a2ab6a7b766d79)

[https://flask-sqlalchemy.palletsprojects.com/en/2.x/config/](https://flask-sqlalchemy.palletsprojects.com/en/2.x/config/)

[https://blog.csdn.net/weixin_39956308/article/details/84816692](https://blog.csdn.net/weixin_39956308/article/details/84816692)