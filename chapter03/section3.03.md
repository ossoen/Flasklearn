# Flask-MongoEngine

[MongoDB](https://www.mongodb.com/) 是一个文档型数据库，是 NoSQL (not only SQL) 的一种，具有灵活、易扩展等诸多优点，受到许多开发者的青睐。[MongoEngine](https://github.com/MongoEngine/mongoengine) 是一个用来操作 MongoDB 的 ORM 框架，如果你不知道什么是 ORM，可以参考 Flask-SQLAlchemy 一节。在 Flask 中，我们可以直接使用 MongoEngine，也可使用 [Flask-MongoEngine](http://docs.mongoengine.org/projects/flask-mongoengine/en/latest/) ，它使得在 Flask 中使用 MongoEngine 变得更加简单。


# 安装

使用 pip 安装，如下：

```
$ pip install flask-mongoengine
```

# 使用

在使用之前，请确保 mongo 服务已经开启。

## 配置

我们需要配置 MongoDB 的相关参数，以便我们能访问数据库。

```python
# -*- coding: utf-8 -*-

from flask import Flask
from flask_mongoengine import MongoEngine

app = Flask(__name__)
app.config['MONGODB_SETTINGS'] = {
    'db': 'test',
    'host': '127.0.0.1',
    'port': 27017
}

db = MongoEngine(app)
```

上面的代码中，我们在 `app.config` 的 `MONGODB_SETTINGS` 字典中配置了数据库、主机和端口。如果数据库需要身份验证，那我们需要在该字典中添加 `username` 和 `password` 参数，比如：

```python
app.config['MONGODB_SETTINGS'] = {
  'db': 'test', 
  'username':'admin', 
  'password':'12345'
}
```

另外，上面的配置也可以改成下面的方式：

```python
app.config['MONGODB_DB'] = 'test'
app.config['MONGODB_HOST'] = '127.0.0.1'
app.config['MONGODB_PORT'] = 27017
app.config['MONGODB_USERNAME'] = 'admin'
app.config['MONGODB_PASSWORD'] = '12345'
```

如果我们想在应用初始化前配置数据库，比如使用工厂方法，可以类似这样做：

```python
from flask import Flask
from flask_mongoengine import MongoEngine
db = MongoEngine()

...

app = Flask(__name__)
app.config.from_pyfile('config.json')
db.init_app(app)
```

## 定义数据模型

接下来，我们需要定义数据模型。这里，我们以一个 `Todo` 数据库为例，数据模型定义如下：

```python
from datetime import datetime

class Todo(db.Document):
    meta = {
        'collection': 'todo',
        'ordering': ['-create_at'],
        'strict': False,
    }

    task = db.StringField()
    create_at = db.DateTimeField(default=datetime.now)
    is_completed = db.BooleanField(default=False)
```

在上面的代码中，我们定义了一个 `Todo` 类，`meta` 字典设置了 `collection`，`ordering` 和 `strict`，其中 `ordering` 的值可以指定你的 [QuerySet](http://docs.mongoengine.org/guide/querying.html) 的默认顺序，`strict` 的值指定是否使用严格模式，默认值是 `True`，也就是使用严格模式，这就意味着如果数据库的记录如果存在某些字段没有在我们的数据模型中声明，那程序在运行时会产生一个 `FieldDoesNotExist` 的错误。因此，我们的数据模型定义最好跟记录中的字段保持一致。

## 查询数据

- 查询所有数据使用 `all()` 方法

```
todos = Todo.objects().all()
```

- 查询满足某些条件的数据

```
task = 'cooking'
todo = Todo.objects(task=task).first()
```

其中，`first()` 方法会取出满足条件的第 1 条记录。

## 添加数据

- 添加数据使用 `save()` 方法

```
todo1 = Todo(task='task 1', is_completed=False)
todo1.save()
```


## 数据排序

- 排序使用 `order_by()` 方法

```
todos = Todo.objects().order_by('create_at')
```

## 更新数据

- 更新数据需要先查找数据，然后再更新

```python
task = 'task 1'
todo = Todo.objects(task=task).first()  # 先查找
if not todo:
    return "the task doesn't exist!"

todo.update(is_completed=True)   # 再更新
```

## 删除数据

- 删除数据使用 `delete()` 方法：先查找，再删除

```python
task = 'task 6'
todo = Todo.objects(task=task).first()  # 先查找
if not todo:
    return "the task doesn't exist!"

todo.delete()   # 再删除
```

## 分页

- 分页可结合使用 `skip()` 和 `limit()` 方法

```python
skip_nums = 1
limit = 3

todos = Todo.objects().order_by(
    '-create_at'
).skip(
    skip_nums
).limit(
    limit
)
```

- 使用 `paginate()` 方法

```python
def view_todos(page=1):
    todos = Todo.objects.paginate(page=page, per_page=10)
```

本文完整的代码在[这里](https://github.com/ethan-funny/flask-demos/blob/ext/flask-mongoengine-demo/app.py)。

# 更多阅读

- [flask-mongoengine](http://docs.mongoengine.org/projects/flask-mongoengine/en/latest/)

