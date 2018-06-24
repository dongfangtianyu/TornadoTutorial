# Tornado框架
### 0. 引子
假设一个场景：数万人正在看在线直播，同时使用弹幕进行交流。
一个非常基本的要求是，当某个人发送了一个新的弹幕之后，其他的所有人应该及时的收（看起来像个网络聊天室），应该怎么样实现的？

1. 每个人不断的刷新，这样有新弹幕的时候会在他某一次刷新之后显示处理
2. 每个人发送一个请求，要求服务器去查有没有新数据，没有就一直查下去，知道有了再返回结果，下一次要刷新的时候再发一个请求，让服务器去查，直到又查到数据
3. 建立一个长链接，没当有新弹幕服务器主动发送给每个人



### 1. 介绍
   python web框架及异步网络库，是FriendFeed网站的基础框架，通过使用非阻塞的网络I/O，**Tornado可以支持数万的开放连接**，对于需要保持活动用户和服务器连接的应用是理想的选择。**对长轮询，websocket都有很好的支持**。

##### 高并发就像上厕所
坑位有限，出一个进一个，
还是都先依次进去，然后依次出来

### 特色：

    1. 提供可直接用于生产环境的内部http服务器
    2. 全栈web框架
    3. 提供高效网络库，尤其是异步I/O支持，超时事件处理，不仅可以做web应用服务器框架，同也可以用做爬虫、物联网关、游戏服务器等后台应用
    4. 完备的websocket支持
    5. 非WSGI部署
    6. 大型站点的接口服务框架

### 学习Tornado学什么？
1. 异步HTTPServer 和 HTTPClient，WebSocket 、异步的ORM、 异步函数中调用同步函数
1. 向下：异步技术支持IOLoop 和 IOStream
1. 向上：HTTPServer的基础上Web 框架
1. 整体：基于协程的异步编程



很多公司选择Tornado:
	FriendFeed FaceBook,Quora，国内的知乎

官方文档： http://www.tornadoweb.org/en/stable/index.html
中文文档： http://www.tornadoweb.cn/documentation

### 2. 安装
    ```bash
    pip install tornado
    ```
    
### 3. Helloword
    ```python
    import tornado.ioloop
    import tornado.web
    
    class MainHandler(tornado.web.RequestHandler):
        def get(self):
            self.write("Hello, world")
    
    def make_app():
        return tornado.web.Application([
            (r"/", MainHandler),
        ])
    
    if __name__ == "__main__":
        app = make_app()
        app.listen(8888)
        tornado.ioloop.IOLoop.current().start()
    ```
    
    
### 4.非堵塞的请求