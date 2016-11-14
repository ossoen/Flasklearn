# 蓝图

在前面，我们都是把代码写在单一的文件里面，虽然看起来很方便，但也只是供学习的时候用用而已，真正在一个实际项目中，是不应该这样做的，为什么呢？

我们还是从 `hello world` 开始讲起，新建一个脚本文件，比如 `hello.py`。

```python
$ cat hello.py

# -*- coding: utf-8 -*-

from flask import Flask

app = Flask(__name__)

@app.route("/")
def index():
    return "Hello World!"

if __name__ == "__main__":
    app.run(host='127.0.0.1', port=5200, debug=True)
```

运行该脚本，在浏览器输入链接 `http://localhost:5200/`，可以看到 `Hello World!` 的字样。

OK，现在我们需要添加一个『读书』频道，允许查看并添加书籍。这不很简单吗，将上面的代码修改一下，如下：

```py
$ cat hello.py

# -*- coding: utf-8 -*-

from flask import Flask, request, redirect, url_for

app = Flask(__name__)
app.secret_key = 'some secret key'

books = ['the first book', 'the second book', 'the third book']

@app.route("/")
def index():
    render_string = '<ul>'

    for book in books:
        render_string += '<li>' + book + '</li>'

    render_string += '</ul>'

    return render_string

@app.route("/book", methods=['POST', 'GET'])
def book():
    _form = request.form

    if request.method == 'POST':
        title = _form["title"]
        books.append(title)
        return redirect(url_for('index'))

    return '''
        <form name="book" action="/book" method="post">
            <input id="title" name="title" type="text" placeholder="add book">
            <button type="submit">Submit</button>
        </form>
        '''

if __name__ == "__main__":
    app.run(host='127.0.0.1', port=5200, debug=True)
```

OK，上面的脚本实现了最基本的查看并添加书籍的功能。
 
接着，我们还想添加『电影』、『音乐』、『美食』和『旅游』等频道，我们是继续往这个文件添加功能吗？当然不是，是时候把这些功能进行拆分了，将我们的应用模块化，把一个大应用分解成若干个小应用。你可能会想：把这些功能分别写到多个文件，比如 `movie.py`，`music.py` 等等，可是这样的话，我们的应用是跑不起来的。为什么呢？因为 `app` 对象只有一个。

那怎么办呢？

事实上，Flask 提供了 Blueprint (蓝图) 的功能，让我们可以实现模块化的应用。使用它主要有以下好处：

- 将一个复杂的大型应用分解成若干蓝图的集合，也就是若干个子应用或者说模块，每个蓝图都包含了可以作为独立模块的视图、模板和静态文件等；

- 制作通用的组件，使开发者更易复用组件；

不过目前 Flask 蓝图的注册是静态的，不支持可插拔。

现在，让我们将上面的代码进行改写，加入蓝图的功能。

首先，我们对程序的结构做一些调整，如下：

```
├── app.py            -- 启动程序
├── book              -- book 模块
│   ├── __init__.py
│   └── book.py
├── movie             -- movie 模块
│   ├── __init__.py
│   └── movie.py
└── templates         -- 模板
    ├── 404.html
    ├── book.html
    ├── layout.html
    └── movie.html
```

其中，movie 模块跟 book 模块是独立的，它们有各自的业务逻辑，互不影响。我们这里只分析 book 模块，book.py 的代码如下：

```python
# -*- coding: utf-8 -*-

from flask import Blueprint, url_for, render_template, request, flash, redirect

# 创建一个蓝图对象
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
        flash("add book successfully!")
        return redirect(url_for('book.handle_book'))

    return render_template(
        'book.html',
        books=books
   )

@book_bp.route('/book/<name>')
def get_book_info(name):
    book = [name]
    if name not in books:
        book = []

    return render_template(
        'book.html',
        books=book
    )
```

注意到，我们使用了下面的代码创建一个蓝图对象：

```
book_bp = Blueprint('book', __name__, template_folder='../templates')
```

Blueprint 要求至少传入两个参数，第一个参数是蓝图的名称，第二个参数是蓝图所在的包或模块，其他参数是可选的，比如`template_folder`，`url_prefix`和`static_folder`等。

在蓝图中使用路由是这样的：

```
# book_bp 就是我们创建的蓝图对象
@book_bp.route('/book/<name>')
```

创建好了蓝图之后，我们还需要注册它，否则不能使用。我们在`app.py`中注册：

```python
# -*- coding: utf-8 -*-

from flask import Flask, render_template
from book import book_bp
from movie import movie_bp

app = Flask(__name__)
app.secret_key = 'The quick brown fox jumps over the lazy dog'

# 注册蓝图
app.register_blueprint(book_bp)
app.register_blueprint(movie_bp)

@app.errorhandler(404)
def page_not_found(error):
    return render_template('404.html'), 404


if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5200, debug=True)
```

本文完整的代码在[这里](https://github.com/ethan-funny/flask-demos/tree/v0.3)。

在命令行中输入`$ python app.py`，我们就可以访问该应用了，如下：

![Imgur](http://i.imgur.com/AvItISQ.png)

![Imgur](http://i.imgur.com/Wvnn7sS.png)

## 总结

- 使用蓝图对我们的应用进行模块化
- 通过添加蓝图扩展我们的应用
- 蓝图定义完之后，还需要在应用中注册
- 可以给蓝图中的所有路由定义一个 URL 前缀 (url_prefix)

# 更多阅读

- [用蓝图实现模块化的应用](http://docs.jinkan.org/docs/flask/blueprints.html)
- [蓝图 | Flask之旅](https://spacewander.github.io/explore-flask-zh/7-blueprints.html)
- [Flask进阶系列(六)–蓝图 – 思诚之道](http://www.bjhee.com/flask-ad6.html)
- [如何理解 Flask 中的蓝图？ - 知乎专栏](https://zhuanlan.zhihu.com/p/21743996)


