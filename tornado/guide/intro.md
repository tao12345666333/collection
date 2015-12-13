译者说
-----
`Tornado 4.3`于2015年11月6日发布，该版本正式支持`Python3.5`的`async`/`await`关键字，并且用旧版本CPython编译Tornado同样可以使用这两个关键字，这无疑是一种进步。其次，这是最后一个支持`Python2.6`和`Python3.2`的版本了，在后续的版本了会移除对它们的兼容。现在网络上还没有`Tornado4.3`的中文文档，所以为了让更多的朋友能接触并学习到它，我开始了这个翻译项目，希望感兴趣的小伙伴可以一起参与翻译，项目地址是[tornado-zh on Github](https://github.com/tao12345666333/tornado-zh)，翻译好的文档在[Read the Docs上](https://tornado-zh.readthedocs.org/)直接可以看到。欢迎Issues or PR。

介绍
------------

[Tornado](http://www.tornadoweb.org) 是一个Python web框架和异步网络库起初由 [FriendFeed](http://friendfeed.com)开发. 通过使用非阻塞网络I/O, Tornado 可以支持上万级的连接，处理[长连接](http://en.wikipedia.org/wiki/Push_technology#Long_polling),[WebSockets](http://en.wikipedia.org/wiki/WebSocket), 和其他需要与每个用户保持长久连接的应用.

Tornado 大体上可以被分为4个主要的部分:

* web框架 (包括创建web应用的 `RequestHandler` 类，还有很多其他支持的类).
* HTTP的客户端和服务端实现 (`HTTPServer` and `AsyncHTTPClient`).
* 异步网络库 (`IOLoop` and `IOStream`),
  为HTTP组件提供构建模块，也可以用来实现其他协议.
* 协程库 (`tornado.gen`) 允许异步代码写的更直接而不用链式回调的方式.

Tornado web 框架和HTTP server 一起为[WSGI](http://www.python.org/dev/peps/pep-3333/)提供了一个全栈式的选择.
在WSGI容器 (`.WSGIAdapter`) 中使用Tornado web框架或者使用Tornado HTTP server作为一个其他WSGI框架(`.WSGIContainer`)的容器,这样的组合方式都是有局限性的.为了充分利用Tornado的特性,你需要一起使用Tornado的web框架和HTTP server.
