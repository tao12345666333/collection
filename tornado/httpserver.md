译者说
-----
`Tornado 4.3`于2015年11月6日发布，该版本正式支持`Python3.5`的`async`/`await`关键字，并且用旧版本CPython编译Tornado同样可以使用这两个关键字，这无疑是一种进步。其次，这是最后一个支持`Python2.6`和`Python3.2`的版本了，在后续的版本了会移除对它们的兼容。现在网络上还没有`Tornado4.3`的中文文档，所以为了让更多的朋友能接触并学习到它，我开始了这个翻译项目，希望感兴趣的小伙伴可以一起参与翻译，项目地址是[tornado-zh on Github](https://github.com/tao12345666333/tornado-zh)，翻译好的文档在[Read the Docs上](https://tornado-zh.readthedocs.org/)直接可以看到。欢迎Issues or PR。本节感谢[@ladrift](https://github.com/ladrift)翻译

####PS:本节最好直接在[https://tornado-zh.readthedocs.org](https://tornado-zh.readthedocs.org/zh/latest/)或者[http://tornado.moelove.info/](http://tornado.moelove.info/)阅读，以获得更好的阅读体验(格式支持)。原谅我没排好版QAQ

# tornado.httpserver — 非阻塞 HTTP server
非阻塞，单线程 HTTP server。

典型的应用很少与 HTTPServer 类直接交互，除非在进程开始时开启server （尽管这经常间接的通过 tornado.web.Application.listen 来完成）。

在 4.0 版更改: 曾经在此模块中的 HTTPRequest 类 已经被移到 tornado.httputil.HTTPServerRequest 。 其旧名称仍作为一个别名。

##HTTP Server
### class tornado.httpserver.HTTPServer(*args, **kwargs)
非阻塞，单线程 HTTP server。

一个server可以由一个 HTTPServerConnectionDelegate 的子类定义， 或者，为了向后兼容，由一个以 HTTPServerRequest 为参数的callback定义。 它的委托对象(delegate)通常是 tornado.web.Application 。

HTTPServer 默认支持keep-alive链接（对于HTTP/1.1自动开启，而对于HTTP/1.0， 需要client发起 Connection: keep-alive 请求）。

如果 xheaders 是 True ，我们支持 X-Real-Ip/X-Forwarded-For 和 X-Scheme/X-Forwarded-Proto 首部字段，他们将会覆盖 所有请求的 remote IP 与 URI scheme/protocol 。 当Tornado运行在反向代理或者负载均衡(load balancer)之后时， 这些首部字段非常有用。如果Tornado运行在一个不设置任何一个支持的 xheaders 的SSL-decoding代理之后， protocol 参数也能设置为 https 。

要使server可以服务于SSL加密的流量，需要把 ssl_option 参数 设置为一个 ssl.SSLContext 对象。为了兼容旧版本的Python ssl_options 可能也是一个字典(dictionary)，其中包含传给 ssl.wrap_socket 方法的关键 字参数。:

```python
ssl_ctx = ssl.create_default_context(ssl.Purpose.CLIENT_AUTH)
ssl_ctx.load_cert_chain(os.path.join(data_dir, "mydomain.crt"),
                        os.path.join(data_dir, "mydomain.key"))
HTTPServer(applicaton, ssl_options=ssl_ctx)
```
HTTPServer 的初始化依照以下三种模式之一（初始化方法定义 在 tornado.tcpserver.TCPServer ）：

### listen: 简单的单进程:

```python
server = HTTPServer(app)
server.listen(8888)
IOLoop.current().start()
```
在很多情形下， tornado.web.Application.listen 可以用来避免显式的 创建 HTTPServer 。

### bind/start: 简单的多进程:

```python
server = HTTPServer(app)
server.bind(8888)
server.start(0)  # Fork 多个子进程
IOLoop.current().start()
```
当使用这个接口时，一个 IOLoop 不能被传给 HTTPServer 的构造方法(constructor)。 start 将默认 在单例 IOLoop 上开启server。

### add_sockets: 高级多进程:

```python
sockets = tornado.netutil.bind_sockets(8888)
tornado.process.fork_processes(0)
server = HTTPServer(app)
server.add_sockets(sockets)
IOLoop.current().start()
```

add_sockets 接口更加复杂， 但是，当fork发生的时候，它可以与 tornado.process.fork_processes 一起使用来提供更好的灵活性。 如果你想使用其他的方法，而不是 tornado.netutil.bind_sockets ， 来创建监听socket， add_sockets 也可以被用在单进程server中。

在 4.0 版更改: 增加了 decompress_request, chunk_size, max_header_size, idle_connection_timeout, body_timeout, max_body_size 参数。支持 HTTPServerConnectionDelegate 实例化为 request_callback 。

在 4.1 版更改: HTTPServerConnectionDelegate.start_request 现在需要传入两个参数来调用 (server_conn, request_conn) （根据文档内容）而不是一个 (request_conn).

在 4.2 版更改: HTTPServer 现在是 tornado.util.Configurable 的一个子类。
