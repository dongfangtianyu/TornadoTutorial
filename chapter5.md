# Web框架

> Web程序是最常见的网络程序，和之前的HTTPClient不同的是，Web程序属于HTTPServer，最大的区别在于，Server程序为 等待客户端的访问，是需要一直保持运行的。并且为了支持多个客户端同时进行的访问，异步显得比较重要

### 主要内容

1. RequestHandler
2. Application
3. 静态文件
4. 模板
5. 路由
6. 彩蛋: HTTPServer

#### 1. RequestHandler

##### 1.1 helloword

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

##### 1.2 RequestHandler 的方法
> 主要业务逻的 在RequestHandler 中完成

1.2.1 刚才程序的执行顺序：

1. 【ioloop】使程序一直运行，并在收到请求的的时候通知【Application】

2. 【Application】 的实例收到新的请求会根据【router】，实例化合适的【RequestHandler】（tornado/routing.py:331）

3. 【RequestHandler】根据 request 调用 method同名的 【实例方法】 （/tornado/web.py:1590）

1.2.2 其中常用的request.method有 （tornado/web.py:162）：

* POST
* DELETE
* GET
* PUT
* PATCH
* HEAD
* OPTIONS

  所以RequestHandler中可以重写父类的以上方法，处理相应的请求。
  注意：如果要处理上面列表之外的方法，需要先扩展SUPPORTED_METHODS属性
  ```python
  class WebDAVHandler(RequestHandler):
    SUPPORTED_METHODS = RequestHandler.SUPPORTED_METHODS + ('PROPFIND',)

    def propfind(self):
        pass
        
    ```

1.2.3 此外，RequestHandler还有其他的方法在这个过程中被执行，都可以重写

| 执行时机 | 执行方法 |
| :--- | :--- |
| RequestHandler 实例化 |initialize  |
| 找到处理方法 |prepare  |
| 调用处理方法 |get/post/put/delete/……  |
| 请求处理完成 |on_finish  |



#### 1. Application


