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
> 课件： Tornado知识框架.xmind

1. 异步HTTPServer 和 HTTPClient，WebSocket 、异步的ORM
1. 向下：异步技术支持IOLoop 和 IOStream
1. 向上：HTTPServer的基础上Web 框架
1. 整体：基于协程的异步编程
    1. 调用阻塞函数
    1. 并行
    1. 交叉运行
    1. 后台运行



很多公司选择Tornado:
​	FriendFeed FaceBook,Quora，国内的知乎

官方文档： http://www.tornadoweb.org/en/stable/index.html
中文文档： http://www.tornadoweb.cn/documentation

### 2. 安装

~~~bash
pip install tornado
~~~

### 3. HelloWord

```python
from tornado import web, ioloop,


class MainHandler(web.RequestHandler):
    def get(self):
        self.write("Hello, world")


if __name__ == "__main__":
    app = web.Application([
        (r"/", MainHandler),
    ])
    app.listen(8888)
    print("server run http://127.0.0.1:8888")
    ioloop.IOLoop.current().start()
```

### 4. 阻塞的请求
```python
class SyncHandler(web.RequestHandler):

    def get(self):
        start = int(time.time())
        time.sleep(3)

        end = int(time.time())
        self.write(f"耗时{end - start}, 开始时间{start},结束时间{end}")
```

### 5. 非阻塞的请求

```python
class ASyncHandler(web.RequestHandler):

    async def get(self):
        start = int(time.time())
        await gen.sleep(3)

        end = int(time.time())
        self.write(f"耗时{end - start}, 开始时间{start},结束时间{end}")
```

### 练习

1. 使用Tonrado创建一个网页，监听端口9819
2. 访问http://127.0.0.1:9819/ 时 显示当前的日期和时间
3. 访问http://127.0.0.1:9819/sync 等待3秒后显示'hello'，在这时间中**不可以**访问首页
4. 访问http://127.0.0.1:9819/async 等待3秒后显示`'hello'，在这时间中可以访问首页
