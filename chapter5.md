# Web框架
> Web程序是最常见的网络程序，和之前的HTTPClient不同的是，Web程序属于HTTPServer，最大的区别在于，Server程序为 等待客户端的访问，是需要一直保持运行的。并且为了支持多个客户端同时进行的访问，异步显得比较重要


### 主要内容
1. RequestHandler
2. Application
3. 静态文件
4. 模板
5. 路由
5. 彩蛋: HTTPServer

#### 1. RequestHandler

#### helloword

```python
from tornado import ioloop, web


class MainHandler(web.RequestHandler):  # 1. 定义 Handler，处理请求
    def get(self):                      # 2. 定义http请求发放同名的方法，根据http方法自动调用
        self.write("Hello, world")      # 3， 调用父类方法，创建响应内容
        self.set_status(201)            # 4.  调用父类方法，指定响应状态码


def make_app():
    router = [
        (r"/", MainHandler),             # 5. 将处理器和 根路径 "/"关联起来，组成路由映射表
    ]
    return web.Application(router)       # 6. 实例化Application ，并接收路由表


if __name__ == "__main__":
    app = make_app()  # 创建app
    app.listen(8888)  # 指定端口
    ioloop.IOLoop.current().start()  # 启动事件循环，程序就可以一直运行下去了

```
```