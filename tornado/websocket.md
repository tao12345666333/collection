译者说
-----
`Tornado 4.3`于2015年11月6日发布，该版本正式支持`Python3.5`的`async`/`await`关键字，并且用旧版本CPython编译Tornado同样可以使用这两个关键字，这无疑是一种进步。其次，这是最后一个支持`Python2.6`和`Python3.2`的版本了，在后续的版本了会移除对它们的兼容。现在网络上还没有`Tornado4.3`的中文文档，所以为了让更多的朋友能接触并学习到它，我开始了这个翻译项目，希望感兴趣的小伙伴可以一起参与翻译，项目地址是[tornado-zh on Github](https://github.com/tao12345666333/tornado-zh)，翻译好的文档在[Read the Docs上](https://tornado-zh.readthedocs.org/)直接可以看到。欢迎Issues or PR。

####PS:本节最好直接在[https://tornado-zh.readthedocs.org](https://tornado-zh.readthedocs.org/zh/latest/)或者[http://tornado.moelove.info/](http://tornado.moelove.info/)阅读，以获得更好的阅读体验(格式支持)。原谅我没排好版QAQ

# tornado.websocket — 浏览器与服务器双向通信
WebSocket 协议的实现

WebSockets 允许浏览器和服务器之间进行 双向通信

所有主流浏览器的现代版本都支持WebSockets(支持情况详见：http://caniuse.com/websockets)

该模块依照最新 WebSocket 协议 RFC 6455 实现.

在 4.0 版更改: Removed support for the draft 76 protocol version.

### class tornado.websocket.WebSocketHandler(application, request, **kwargs)
通过继承该类来创建一个基本的 WebSocket handler.

重写 on_message 来处理收到的消息, 使用 write_message 来发送消息到客户端. 你也可以重写 open 和 on_close 来处理连接打开和关闭这两个动作.

有关JavaScript 接口的详细信息： http://dev.w3.org/html5/websockets/ 具体的协议： http://tools.ietf.org/html/rfc6455

一个简单的 WebSocket handler 的实例： 服务端直接返回所有收到的消息给客户端

class EchoWebSocket(tornado.websocket.WebSocketHandler):
    def open(self):
        print("WebSocket opened")

    def on_message(self, message):
        self.write_message(u"You said: " + message)

    def on_close(self):
        print("WebSocket closed")
WebSockets 并不是标准的 HTTP 连接. “握手”动作符合 HTTP 标准,但是在”握手”动作之后, 协议是基于消息的. 因此,Tornado 里大多数的 HTTP 工具对于这类 handler 都是不可用的. 用来通讯的方法只有 write_message() , ping() , 和 close() . 同样的,你的 request handler 类里应该使用 open() 而不是 get() 或者 post()

如果你在应用中将这个 handler 分配到 /websocket, 你可以通过如下代码实现:

var ws = new WebSocket("ws://localhost:8888/websocket");
ws.onopen = function() {
   ws.send("Hello, world");
};
ws.onmessage = function (evt) {
   alert(evt.data);
};
这个脚本将会弹出一个提示框 :”You said: Hello, world”

浏览器并没有遵循同源策略(same-origin policy),相应的允许了任意站点使用 javascript 发起任意 WebSocket 连接来支配其他网络.这令人惊讶,并且是一个潜在的安全漏洞,所以 从 Tornado 4.0 开始 WebSocketHandler 需要对希望接受跨域请求的应用通过重写.

check_origin (详细信息请查看文档中有关该方法的部分)来进行设置. 没有正确配置这个属性,在建立 WebSocket 连接时候很可能会导致 403 错误.

当使用安全的 websocket 连接(wss://) 时, 来自浏览器的连接可能会失败,因为 websocket 没有地方输出 “认证成功” 的对话. 你在 websocket 连接建立成功之前,必须 使用相同的证书访问一个常规的 HTML 页面.

## Event handlers
### WebSocketHandler.open(*args, **kwargs)
当打开一个新的 WebSocket 时调用

open 的参数是从 tornado.web.URLSpec 通过正则表达式获取的, 就像获取 tornado.web.RequestHandler.get 的参数一样

### WebSocketHandler.on_message(message)
处理在 WebSocket 中收到的消息

这个方法必须被重写

### WebSocketHandler.on_close()
当关闭该 WebSocket 时调用

当连接被彻底关闭并且支持 status code 或 reason phtase 的时候, 可以通过 self.close_code 和 self.close_reason 这两个属性来获取它们

在 4.0 版更改: Added close_code and close_reason attributes. 添加 close_code 和 close_reason 这两个属性

### WebSocketHandler.select_subprotocol(subprotocols)
当一个新的 WebSocket 请求特定子协议(subprotocols)时调用

subprotocols 是一个由一系列能够被客户端正确识别出相应的子协议 （subprotocols）的字符串构成的 list . 这个方法可能会被重载,用来返回 list 中某 个匹配字符串, 没有匹配到则返回 None. 如果没有找到相应的子协议,虽然服务端并 不会自动关闭 WebSocket 连接,但是客户端可以选择关闭连接.

## Output
### WebSocketHandler.write_message(message, binary=False)
将给出的 message 发送到客户端

message 可以是 string 或者 dict（将会被编码成 json ) 如果 binary 为 false, message 将会以 utf8 的编码发送; 在 binary 模式下 message 可以是 任何 byte string.

如果连接已经关闭, 则会触发 WebSocketClosedError

在 3.2 版更改: 添加了 WebSocketClosedError (在之前版本会触发 AttributeError)

在 4.3 版更改: 返回能够被用于 flow control 的 Future.

### WebSocketHandler.close(code=None, reason=None)
关闭当前 WebSocket

一旦挥手动作成功,socket将会被关闭.

code 可能是一个数字构成的状态码, 采用 RFC 6455 section 7.4.1. 定义的值.

reason 可能是描述连接关闭的文本消息. 这个值被提给客户端,但是不会被 WebSocket 协议单独解释.

在 4.0 版更改: Added the code and reason arguments.

## Configuration
### WebSocketHandler.check_origin(origin)
通过重写这个方法来实现域的切换

参数 origin 的值来自 HTTP header 中的``Origin``,url 负责初始化这个请求. 这个方法并不是要求客户端不发送这样的 heder;这样的请求一直被允许（因为所有的浏览器 实现的 websockets 都支持这个 header ,并且非浏览器客户端没有同样的跨域安全问题.

返回 True 代表接受,相应的返回 False 代表拒绝.默认拒绝除 host 外其他域的请求.

这个是一个浏览器防止 XSS 攻击的安全策略,因为 WebSocket 允许绕过通常的同源策略 以及不使用 CORS 头.

要允许所有跨域通信的话（这在 Tornado 4.0 之前是默认的）,只要简单的重写这个方法 让它一直返回 true 就可以了:

def check_origin(self, origin):
    return True
要允许所有所有子域下的连接,可以这样实现:

def check_origin(self, origin):
    parsed_origin = urllib.parse.urlparse(origin)
    return parsed_origin.netloc.endswith(".mydomain.com")
4.0 新版功能.

### WebSocketHandler.get_compression_options()
重写该方法返回当前连接的 compression 选项

如果这个方法返回 None (默认), compression 将会被禁用. 如果它返回 dict (即使 是空的),compression 都会被开启. dict 的内容将会被用来控制 compression 所 使用的内存和CPU.但是这类的设置现在还没有被实现.

4.1 新版功能.

### WebSocketHandler.set_nodelay(value)
为当前 stream 设置 no-delay

在默认情况下, 小块数据会被延迟和/或合并以减少发送包的数量. 这在有些时候会因为 Nagle’s 算法和 TCP ACKs 相互作用会造成 200-500ms 的延迟.在 WebSocket 连接 已经建立的情况下,可以通过设置 self.set_nodelay(True) 来降低延迟（这可能 会占用更多带宽）

更多详细信息： BaseIOStream.set_nodelay.

在 BaseIOStream.set_nodelay 查看详细信息.

3.1 新版功能.

## Other
### WebSocketHandler.ping(data)
发送 ping 包到远端.

### WebSocketHandler.on_pong(data)
当收到ping 包的响应时执行.

### exception tornado.websocket.WebSocketClosedError
出现关闭连接错误触发.

3.2 新版功能.

## Client-side support
### tornado.websocket.websocket_connect(url, io_loop=None, callback=None, connect_timeout=None, on_message_callback=None, compression_options=None)
客户端 WebSocket 支持 需要指定 url, 返回一个结果为 WebSocketClientConnection 的 Future 对象

compression_options 作为 WebSocketHandler.get_compression_options 的 返回值, 将会以同样的方式执行.

这个连接支持两种类型的操作.在协程风格下,应用程序通常在一个循环里调用`～.WebSocket ClientConnection.read_message`:

conn = yield websocket_connect(url)
while True:
    msg = yield conn.read_message()
    if msg is None: break
    # Do something with msg
在回调风格下,需要传递 on_message_callback 到 websocket_connect 里. 在这两种风格里,一个内容是 None 的 message 都标志着 WebSocket 连接已经.

在 3.2 版更改: 允许使用 HTTPRequest 对象来代替 urls.

在 4.1 版更改: 添加 compression_options 和 on_message_callback .

不赞成使用 compression_options .

### class tornado.websocket.WebSocketClientConnection(io_loop, request, on_message_callback=None, compression_options=None)
WebSocket 客户端连接

这个类不应当直接被实例化, 请使用 websocket_connect

#### close(code=None, reason=None)
关闭 websocket 连接

code 和 reason 的文档在 WebSocketHandler.close 下已给出.

3.2 新版功能.

在 4.0 版更改: 添加 code 和 reason 这两个参数

#### write_message(message, binary=False)
发送消息到 websocket 服务器.

#### read_message(callback=None)
读取来自 WebSocket 服务器的消息.

如果在 WebSocket 初始化时指定了 on_message_callback ,那么这个方法永远不会返回消息

如果连接已经关闭,返回结果会是一个结果是 message 的 future 对象或者是 None. 如果 future 给出了回调参数, 这个参数将会在 future 完成时调用.
