# Flask-Cache

假设你的 Web 服务对于某些请求比较耗时，而该请求的返回结果在较短的时间内（比如 5 分钟内）都是足够有效的，这时你能想到什么方法去改善这种状况呢？缓存？对，至少这是一种提高性能的最简单的方法。

Flask 本身不提供缓存功能，但是作为 Flask 核心的 [Werkzeug](http://werkzeug.pocoo.org/) 框架则提供了一个简单的缓存对象 [SimpleCache](http://werkzeug.pocoo.org/docs/0.11/contrib/cache/#werkzeug.contrib.cache.SimpleCache)，它将缓存项存放在 Python 解释器的内存中。使用 `SimpleCache` 需要自己实现缓存装饰器，稍微有点麻烦。幸运的是，[Flask-Cache](https://pythonhosted.org/Flask-Cache/) 扩展使我们在 Flask 中使用缓存变得很简单。

# 安装

使用 pip 安装：

```
$ pip install Flask-Cache
```

# 使用

## 缓存视图函数

同使用其他扩展一样，我们要先创建扩展的实例：

```python
from flask import Flask
from flask_cache import Cache

app = Flask(__name__)
cache = Cache(app, config={'CACHE_TYPE': 'simple'})
```

同样，我们也可以初始化 `Cache` 后再使用 `init_app` 方法：

```python
from flask import Flask
from flask_cache import Cache

cache = Cache(config={'CACHE_TYPE': 'simple'})

app = Flask(__name__)
cache.init_app(app)
```

上面的代码中，我们设置 `CACHE_TYPE` 即缓存类型为 'simple'，其内部实现就是 Werkzeug 中的SimpleCache。

另外，我们也可以使用第三方的缓存服务器，比如 Redis，可以这样设置：

```
cache = Cache(
    app, 
    config={
        'CACHE_TYPE': 'redis',          
        'CACHE_REDIS_HOST': '127.0.0.1',
        'CACHE_REDIS_PORT': 6379,
        'CACHE_REDIS_PASSWORD': '123456',  
        'CACHE_REDIS_DB': 0
    }
)
```

现在，让我们来看一个简单的例子，代码如下：

```python
# -*- coding: utf-8 -*-

from flask import Flask
from flask_cache import Cache

app = Flask(__name__)
cache = Cache(app, config={'CACHE_TYPE': 'simple'})

@app.route('/')
@cache.cached(timeout=300)
def index():
    print 'view hello called'
    return 'Hello World!'

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5205, debug=True)
```

上面的代码中，我们使用了 `cache` 对象的 `cached()` 方法来装饰视图函数，在 `cached()` 方法中，我们设置了 `timeout` 参数为 300 (单位为：秒)，这就意味着该视图函数在每 5 分钟最多运行一次，响应的结果会被保存在缓存中，并可以让期间的每一个请求获取。

那我们怎么验证上面的视图函数是每 5 分钟最多运行一次呢？其实很简单，我们把上面的代码跑起来，浏览器访问 `http://127.0.0.1:5205/`，这时出现 Hello World!，在终端可以看到 view hello called，刷新页面，也可以看到 Hello World!，但是终端已经看不到 view hello called 了，过 5 分钟后，重复上面的步骤，这时在终端又可以看到 view hello called 了，这说明什么呢？缓存起效了！

在终端的打印日志如下：

```
$ python app.py
 * Running on http://127.0.0.1:5205/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger pin code: 109-566-036
view hello called
127.0.0.1 - - [20/Oct/2016 22:50:28] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [20/Oct/2016 22:50:29] "GET /favicon.ico HTTP/1.1" 404 -
127.0.0.1 - - [20/Oct/2016 22:50:42] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [20/Oct/2016 22:52:16] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [20/Oct/2016 22:52:59] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [20/Oct/2016 22:53:14] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [20/Oct/2016 22:55:27] "GET / HTTP/1.1" 200 -
view hello called
127.0.0.1 - - [20/Oct/2016 22:55:37] "GET / HTTP/1.1" 200 -
```

## 缓存普通函数

上面，我们对视图函数进行了缓存。事实上，我们也可以给非视图函数加缓存，比如：

```python
@app.route('/posts')
def get_posts():
    return ', '. join(get_posts())

# 对非视图函数进行缓存
@cache.cached(timeout=60, key_prefix='get_posts')
def get_posts():
    print 'method get_posts called'
    return ['a', 'b', 'c', 'd']
```

给非视图函数加缓存，跟缓存视图函数类似，但有一点需要注意的是，此时 `cached()` 方法需要加上 `key_prefix` 参数，由于视图函数在默认情况下使用请求路径 (request.path) 作为 cache_key，所以可以省略。

运行上面的代码，在终端可以看到 method get_posts called，刷新页面，只有过了 60 秒后，在终端才能再次看到。


## 使用 Memoization（一种缓存技术）

上面的普通函数是没有带参数的，如果带参数，就不能像上面那样做了，为什么？不妨一试：

```python
@app.route('/comments/<int:num>')
def get_comments(num):
    return ', '.join(get_comments(num))

# 缓存带参数的函数
@cache.cached(timeout=60, key_prefix='get_comments')
def get_comments(num):
    print 'method get_comments called'
    return [str(i) for i in xrange(num)]
```

运行上面的代码，会发生什么情况呢？

我们发现：当在浏览器输入 `http://127.0.0.1:5205/comments/3` 时，返回了 '0, 1, 2'，同时在终端看到 'method get_comments called'，这是符合预期的。但是，如果此时我们把链接改成 `http://127.0.0.1:5205/comments/4`，这时依然返回 '0, 1, 2'，而我们想要的应该是 '0, 1, 2, 3'。仔细思考下，不能发现原因，因为我们对函数的结果进行了缓存，这时不管输入是什么，输出都是一样的，只有过了 60 秒后，结果才会改变。

那应该怎么改进呢？其实很简单，我们不用 `cache.cached`，而用 Flask-Cache 提供的另一个装饰器方法 `cache.memoize`，它与 `cache.cached` 的区别就是它会将函数的参数也放在缓存项的键值中。

我们将上面的代码改写如下：

```python
@app.route('/comments/<int:num>')
def get_comments(num):
    return ', '.join(get_comments(num))

# 缓存带参数的函数
@cache.memoize(timeout=60)
def get_comments(num):
    print 'method get_comments called'
    return [str(i) for i in xrange(num)]
```

运行上面的代码，在浏览器输入 `http://127.0.0.1:5205/comments/3`，返回 '0, 1, 2'，同时终端打印出 'method_get_comments called'，刷新页面，返回没有改变，终端也没有打印出上述字样，这时把链接改成 `http://127.0.0.1:5205/comments/4`，返回了 '0, 1, 2, 3'，同时终端也打印出了上述字样，符合我们的预期。

**小结**
> 如果函数不接受参数的话，cached() 和 memoize() 两者的作用是一样的。

## 删除缓存

- 使用 `delete()` 方法删除普通缓存

```
cache.delete('get_posts')
```

- 使用 `delete_many()` 方法删除多个缓存

```
cache.delete_many('get_posts', 'index') 
```

- 使用 `delete_memoized()` 方法删除 memoize 缓存

```
# 删除调用'get_comments'函数并且参数为 3 的缓存项
cache.delete_memoized('get_comments')
```

- 使用 `clear()` 方法删除所有缓存

```
cache.clear()
```

本文完整的代码在[这里](https://github.com/ethan-funny/flask-demos/blob/ext/flask-cache-demo/app.py)。

# 更多阅读

- [Flask-Cache](http://www.pythondoc.com/flask-cache/)
- [Flask扩展系列(六)–缓存 – 思诚之道](http://www.bjhee.com/flask-ext6.html)


