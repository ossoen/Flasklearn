
<img src='https://ooo.0o0.ooo/2016/11/03/581b594a5150f.png' width='300'>

Flask Web 开发入门
===

## Flask 简介

Python 中有许多 [Web 开发框架](https://wiki.python.org/moin/WebFrameworks)，比如 [Django][1]，[Flask][2]，[Tornado][3]，[Bottle][4] 和 [web.py][5] 等，其中，Django 可以说是一个全能型（all in one）的框架，自带管理后台；而 Flask 则是一个非常轻量级的框架，提供了搭建 Web 服务的必要组件，如果你不喜欢自带的组件，由于 Flask 良好的扩展性，你也可以使用其他开源的 Flask 扩展插件，甚至可以自己写一个，让喜欢折腾的开发者一展身手；Tornado 则主打异步处理，高并发，这也是它的一个显著特点。

第一次接触到 Flask 时被它的简洁感动了，几行代码就可以快速搭建出一个简单的 Web 服务，于是就义无反顾地踏上了 Flask 的学习之路，慢慢地就学习到了诸如 Jinja2 模板引擎，路由，视图，静态文件和蓝图等。Flask 非常小，源码文件包括注释在内，总共才 6000 多行，当你能熟练使用 Flask 的各个模块时，相信你也可以读懂它的所有源码。

## 关于本书

本书的写作开始于 2016 年 7 月，当时的初衷就是想把学的东西记录下来，但是比较分散，后来想到可以把它写成一本开源的电子书，何乐而不为？可是真正写的时候，才发现写书真的好费精力。但不管怎样，最后还是写了一些东西。9 月份发布了第 1 版，收到不少网友的良好建议，所以又抽空进行了完善，当然，也拖了不少时间。

本书主要介绍 Flask 的基本使用，这也是我一开始在学习 Flask 过程中经常用到的。我也希望读者能通过本书快速掌握 Flask 的基本功能，快速构建出自己的 Web 服务。阅读本书可能需要读者掌握基本的 Python 语法知识，以及简单的 HTML 语法。

本书主要分为五个章节：

- 第 1 章：介绍 Flask 的安装和快速使用。
- 第 2 章：介绍 Flask 的基本使用方法，比如路由和视图，静态模板，蓝图和工厂方法等。
- 第 3 章：介绍 Flask 常用扩展插件的使用方法。
- 第 4 章：Flask 实战，介绍了如何开发一个 Web TODO 应用。
- 第 5 章：结束语，包含一些相关的参考资料以及资源推荐。

## 声明

<a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a>

本书由 [ethan-funny][6] 编写，采用 [CC BY-NC-ND 4.0][7] 协议发布。

这意味着你可以在非商业性使用的前提下自由转载，但必须：

1. 保持署名
2. 不对本书进行修改

## 更新记录

| 时间 | 说明 |
| --- | --- |
| 2016-11-14 | 发布版本 v1.1，增加了蓝图、工厂方法、消息闪现和 Flask 常用扩展等 |
| 2016-09-10 | 发布版本 v1.0，包含基本的路由和视图，模板引擎，部署等 |
| 2016-08-22 | 基本完成初稿 |

## 联系我

如果你对于本书有什么建议或意见，欢迎批评指正，并联系我。

- [个人主页][8]
- [GitHub][9]
- [Twitter][10]

## 支持我

如果你觉得本书对你有所帮助，不妨请我喝杯咖啡，感谢支持！

<img src='http://o9txbs6d7.bkt.clouddn.com/WechatPay.png' width='300'>

<img src='http://o9txbs6d7.bkt.clouddn.com/Ali_Pay.png' width='300'>


[1]:	https://www.djangoproject.com/
[2]:	http://flask.pocoo.org/
[3]:	https://github.com/tornadoweb/tornado
[4]:	https://github.com/bottlepy/bottle
[5]:	http://webpy.org/
[6]:	https://github.com/ethan-funny
[7]:	http://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh
[8]:	https://funhacks.net
[9]:	https://github.com/ethan-funny
[10]:	https://twitter.com/pihacks


