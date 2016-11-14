# Flask-Bootstrap

[Bootstrap](http://getbootstrap.com/) 是 Twitter 开源的一个 CSS/HTML 框架，它让 Web 开发变得更加迅速，简单。要想在我们的 Flask 应用中使用 Boostrap，有两种方案可供选择：

- 第 1 种，在我们的 Jinja 模板中直接引入 Bootstrap 层叠样式表 (CSS) 和 JavaScript 文件，比如 [bootstrap.min.css](https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css)，[bootstrap.min.js](https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js)；
- 第 2 种，也是更简单的方法，就是使用一个 [Flask-Bootstrap](https://pythonhosted.org/Flask-Bootstrap/) 的扩展，它简化了集成 Bootstrap 的过程；

# 安装

使用 `pip` 安装：

```python
$ pip install flask-bootstrap
```

# 使用

现在，让我们写一个简单的 `hello world` 例子。

首先，新建一个 `app.py` 文件，代码如下：

```python
# -*- coding: utf-8 -*-

from flask import Flask, render_template
from flask_bootstrap import Bootstrap

app = Flask(__name__)
Bootstrap(app)  # 把程序实例即 app 传入构造方法进行初始化

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/book')
def book():
    books = ['the first book', 'the second book']
    return render_template('index.html', books=books)

@app.route('/movie')
def movie():
    movies = ['the first movie', 'the second movie']
    return render_template('index.html', movies=movies)

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5200, debug=True)
```

初始化 Flask-Bootstrap 之后，我们的模板通过继承一个基模板，即 `bootstrap/base.html`，就可以使用 Bootstrap 了。

新建一个 `templates` 文件夹，在该文件夹中添加一个 `index.html` 的文件，代码如下：

```
{% extends "bootstrap/base.html" %}

{% block title %}Demo{% endblock %}

{% block navbar %}
<div class="navbar navbar-inverse">
    <div class="container">
        <div class="navbar-header">
            <a class="navbar-brand" href="{{ url_for('index') }}">首页</a>
        </div>
        <div class="navbar-collapse collapse">
            <ul class="nav navbar-nav">
                <li><a href="{{ url_for('book') }}">读书</a></li>
                <li><a href="{{ url_for('movie') }}">电影</a></li>
            </ul>
        </div>
    </div>
</div>
{% endblock %}

{% block content %}
<div class="container">
    {% if books %}
        <ul>
            {% for book in books %}
                <li>{{ book }}</li>
            {% endfor %}
        </ul>
    {% elif movies %}
        <ul>
            {% for movie in movies %}
                <li>{{ movie }}</li>
            {% endfor %}
        </ul>
    {% else %}
        <p><a href="{{ url_for('book') }}"> 读书 </a></p>
        <p><a href="{{ url_for('movie') }}"> 电影 </a></p>
    {% endif %}
</div>
{% endblock %}
```

注意到上面模板中的第 1 行：

```
{% extends "bootstrap/base.html" %}
```

这是利用了 Jinja2 的模板继承机制，我们在前面也已经讲过了，如果忘了，可[点此查看](https://funhacks.gitbooks.io/head-first-flask/content/chapter02/section2.04.html)。`extends` 指令从 [Flask-Bootstrap](https://pythonhosted.org/Flask-Bootstrap/) 中导入 `bootstrap/base.html`，该基模板引入了 Bootstrap 中的所有 CSS 和 JavaScript 文件。通过这一句，我们就可以在模板中使用 Bootstrap 了，是不是很简单？

注意到，我们在模板中也定义了诸如 `{% block title %} Demo {% endblock %}` 的块，这些块是基模板提供的，可在我们的衍生模板中重新定义。其中，`title` 块中的内容会被放到渲染后的 `<title>` 标签中，`navbar` 块表示页面中的导航条，`content` 块表示页面的主体内容。

基模板除了提供上面的块，还定义了很多其他块，下面的表格列出了所有可用的块：


| 块名 | 说明 |
| :-: | :-: |
|  doc | 整个 HTML 文档 |
| html | `<html>` 标签中的内容 |
| html_attribs | `<html>` 标签的属性 |
| head | `<head>` 标签中的内容 |
| body | `<body>` 标签中的内容 |
| body_attribs | `<body>` 标签的属性 |
| title | `<title>` 标签中的内容 |
| styles | CSS 定义 |
| metas | 一组 `<meta>` 标签 |
| navbar | 导航条 |
| content | 页面内容 |
| scripts | 文档底部的 JavaScript 声明 |

上面的块，比如 `title`, `navbar` 和 `content` 是可以直接重定义的，但是，如果程序需要向已经有内容的块添加新内容，必须使用 Jinja2 提供的 `super()` 函数，比如 `styles` 块和 `scripts` 块：

- 如果我们想添加自己的样式表，需要这样：

```
{% block styles %}
{{ super() }}
<link rel="stylesheet" href="{{url_for('.static', filename='mystyle.css')}}">
{% endblock %}
```

- 如果我们想添加自己的 JavaScript 文件，需要这样：

```
{% block scripts %}
{{ super() }}
<script src="{{url_for('.static', filename='myscripts.js')}}"></script>
{% endblock %}
```

本文完整的代码可在[这里](https://github.com/ethan-funny/flask-demos/tree/ext/flask-bootstrap-demo)下载。

# 更多阅读

- [Flask-Bootstrap](https://pythonhosted.org/Flask-Bootstrap/basic-usage.html)


