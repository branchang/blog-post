---
title: flower celery 监控工具
date: 2019-07-29 11:05:00
categories: [monitor]
tags: [flower, celery]
comments: true
toc: true
---

flower是为celery 提供实时监控和web管理的工具.

## 特性
* 通过celery events 进行实时监控
  * task 监控与历史记录
  * task 详情，参数，开始时间，运行时长等
  * 图表统计
* 远程控制
  * 查看worker状态，统计
  * 停止，重启worker实例
  * 控制worker pool size，配置autoscale
  * 查看和修改worker实例消耗的队列
  * 查看当前正在运行的任务
  * 查看调度的任务(ETA/countdown)
  * 查看revoked与reserved 的任务
  * revoked and terminate task
  * Apply time and rate limits
  * Configuration viewer
* broker 监控
  * 所有celery queues的视图统计
  * 图表监控队列长度
* HTTP API
* Basic Auth, GitHub OAuth2 and Google OpenID authentication


## 安装
通过pip安装
```bash
$ pip install flower
```
```bash
$ pip install https://github.com/mher/flower/zipball/master#egg=flower
```

### 启动方式
```bash 
$ flower -A proj --port=5555
```
```bash 
$ celery flower -A proj --address=127.0.0.1 --port=5555
```
```bash 
$ celery flower -A proj --broker=amqp://guest:guest@localhost:5672//
```

## 配置
命令行方式
```bash
$ flower --auto_refresh=False
```
flowerconfig.py配置文件
```python
# RabbitMQ management api
broker_api = 'http://guest:guest@localhost:15672/api/'

# Enable debug logging
logging = 'DEBUG'
```
添加环境变量，FLOWER_前缀开头
```bash
$ export FLOWER_BASIC_AUTH=foo:bar
```
通过命令行传递的配置优先于配置文件.
配置文件可以通过--conf指定.

### 配置选项
celery 配置文件与命令行配置，同样可以传递给flower启动配置使用. flower的配置文件会覆盖celery的标准配置.
[Celery Configuration reference](http://docs.celeryproject.org/en/latest/userguide/configuration.html)

* address  
  给定地址运行http server
* auth  
  开启google openID 认证
* auto_refresh  
开启dashboards自动刷新, (by default, auto_refresh=True)
* basic_auth  
  开启http 基础认证，basic_auth是分割类型，username:password
* broker_api  
  url of RabbitMQ HTTP API 包括用户认证
* ca_certs  
  ca_certs 证书地址
* certfile  
  SSL 证书地址
* conf   
  配置文件路径.(by default, flowerconfig.py)
* db  
  持久化模式下的数据库文件.(by default, db=flower)
* debug  
  开启调试模式
* enable_events  
事件
  




