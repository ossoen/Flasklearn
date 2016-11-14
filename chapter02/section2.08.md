# 工厂方法

在前面，我们都是直接通过`app=Flask(__name__)`来创建一个`app`实例的。这样做没什么问题，但如果我们想为每个实例分配不同的配置，比如有测试环境的配置，开发环境的配置和生产环境的配置等，这时就比较麻烦了。

有什么办法呢？

其实我们可以通过调用一个函数来返回一个应用实例，比如下面的方法：

```python
def create_app(config_filename):
    app = Flask(__name__)
    app.config.from_pyfile(config_filename)

    from yourapplication.views.admin import admin
    from yourapplication.views.user import user
    app.register_blueprint(admin)
    app.register_blueprint(user)

    return app
```

上面的`create_app`函数就是一个`工厂方法`，我们将创建应用程序实例的工作交给了它来完成，我们以后就可以通过传入不同的配置名，以此批量生产`app`。

说到这，你也应该明白`工厂方法`的优势所在了：

- 将创建应用实例的过程交给工厂函数，通过传入不同的配置，我们可以创建不同环境下的应用
- 在做测试时，为每个实例分配分配不同的配置，从而测试每一种不同的情况

现在，我们对[上一篇](https://funhacks.gitbooks.io/head-first-flask/content/chapter02/section2.06.html)的例子进行重构，引入工厂函数。

核心代码在下面，完成代码请参考[这里](https://github.com/ethan-funny/flask-demos/tree/v0.5)。

现在，`app.py`代码如下：

```python
# -*- coding: utf-8 -*-

from flask import Flask, render_template
from configs import config
from book import book_bp
from movie import movie_bp

def create_app(config_name):
    app = Flask(__name__)

    # basic config
    app.config.from_object(config[config_name])

    # blueprints
    app.register_blueprint(book_bp)
    app.register_blueprint(movie_bp)

    # error handler
    handle_errors(app)

    return app

def handle_errors(app):
    @app.errorhandler(404)
    def page_not_found(error):
        return render_template('404.html'), 404
```

我们还需要一个启动程序，比如`run.py`，如下：

```python
# -*- coding: utf-8 -*-

from app import create_app

if __name__ == '__main__':
    app = create_app('default')
    app.run(host='127.0.0.1', port=5200, debug=True)
```

打开终端，运行下面命令，就可以在浏览器访问我们的应用了。

```
$ python run.py
```

# 更多阅读

- [应用程序的工厂函数 — Flask 0.10.1 文档](http://docs.jinkan.org/docs/flask/patterns/appfactories.html)
- [Flask进阶系列(七)–应用最佳实践 – 思诚之道](http://www.bjhee.com/flask-ad7.html)

