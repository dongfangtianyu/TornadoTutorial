# Tornado框架

### 1. 介绍
   python web框架及异步网络库，是FriendFeed网站的基础框架，通过使用非阻塞的网络I/O，Tornado可以支持数万的开放连接，对于需要保持活动用户和服务器连接的应用是理想的选择。对长轮询，websocket都有很好的支持。

特色：
    1. 提供可直接用于生产环境的内部http服务器
    2. 全栈web框架
    3. 提供高效网络库，尤其是异步I/O支持，超时事件处理，不仅可以做web应用服务器框架，同也可以用做爬虫、物联网关、游戏服务器等后台应用
    4. 完备的websocket支持
    5. 非WSGI部署
    6. 大型站点的接口服务框架

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