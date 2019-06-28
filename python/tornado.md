# Tornado.websocket
## Tornado简介
Tornado-基于Python的web服务端框架，
与现有主流的web服务端（以及大多数Python框架）有着明显的区别：
它是非阻塞式，速度相当快。得利于其非阻塞式的方式，Tornado每秒可以处理数以千计的连接，因此是实时web服务的一个理想框架。

## Tornado使用

### Tornado安装
`pip install tornado # 这里我们使用的版本是5.1  Python版本是3.6`

### Tornado web程序流程
1. 创建web应用实例对象，第一个初始化参数为路由映射列表
2. 定义实现路由映射列表的handler类
3. 创建实例，监听服务器端口
4. 启动。执行IOLoop类的start()方法  
### 处理HTTP请求
实现下列方法来处理HTTP请求
* RequestHandler.get(*args, **kwargs)
* RequestHandler.post(*args, **kwargs)
* RequestHandler.put(*args, **kwargs)
* RequestHandler.delete(*args, **kwargs)
### Hello world
 ```python
import tornado.ioloop
import tornado.web

class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.write("Hello world")

class Application(tornado.web.Application):
    def __init__(self):
        handlers = [
            (r'/index', MainHandler),
        ]
        tornado.web.Application.__init__(self, handlers)

if __name__=="__main__":
    app = Application()
    app.listen(8000)
    tornado.ioloop.IOLoop.current().start()

```
这个Hello world创建了一个socket服务并监听8000端口，当接收到请求时根据路由规则来找到相应的类处理该请求，并根据请求方式指定相应类中的指定方法处理。
所以当我们在浏览器键入127.0.0.1:8888/index，服务端会给浏览器返回Hello world。
## Websocket
WebSocket 是 HTML5 开始提供的一种在单个 TCP 连接上进行全双工通讯的协议。  
Tornado中定义了tornado.websocket.WebSocketHandler来处理websocket请求。
### WebSocketHandler类中方法简介
#### open()
当websocket连接建立后被调用
#### on_message(message) *该方法必须被重写
当收到客户端发送的消息时被调用
#### on_close()
当websocket连接关闭后被调用
#### write_message(message, binary=False)
向客户端发送消息，message可以是字符串或字典（字典会被转为json）。若binary为False，则message以utf8编码发送；二进制模式（binary=True）时，可发送任何字节码。
#### close()
关闭websocket连接
#### check_origin(origin)
判断源origin，对于符合条件（返回判断结果为True）的请求源origin允许其连接，否则返回403。可以重写此方法来解决WebSocket的跨域请求（如始终return True）。

### websocket demo
 ```python
import tornado.ioloop
import tornado.web
import tornado.websocket


class ConnectHandler(tornado.websocket.WebSocketHandler) :
    def check_origin(self, origin) :
        '''重写同源检查 解决跨域问题'''
        return True

    def open(self) :
        '''新的websocket连接后被调动'''
        self.write_message('Welcome')

    def on_close(self) :
        '''websocket连接关闭后被调用'''

    def on_message(self, message) :
        '''接收到客户端消息时被调用'''
        self.write_message('new message :' + message)  # 向客服端发送


class MainHandler(tornado.web.RequestHandler) :
    def get(self) :
        self.write("Hello world")


class Application(tornado.web.Application) :
    def __init__(self) :
        handlers = [
            (r'/index', MainHandler),
            (r'/ws', ConnectHandler)
        ]
        tornado.web.Application.__init__(self, handlers)


if __name__ == "__main__" :
    app = Application()
    app.listen(8000)
    tornado.ioloop.IOLoop.current().start()

```
```html
<html>
<header>
    <title>Tornado WebSocket</title>

</header>
<body>
<strong>Connect websocket:</strong>
 <input type="button" onclick="sendMsg();" value="Connect">

 <script type="text/javascript">


  function sendMsg() {
      if ("WebSocket" in window) {
          var ws = new WebSocket("ws://localhost:8000/ws");
            ws.onopen = function() {
               ws.send("Hello");
            };
            ws.onmessage = function (evt) {
               alert(evt.data);
            };
      }
  }
</script>
</body>
</html>

```
这个demo创建了一个websocket服务并监听8000端口，在浏览器建立连接之后会触发open()方法向浏览器推送“welcome”消息，在收到浏览器发送的消息后会触发on_message()方法向浏览器推送“new message :”消息。  
到此为之，一个简单的websocket应用已经可以了。  
但是这个demo太过于简单，他只实现了在建立连接时和收到消息时的一些操作，并没有体现websocket的主要作用（实时推送数据）。
### websocket demo2

```python
import tornado.ioloop
import tornado.web
import tornado.websocket


class ProStatus():
    connector = {}  # 记录当前连接的user

    def user_connect(self, user):
        if user not in self.connector:
            self.connector[user] = set()

    def user_remove(self, user):
        self.connector.remove(user)

    def trigger(self, message):
        ''' 向所有被记录的客户端推送最新内容 '''
        for user in self.connector:
            user.write_message(message)


class ReceiveHandler(tornado.web.RequestHandler):
    def get(self):
        msg = self.get_argument('msg', '')
        ProStatus().trigger(msg) # 接收到消息之后推送


class ConnectHandler(tornado.websocket.WebSocketHandler):
    def check_origin(self):
        '''重写同源检查 解决跨域问题'''
        return True

    def open(self):
        '''新的websocket连接后被调动'''
        ProStatus().user_connect(self) #用户连接后记录
        self.write_message('Welcome')

    def on_close(self):
        '''websocket连接关闭后被调用'''
        ProStatus().user_remove(self)  # 断开连接后remove

    def on_message(self, message):
        '''接收到客户端消息时被调用'''
        self.write_message('new message :' + message)


class IndexHandler(tornado.web.RequestHandler):
    def get(self):
        self.write("Hello world")


class Application(tornado.web.Application):
    def __init__(self):
        handlers = [
            (r'/index', IndexHandler),
            (r'/ws', ConnectHandler),
            (r'/receive', ReceiveHandler)
        ]
        tornado.web.Application.__init__(self, handlers)


if __name__ == "__main__":
    app = Application()
    app.listen(8000)
    tornado.ioloop.IOLoop.current().start()
```
### 一些问题
#### 心跳包
如果websocket隔一段时间不推送数据，那么前端的连接就会自动断开
所以前端建立websocket连接时，需要加入心跳包机制。
**当然心跳包不单单是为了解决该问题，也是为了解决各种意外断开的情况，心跳包很有必要。**
心跳包的原理就是每隔一定的时间检查websocket连接是否断开，在没有断开的情况下隔一定的时间给服务端发送一条消息，这条消息要确保服务端可以回应，如果断开则进行重连。至于前端具体怎么实现心跳包的代码请自行百度。
