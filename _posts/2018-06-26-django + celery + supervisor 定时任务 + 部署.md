---
author: zyqzyq
date: 2018-06-26 16:16:32+00:00
layout: post
title: django + celery + supervisor 定时任务 + 部署
key: 20180626
tags:
- django
- celery 
- supervisor
---

## 环境
    django 2.0.6 (上一篇文章有详细项目新建的描述)
    celery 4.2.0
    django-celery-beat 1.1.1
    django-celery-results 1.0.1
    
# 安装配置celery

### 选择broker
    我选择了RabbitMQ(官方推荐，安装配置简单) [了解更多](http://docs.celeryproject.org/en/latest/getting-started/first-steps-with-celery.html#choosing-a-broker)
    安装RabbitMQ
    
```
sudo apt-get install rabbitmq-server
```
添加配置到setting.py末尾

```
# celery broker
BROKER_URL = 'amqp://localhost//'

```

### 安装celery
    celery是一个python包所以可以通过pip直接安装
    
```
cd ~/user/myproject
pipenv shell
pipenv install celery 
```
    
    celery 3.1版本以后不需要再安装django-celery模块。
添加配置到setting.py末尾

```

CELERY_ACCEPT_CONTENT = ['pickle']
CELERY_TASK_SERIALIZER = 'pickle'
CELERY_RESULT_SERIALIZER = 'pickle'

# celery时区设置，使用settings中TIME_ZONE同样的时区
CELERY_TIMEZONE = TIME_ZONE

```

### 安装django-celery-results
    django-celery-results使用django orm提供结果。
安装 django-celery-restults

```
pip install django-celery-results

```
    添加 django-celery-restults 到setting.py


```
INSTALLED_APPS = (
    ...,
    'django_celery_results',
)
```
    添加配置到setting.py末尾

```
# celery结果返回
CELERY_RESULT_BACKEND = 'django-db'

```

### 安装django-celery-beat
自定义调度程序类可以在命令行中指定（--scheduler参数）。

默认调度程序是celery.beat.PersistentScheduler，它只是跟踪本地搁置数据库文件中的最后一次运行时间。

还有一个django-celery-beat扩展，它将计划存储在Django数据库中，并提供了一个方便的管理界面来管理运行时的定期任务。
    
安装 django-celery-beat

```
pip install django-celery-beat

```
添加 django-celery-beat 到setting.py


```
INSTALLED_APPS = (
    ...,
    'django_celery_beat',
)
```
添加配置到setting.py末尾

```
# celery 定时任务
CELERY_BEAT_SCHEDULER = "django_celery_beat.schedulers:DatabaseScheduler"

```
启动任务

```
celery -A proj beat -l info --scheduler django_celery_beat.schedulers:DatabaseScheduler
```
访问Django后台界面设置定期任务。

# 添加任务
新建一个项目

```
python manage.py startapp demoapp
```
添加demoapp/tasks.py
@shared_task修饰器可以让你创建任务而不需要任何具体的应用程序实例

```
# Create your tasks here
from __future__ import absolute_import, unicode_literals
from celery import shared_task


@shared_task
def add(x, y):
    return x + y


@shared_task
def mul(x, y):
    return x * y


@shared_task
def xsum(numbers):
    return sum(numbers)
```

# supervisor 配置

pip 安装supervisor (建议用系统自带的python2进行安装，py3不确定是否支持)

```
pip install supervisor
```

生成配置文件

```
echo_supervisord_conf > /etc/supervisord.conf
```
编辑配置文件，在末尾添加

```
[program:celery.worker]
command= myprojectenv/bin/celery -A myproject worker -l info
directory=/home/user/myproject
numprocs=1
autostart=true
autorestart=true
startretries=3
redirect_stderr=true
stdout_logfile=/home/home/user/myproject/log/celery_worker_out.log

[program:celery.beat]
command= myprojectenv/bin/celery -A myproject beat -l info
directory=/home/user/myproject
numprocs=1
autostart=true
autorestart=true
startretries=3
redirect_stderr=true
stdout_logfile=/home/home/user/myproject/log/celery_beat_out.log

```
启动supervisor

```
supervisord -c supervisord.conf
```

至此，大功告成。
django的配置可以按上一篇文章进行配置。


参考文件：
[http://docs.celeryproject.org/en/latest/django/first-steps-with-django.html](http://docs.celeryproject.org/en/latest/django/first-steps-with-django.html)

[http://docs.celeryproject.org/en/latest/getting-started/first-steps-with-celery.html](http://docs.celeryproject.org/en/latest/getting-started/first-steps-with-celery.html)


