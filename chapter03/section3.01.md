# Flask-Mail

给用户发送邮件是 Web 应用中最常见的任务之一，比如用户注册，找回密码等。Python 内置了一个 [smtplib](http://www.runoob.com/python/python-email.html) 的模块，可以用来发送邮件，这里我们使用 [Flask-Mail](https://pythonhosted.org/Flask-Mail/)，是因为它可以和 Flask 集成，让我们更方便地实现此功能。

# 安装

使用`pip`安装：

```python
$ pip install Flask-Mail
```

或下载源码安装：

```python
$ git clone https://github.com/mattupstate/flask-mail.git
$ cd flask-mail
$ python setup.py install
```

# 发送邮件

Flask-Mail 连接到简单邮件传输协议 (Simple Mail Transfer Protocol, SMTP) 服务器，并把邮件交给这个服务器发送。这里以`QQ`邮箱为例，介绍如何简单地发送邮件。在此之前，我们需要知道`QQ`邮箱的服务器地址和端口是什么，[点此查看](https://kf.qq.com/faq/120322fu63YV130422nqIrqu.html)。

```python
# -*- coding: utf-8 -*-

from flask import Flask
from flask_mail import Mail, Message
import os

app = Flask(__name__)

app.config['MAIL_SERVER'] = 'smtp.qq.com'  # 邮件服务器地址
app.config['MAIL_PORT'] = 25               # 邮件服务器端口
app.config['MAIL_USE_TLS'] = True          # 启用 TLS
app.config['MAIL_USERNAME'] = os.environ.get('MAIL_USERNAME') or 'me@example.com'
app.config['MAIL_PASSWORD'] = os.environ.get('MAIL_PASSWORD') or '123456'

mail = Mail(app)

@app.route('/')
def index():
    msg = Message('Hi', sender='me@example.com', recipients=['he@example.com'])
    msg.html = '<b>Hello Web</b>'
    # msg.body = 'The first email!'
    mail.send(msg)
    return '<h1>OK!</h1>'

if __name__ == '__main__':
    app.run(host='127.0.0.1', debug=True)
```

在发送前，需要先设置用户名和密码，当然你也可以直接写在文件里，如果是从环境变量读取，可以这么做：

```
$ export MAIL_USERNAME='me@example.com'
$ export MAIL_PASSWORD='123456'
```

将上面的`sender`和`recipients`改一下，就可以进行测试了。

从上面的代码，我们可以知道，使用 Flask-Mail 发送邮件主要有以下几个步骤：

- 配置 app 对象的邮件服务器地址，端口，用户名和密码等
- 创建一个 Mail 的实例：`mail = Mail(app)`
- 创建一个 Message 消息实例，有三个参数：邮件标题、发送者和接收者
- 创建邮件内容，如果是 HTML 格式，则使用`msg.html`，如果是纯文本格式，则使用`msg.body`
- 最后调用`mail.send(msg)`发送消息

**Flask-Mail 配置项**

Flask-Mail 使用标准的 Flask 配置 API 进行配置，下面是一些常用的配置项：

| 配置项 | 说明 |
| --- | --- |
| MAIL_SERVER  | 邮件服务器地址，默认为 localhost |
| MAIL_PORT | 邮件服务器端口，默认为 25 |
| MAIL_USE_TLS | 是否启用传输层安全 (Transport Layer Security, TLS)协议，默认为 False |
| MAIL_USE_SSL | 是否启用安全套接层 (Secure Sockets Layer, SSL)协议，默认为 False |
| MAIL_DEBUG | 是否开启 DEBUG，默认为 app.debug |
| MAIL_USERNAME | 邮件服务器用户名，默认为 None |
| MAIL_PASSWORD | 邮件服务器密码，默认为 None |
| MAIL_DEFAULT_SENDER | 邮件发件人，默认为 None，也可在 Message 对象里指定 |
| MAIL_MAX_EMAILS | 邮件批量发送个数上限，默认为 None |
| MAIL_SUPPRESS_SEND | 默认为 app.testing，如果为 True，则不会真的发送邮件，供测试用 |

# 异步发送邮件

使用上面的方式发送邮件，会发现页面卡顿了几秒才出现消息，这是因为我们使用了同步的方式。为了避免发送邮件过程中出现的延迟，我们把发送邮件的任务移到后台线程中，代码如下：

```python
# -*- coding: utf-8 -*-

from flask import Flask
from flask_mail import Mail, Message
from threading import Thread
import os

app = Flask(__name__)

app.config['MAIL_SERVER'] = 'smtp.qq.com'
app.config['MAIL_PORT'] = 25
app.config['MAIL_USE_TLS'] = True
app.config['MAIL_USERNAME'] = os.environ.get('MAIL_USERNAME') or 'smtp.example.com'
app.config['MAIL_PASSWORD'] = os.environ.get('MAIL_PASSWORD') or '123456'

mail = Mail(app)

def send_async_email(app, msg):
    with app.app_context():
        mail.send(msg)

@app.route('/sync')
def send_email():
    msg = Message('Hi', sender='me@example.com', recipients=['he@example.com'])
    msg.html = '<b>send email asynchronously</b>'

    thr = Thread(target=send_async_email, args=[app, msg])
    thr.start()
    return 'send successfully'

if __name__ == '__main__':
    app.run(host='127.0.0.1', debug=True)
```

在上面，我们创建了一个线程，执行的任务是`send_async_email`，该任务的实现涉及一个问题[^1]：

> 很多 Flask 扩展都假设已经存在激活的程序上下文和请求上下文。Flask-Mail 中的`send()`函数使用 `current_app`，因此必须激活程序上下文。不过，在不同线程中执行`mail.send()`函数时，程序上下文要使用 `app.app_context()`人工创建。

# 带附件的邮件

有时候，我们发邮件的时候需要添加附件，比如文档和图片等，这也很简单，代码如下：

```python
# -*- coding: utf-8 -*-

from flask import Flask
from flask_mail import Mail, Message
import os

app = Flask(__name__)

app.config['MAIL_SERVER'] = 'smtp.qq.com'  # 邮件服务器地址
app.config['MAIL_PORT'] = 25               # 邮件服务器端口
app.config['MAIL_USE_TLS'] = True          # 启用 TLS
app.config['MAIL_USERNAME'] = os.environ.get('MAIL_USERNAME') or 'me@example.com'
app.config['MAIL_PASSWORD'] = os.environ.get('MAIL_PASSWORD') or '123456'

mail = Mail(app)

@app.route('/attach')
def add_attchments():
    msg = Message('Hi', sender='me@example.com', recipients=['other@example.com'])
    msg.html = '<b>Hello Web</b>'

    with app.open_resource("/Users/Admin/Documents/pixel-example.jpg") as fp:
        msg.attach("photo.jpg", "image/jpeg", fp.read())

    mail.send(msg)
    return '<h1>OK!</h1>'

if __name__ == '__main__':
    app.run(host='127.0.0.1', debug=True)
```

上面的代码中，我们通过`app.open_resource(path_of_attachment)`打开了本机的某张图片，然后通过`msg.attach()`方法将附件内容添加到 Message 对象。`msg.attach()`方法的第一个参数是附件的文件名，第二个参数是文件内容的`MIME (Multipurpose Internet Mail Extensions)`类型，第三个参数是文件内容。

如果你不知道附件的`MIME`类型是什么，可以查看 [MIME 参考手册](http://www.w3school.com.cn/media/media_mimeref.asp)。

# 批量发送

在某些情况下，我们需要批量发送邮件，比如给网站的所有注册用户发送改密码的邮件，这时为了避免每次发邮件时都要创建和关闭跟服务器的连接，我们的代码需要做一些调整，类似如下：

```python
with mail.connect() as conn:
    for user in users:
        subject = "hello, %s" % user.name
        msg = Message(recipients=[user.email], body='...', subject=subject)
        conn.send(msg)
```

上面的工作方式，使得应用与电子邮件服务器保持连接，一直到所有邮件已经发送完毕。某些邮件服务器会限制一次连接中的发送邮件的上限，这样的话，你可以配置`MAIL_MAX_EMAILS`。

需要注意的是，更好的发送大量电子邮件的方式是用专门的作业系统，比如用 [Celery](http://www.celeryproject.org/) 任务队列等。

本文完整的代码在[这里](https://github.com/ethan-funny/flask-demos/blob/ext/flask-mail-demo/app.py)。

# 更多阅读

- [flask-mail — Flask-Mail 0.9.1 documentation](http://www.pythondoc.com/flask-mail/#api)
- [Flask扩展系列(二)–Mail – 思诚之道](http://www.bjhee.com/flask-ext2.html)

[^1]: [Flask Web Development](http://shop.oreilly.com/product/0636920031116.do)


