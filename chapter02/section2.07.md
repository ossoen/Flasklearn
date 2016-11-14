# 消息闪现

Flask 提供了消息闪现的功能，以使我们的应用可以向用户反馈信息。比如，当用户登录失败了，我们会提醒用户名错误或者密码错误。

在 Flask 中使用消息闪现很简单。下面我们以[上一篇](https://funhacks.gitbooks.io/head-first-flask/content/chapter02/section2.06.html)的例子进行说明。完整的代码在[这里](https://github.com/ethan-funny/flask-demos/tree/v0.3)。

首先，让我们看一下`book.py`的代码：

```python
# -*- coding: utf-8 -*-

from flask import Blueprint, url_for, render_template, request, flash, redirect

book_bp = Blueprint(
    'book', 
    __name__,
    template_folder='../templates',
)

books = ['The Name of the Rose', 'The Historian', 'Rebecca']


@book_bp.route('/', methods=['GET'])
def index():
    return '<h1>Hello World!</h1>'


@book_bp.route('/book', methods=['GET', 'POST'])
def handle_book():
    _form = request.form

    if request.method == 'POST':
        title = _form["title"]
        books.append(title)
        flash("add book successfully!")   # 使用 flash 反馈消息
        return redirect(url_for('book.handle_book'))

    return render_template(
        'book.html',
        books=books
   )
```

注意到，用户添加完书籍的时候，我们向用户反馈『添加书籍成功』的消息：

```
flash("add book successfully!")
```

但是，还有一个步骤要做，才能让用户接收到这个消息，那就是在模板中获取这个消息，通过在`layout.html`中使用`get_flashed_messages()`获取消息，`layout.html`代码如下：

```html
<!doctype html>
<title>Hello Sample</title>
<link rel="stylesheet" type="text/css" href="{{ url_for('static', filename='style.css') }}">

<div class="page">
    {% block body %} {% endblock %}
</div>

{% for message in get_flashed_messages() %}
    {{ message }}
{% endfor %}
```

## 分类闪现消息

我们还可以对闪现消息进行分类，比如有些消息是正常的通知消息，而有些消息是出错消息。同样，我们需要两个步骤：

- 使用 flash() 函数的第二个参数，不使用的话，默认是 'message'

```
flash('add book fail', 'error')
```

- 接着，在模板中调用`get_flashed_messages()`函数来返回这个分类，类似下面：

```
{% with messages = get_flashed_messages(with_categories=true) %}
  {% for category, message in messages %}
    {{category}}: {{ message }}
  {% endfor %}
{% endwith %}
```

## 过滤闪现消息

过滤闪现消息在模板中可以这样用：

```
{% with error_messages = get_flashed_messages(category_filter=["error"]) %}
  {% for error in error_messages %}
    {{ error }}
  {% endfor %}
{% endwith %}
```

# 更多阅读

- [消息闪现 — Flask 0.10.1 文档](http://docs.jinkan.org/docs/flask/patterns/flashing.html#message-flashing-pattern)
- [浅入浅出Flask框架：flashing system | 樂天笔记](http://www.letiantian.me/2014-06-29-flask-flash/)
- [Flask入门系列(五)–错误处理及消息闪现 – 思诚之道](http://www.bjhee.com/flask-5.html)


