# Flask-RESTful

在[前面][1]，我们介绍了 REST Web 服务，并使用 Flask 提供服务。这里，我们使用第三方库 [Flask-RESTful][2]，它使得在 Flask 中提供 REST 服务变得更加简单。

# 安装

使用 pip 安装：

```
$ pip install flask-restful
```

# 使用

下面我们主要使用[官方文档](http://flask-restful-cn.readthedocs.io/en/0.3.5/quickstart.html)的例子进行说明。

## Hello World

我们先来看一个简单的例子。

```python
# -*- coding: utf-8 -*-

from flask import Flask
from flask_restful import Resource, Api

app = Flask(__name__)
api = Api(app)

class HelloWorld(Resource):
    def get(self):
        return {'hello': 'world'}

api.add_resource(HelloWorld, '/')

if __name__ == '__main__':
    app.run(debug=True)
```

上面的代码应该不难看懂。我们定义了一个 `HelloWorld` 的类，该类继承自 `Resource`，在类里面，我们定义了 `get` 方法，该方法跟 HTTP 请求中的 `GET` 方法对应。接着，我们使用 `add_resource()` 方法添加资源，该方法的第 1 个参数就是我们定义的类，第 2 个参数是 URL 路由。

将上面的代码保存为 `app.py`，在终端运行：

```
$ python app.py
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger pin code: 109-566-036
```

使用 `curl` 测试，如下：

```
$ curl http://127.0.0.1:5000/
{
    "hello": "world"
}
```

另外，`add_resource()` 方法也支持添加多个路由，比如：

```
api.add_resource(HelloWorld, '/', '/hello')
```

这样访问 `http://127.0.0.1:5000/` 和 `http://127.0.0.1:5000/hello` 效果是一样的。


## 带参数的请求

Flask-RESTful 也支持路由带参数的请求，跟 Flask 的实现是类似的，看下面这个例子。

```python
# -*- coding: utf-8 -*-

from flask import Flask, request
from flask_restful import Resource, Api

app = Flask(__name__)
api = Api(app)

todos = {}

class TodoSimple(Resource):
    def get(self, todo_id):
        return {todo_id: todos[todo_id]}

    def put(self, todo_id):
        todos[todo_id] = request.form['data']
        return {todo_id: todos[todo_id]}

# 路由带参数
api.add_resource(TodoSimple, '/<string:todo_id>')

if __name__ == '__main__':
    app.run(debug=True)
```

使用 curl 测试，如下：

```
$ curl -X PUT http://localhost:5000/todo1 -d "data=shopping"
{
    "todo1": "shopping"
}

$ curl -X GET http://localhost:5000/todo1
{
    "todo1": "shopping"
}
```


## 参数解析

Flask-RESTful 提供了 `reqparse` 库来简化我们访问并验证表单的工作，将上面的代码使用 `reqparse` 改写，如下：

```python
# -*- coding: utf-8 -*-

from flask import Flask, request
from flask_restful import Resource, Api, reqparse

app = Flask(__name__)
api = Api(app)

parser = reqparse.RequestParser()
parser.add_argument('task', type=str)  # 相当于添加 form 表单字段，并给出类型

todos = {}

class TodoSimple(Resource):
    def get(self, todo_id):
        return {todo_id: todos[todo_id]}

    def put(self, todo_id):
        args = parser.parse_args()   # 获取表单的内容, args 是一个字典
        if args['task']:
            todos[todo_id] = args['task']   
        else:
            return 'error!'

        return {todo_id: todos[todo_id]}

api.add_resource(TodoSimple, '/<string:todo_id>')

if __name__ == '__main__':
    app.run(debug=True)
```

上面的代码中，我们使用 `add_argument()` 方法添加 form 表单字段，并指定其类型。获取表单内容使用 `parse_args()` 方法，该方法返回一个字典，字典的 key 就是表单的字段。

使用 curl 测试，如下：

```python
$ curl -X PUT http://localhost:5000/todo1 -d "task=shopping"
{
    "todo1": "shopping"
}
```

## 一个完整的例子

下面这个完整的例子来自于[官方文档](http://flask-restful-cn.readthedocs.io/en/0.3.5/quickstart.html)。

```python
from flask import Flask
from flask_restful import reqparse, abort, Api, Resource

app = Flask(__name__)
api = Api(app)

TODOS = {
    'todo1': {'task': 'build an API'},
    'todo2': {'task': '?????'},
    'todo3': {'task': 'profit!'},
}

def abort_if_todo_doesnt_exist(todo_id):
    if todo_id not in TODOS:
        abort(404, message="Todo {} doesn't exist".format(todo_id))

parser = reqparse.RequestParser()
parser.add_argument('task')

# Todo
# shows a single todo item and lets you delete a todo item
class Todo(Resource):
    def get(self, todo_id):
        abort_if_todo_doesnt_exist(todo_id)
        return TODOS[todo_id]

    def delete(self, todo_id):
        abort_if_todo_doesnt_exist(todo_id)
        del TODOS[todo_id]
        return '', 204

    def put(self, todo_id):
        args = parser.parse_args()
        task = {'task': args['task']}
        TODOS[todo_id] = task
        return task, 201

# TodoList
# shows a list of all todos, and lets you POST to add new tasks
class TodoList(Resource):
    def get(self):
        return TODOS

    def post(self):
        args = parser.parse_args()
        todo_id = int(max(TODOS.keys()).lstrip('todo')) + 1
        todo_id = 'todo%i' % todo_id
        TODOS[todo_id] = {'task': args['task']}
        return TODOS[todo_id], 201

## Actually setup the Api resource routing here
api.add_resource(TodoList, '/todos')
api.add_resource(Todo, '/todos/<todo_id>')

if __name__ == '__main__':
    app.run(debug=True)
``` 

运行上面的代码，然后使用 curl 进行测试，如下：

```
# 获取所有 todos
$ curl http://localhost:5000/todos
{
    "todo1": {
        "task": "build an API"
    },
    "todo2": {
        "task": "?????"
    },
    "todo3": {
        "task": "profit!"
    }
}

# 获取单个 todo
$ curl http://localhost:5000/todos/todo2
{
    "task": "?????"
}

# 删除某个 todo
$ curl -v -X DELETE http://localhost:5000/todos/todo2
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 5000 (#0)
> DELETE /todos/todo2 HTTP/1.1
> Host: localhost:5000
> User-Agent: curl/7.43.0
> Accept: */*
>
* HTTP 1.0, assume close after body
< HTTP/1.0 204 NO CONTENT
< Content-Type: application/json
< Content-Length: 0
< Server: Werkzeug/0.11.11 Python/2.7.11
< Date: Tue, 18 Oct 2016 04:02:17 GMT
<
* Closing connection 0

# 添加 todo
$ curl -v -X POST http://localhost:5000/todos -d "task=eating"
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 5000 (#0)
> POST /todos HTTP/1.1
> Host: localhost:5000
> User-Agent: curl/7.43.0
> Accept: */*
> Content-Length: 11
> Content-Type: application/x-www-form-urlencoded
>
* upload completely sent off: 11 out of 11 bytes
* HTTP 1.0, assume close after body
< HTTP/1.0 201 CREATED
< Content-Type: application/json
< Content-Length: 25
< Server: Werkzeug/0.11.11 Python/2.7.11
< Date: Tue, 18 Oct 2016 04:04:16 GMT
<
{
    "task": "eating"
}
* Closing connection 0

# 更新 todo
$ curl -v -X PUT http://localhost:5000/todos/todo3 -d "task=running"
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 5000 (#0)
> PUT /todos/todo3 HTTP/1.1
> Host: localhost:5000
> User-Agent: curl/7.43.0
> Accept: */*
> Content-Length: 12
> Content-Type: application/x-www-form-urlencoded
>
* upload completely sent off: 12 out of 12 bytes
* HTTP 1.0, assume close after body
< HTTP/1.0 201 CREATED
< Content-Type: application/json
< Content-Length: 26
< Server: Werkzeug/0.11.11 Python/2.7.11
< Date: Tue, 18 Oct 2016 04:05:52 GMT
<
{
    "task": "running"
}
* Closing connection 0
```

本文完整的代码在[这里](https://github.com/ethan-funny/flask-demos/blob/ext/flask-restful-demo/app.py)。

# 更多阅读

- [Flask-RESTful 0.2.1 documentation](http://flask-restful-cn.readthedocs.io/en/0.3.5/quickstart.html)
- [Designing a RESTful API using Flask-RESTful - miguelgrinberg.com](https://blog.miguelgrinberg.com/post/designing-a-restful-api-using-flask-restful)
- [Flask扩展系列(一)–Restful – 思诚之道](http://www.bjhee.com/flask-ext1.html)


[1]: https://funhacks.gitbooks.io/head-first-flask/content/chapter02/section2.09.html
[2]: http://flask-restful.readthedocs.io/en/0.3.5/


