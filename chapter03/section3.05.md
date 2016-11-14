# Flask-HTTPAuth

在 Web 应用中，我们经常需要保护我们的 api，以避免非法访问。比如，只允许登录成功的用户发表评论等。[Flask-HTTPAuth](https://github.com/miguelgrinberg/Flask-HTTPAuth) 扩展可以很好地对 HTTP 的请求进行认证，不依赖于 Cookie 和 Session。本文主要介绍两种认证的方式：基于密码和基于令牌 (token)。

# 安装

使用 pip 安装：

```python
$ pip install Flask-HTTPAuth
```

# 基于密码的认证

为了简化代码，这里我们就不引入数据库了。

- 首先，创建扩展对象实例

```python
from flask import Flask
from flask_httpauth import HTTPBasicAuth
 
app = Flask(__name__)
auth = HTTPBasicAuth()
```

这里有一点需要注意的是，我们创建了一个 `auth` 对象，但没有传入 `app` 对象，这跟其他扩展初始化实例有一点区别。

- 接着，写一个验证用户密码的回调函数

```python
from werkzeug.security import generate_password_hash, check_password_hash

# 模拟数据库
books = ['The Name of the Rose', 'The Historian', 'Rebecca']
users = [
    {'username': 'ethan', 'password': generate_password_hash('6666')},
    {'username': 'peter', 'password': generate_password_hash('4567')}
]

# 回调函数
@auth.verify_password
def verify_password(username, password):
    user = filter(lambda user: user['username'] == username, users)

    if user and check_password_hash(user[0]['password'], password):
        g.user = username
        return True
    return False
```

上面，为了对密码进行加密以及认证，我们使用 `werkzeug.security` 包提供的 `generate_password_hash` 和 `check_password_hash` 方法：`generate_password_hash` 会对给定的字符串，生成其加盐的哈希值；`check_password_hash` 验证传入的明文字符串与哈希值是否一致。

- 然后，我们在需要认证的视图函数上，加上 `@auth.login_required` 装饰器，比如

```python
@app.route('/', methods=['POST'])
@auth.login_required
def add_book():
    _form = request.form
    title = _form["title"]
    if not title:
        return '<h1>invalid request</h1>'

    books.append(title)
    flash("add book successfully!")
    return redirect(url_for('index'))
```

上面完整的代码如下：

```python
$ cat app.py

# -*- coding: utf-8 -*-

from flask import Flask, url_for, render_template, request, flash, \
    redirect, make_response, jsonify, g
from werkzeug.security import generate_password_hash, check_password_hash
from flask_httpauth import HTTPBasicAuth


app = Flask(__name__)
app.config['SECRET_KEY'] = 'secret key'

auth = HTTPBasicAuth()

# 模拟数据库
books = ['The Name of the Rose', 'The Historian', 'Rebecca']
users = [
    {'username': 'ethan', 'password': generate_password_hash('6666')},
    {'username': 'peter', 'password': generate_password_hash('4567')}
]

# 回调函数
@auth.verify_password
def verify_password(username, password):
    user = filter(lambda user: user['username'] == username, users)

    if user and check_password_hash(user[0]['password'], password):
        g.user = username
        return True
    return False

# 不需认证，可直接访问
@app.route('/', methods=['GET'])
def index():
    return render_template(
        'book.html',
        books=books
    )

# 需要认证
@app.route('/', methods=['POST'])
@auth.login_required
def add_book():
    _form = request.form
    title = _form["title"]
    if not title:
        return '<h1>invalid request</h1>'

    books.append(title)
    flash("add book successfully!")
    return redirect(url_for('index'))

@auth.error_handler
def unauthorized():
    return make_response(jsonify({'error': 'Unauthorized access'}), 401)

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5206, debug=True)
```
   
```
$ cat templates/layout.html

<!doctype html>
<title>Hello Sample</title>

<div class="page">
    {% block body %} {% endblock %}
</div>

{% for message in get_flashed_messages() %}
    {{ message }}
{% endfor %}
```

```
$ cat templates/book.html

{% extends "layout.html" %}
{% block body %}
{% if books %}
    {% for book in books %}
        <ul>
            <li> {{ book }} </li>
        </ul>
    {% endfor %}
{% else %}
    <p> The book doesn't exists! </p>
{% endif %}

<form method="post" action="{{ url_for('add_book') }}">
    <input id="title" name="title" placeholder="add book" type="text">
    <button type="submit">Submit</button>
</form>

{% endblock %}
```

保存上面的代码，启动该应用，当我们试图提交书籍时，你会浏览器弹出了一个登录框，如下，只有输入正确的用户名和密码，书籍才会添加成功。

![auth](http://i.imgur.com/cVtqlOkm.png)

# 基于 token 的认证

很多时候，我们并不直接通过密码做认证，比如当我们把 api 开放给第三方的时候，我们不可能给它们提供密码，而是对它们进行授权，还可能会有时间限制，比如半年或一年等。这时候，我们往往通过一个令牌，也就是 token 来做认证，Flask-HTTPAuth 提供了 HTTPTokenAuth 对象来做这件事。

我们用[官方文档](http://flask-httpauth.readthedocs.io/en/latest/)的例子来说明。

```python
from flask import Flask, g
from flask_httpauth import HTTPTokenAuth

app = Flask(__name__)
auth = HTTPTokenAuth(scheme='Token')

tokens = {
    "secret-token-1": "john",
    "secret-token-2": "susan"
}

# 回调函数，验证 token 是否合法
@auth.verify_token
def verify_token(token):
    if token in tokens:
        g.current_user = tokens[token]
        return True
    return False

# 需要认证
@app.route('/')
@auth.login_required
def index():
    return "Hello, %s!" % g.current_user

if __name__ == '__main__':
    app.run()
```

上面，我们在初始化 HTTPTokenAuth 对象时，传入了 `scheme='Token'`。这个 scheme，是我们在发送请求时，在 HTTP 头 `Authorization` 中要用的 scheme 字段。用 curl 测试如下：

```
$ curl -X GET -H "Authorization: Token secret-token-1" http://localhost:5000/
```

结果：

```
Hello, john!
```

# 使用 itsdangerous 库来管理令牌

上面的令牌还是比较薄弱的，在实际使用中，我们需要使用加密的签名（Signature）作为令牌，它能够根据用户信息生成相关的签名，并且很难被篡改。[itsdangerous](https://github.com/pallets/itsdangerous) 提供了上述功能，在使用之前请使用 pip 安装: `$ pip install itsdangerous`。

改进后的代码如下：

```python
# -*- coding: utf-8 -*-

from flask import Flask, g
from flask_httpauth import HTTPTokenAuth
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer

app = Flask(__name__)
app.config['SECRET_KEY'] = 'secret key here'

auth = HTTPTokenAuth(scheme='Token')

# 实例化一个签名序列化对象 serializer，有效期 10 分钟
serializer = Serializer(app.config['SECRET_KEY'], expires_in=600)

users = ['john', 'susan']

# 生成 token
for user in users:
    token = serializer.dumps({'username': user})
    print('Token for {}: {}\n'.format(user, token))

# 回调函数，对 token 进行验证
@auth.verify_token
def verify_token(token):
    g.user = None
    try:
        data = serializer.loads(token)
    except:
        return False
    if 'username' in data:
        g.user = data['username']
        return True
    return False

# 对视图进行认证
@app.route('/')
@auth.login_required
def index():
    return "Hello, %s!" % g.user

if __name__ == '__main__':
    app.run()
```

将上面代码保存为 `app.py`，在终端运行，可看到类似如下的输出：

```
$ python app.py
Token for John: eyJhbGciOiJIUzI1NiIsImV4cCI6MTQ3NjY5NzE0NCwiaWF0IjoxNDc2Njk1MzQ0fQ.eyJ1c2VybmFtZSI6IkpvaG4ifQ.vQu0z0Pos2Tgt5jBYMY5IYWUkTK9k3wE_RqvYHDqtyM

Token for Susan: eyJhbGciOiJIUzI1NiIsImV4cCI6MTQ3NjY5NzE0NCwiaWF0IjoxNDc2Njk1MzQ0fQ.eyJ1c2VybmFtZSI6IlN1c2FuIn0.rk8JaTRwag0qiF9_KuRodhw6wx2ZWkOEhFln9hzOLP0

 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

使用 curl 测试如下：

```
$ curl -X GET -H "Authorization: Token eyJhbGciOiJIUzI1NiIsImV4cCI6MTQ3NjY5NzE0NCwiaWF0IjoxNDc2Njk1MzQ0fQ.eyJ1c2VybmFtZSI6IkpvaG4ifQ.vQu0z0Pos2Tgt5jBYMY5IYWUkTK9k3wE_RqvYHDqtyM" http://localhost:5000/

# 结果
$ Hello, john!
```

本节的代码可在[这里](https://github.com/ethan-funny/flask-demos/tree/ext/flask-httpauth-demo)下载。

# 更多阅读

- [使用 Flask 设计 RESTful 的认证 — Designing a RESTful API with Python and Flask 1.0 文档](http://flask-restfull.readthedocs.io/zh_CN/latest/third.html)
- [Flask扩展系列(九)–HTTP认证 – 思诚之道](http://www.bjhee.com/flask-ext9.html)
- [miguelgrinberg/REST-auth: Example application for my RESTful Authentication with Flask article.](https://github.com/miguelgrinberg/REST-auth)
- [Welcome to Flask-HTTPAuth’s documentation! — Flask-HTTPAuth documentation](http://flask-httpauth.readthedocs.io/en/latest/)

