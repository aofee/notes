# Celery分布式任务队列
## 简介
基于Python的异步分布式任务队列。主要适用于异步处理一些比较耗时的任务。
## 概念
* broker 任务队列，这里主要是存任务的
* worker 处理任务，可以在有很多个worker来处理任务
* backend 存储结果，worker处理任务的结果存储在这里  
任务发布者发布任务，任务被存储在broker中。  
任务调度器将broker中的任务分配给一个或多个worker，worker执行完任务将结果存储在backend中。  
由这个结构就可以看出来celery的分布式和异步的特点：任务发布者只需要专注于发布任务，worker只需要专注于处理分配给自己的一个又一个任务。
## 安装及配置
```python
pip install celery
pip install redis # 我一般使用redis做broker和backend 官方推荐使用RabbitMQ和redis
```
配置的话  我就介绍几个我比较常用的配置
```python
app = Celery('data_handle', broker=BROKER, backend=BACKEND)
platforms.C_FORCE_ROOT = True 
# 允许celery在root下运行。celery默认不支持在root下运行 需要引入celery的platforms
app.config_from_object('django.conf:settings') # 引入我django的配置文件中有关celery的配置，以CELERY_开头
```
django setting
```python
CELERYD_MAX_TASKS_PER_CHILD = 100 
# 每个worker最多执行完100个任务就会被销毁。默认不限制 注：销毁该worker会新建一个worker
CELERY_TASK_RESULT_EXPIRES = 600 
# redis中存储的任务过期时间 默认为86400s 因为我的任务特别多并且我的任务不需要知道结果 所以我的任务数据只保存十分钟 
CELERY_TIMEZONE = 'Asia/Shanghai' 
# 设置时区
```
下面是我的celery完整配置 local/celery.py
```python
from __future__ import absolute_import, unicode_literals
from celery import Celery, platforms
from django.conf import settings
import os

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'scs.settings')

BROKER = 'redis://'+settings.REDIS_HOST+':'+str(settings.REDIS_PORT)+'/0'
BACKEND = 'redis://'+settings.REDIS_HOST+':'+str(settings.REDIS_PORT)+'/0'

app = Celery('data_handle', broker=BROKER, backend=BACKEND)
platforms.C_FORCE_ROOT = True
app.config_from_object('django.conf:settings')
app.autodiscover_tasks(lambda: settings.INSTALLED_APPS) # 寻找任务文件 每个app下的task.py
```

我的任务文件（我的任务文件和celery文件在一个位置）local/tasks.py
```python
from .celery import app
from local.management.commands.set_high_attention import Command

command = Command()


@app.task
def cal(symbol):
    '''任务放在此处'''
    return 'ook'
```


调用任务
```python
from local import tasks


tasks.cal.delay() # 调用任务，任务即进入任务队列

```

启动celery
```python
celery multi start celery -A local
```
以上只是celery很基础很入门的用法。
还有一些进阶用法，比如为任务指定不同的队列；为任务设置优先级
### 其他
celery不仅仅可以像上面这样玩，也可以定义计划任务