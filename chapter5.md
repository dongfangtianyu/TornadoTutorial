# Web框架

> Web程序是最常见的网络程序，和之前的HTTPClient不同的是，Web程序属于HTTPServer，最大的区别在于，Server程序为 等待客户端的访问，是需要一直保持运行的。并且为了支持多个客户端同时进行的访问，异步显得比较重要
>
> Tornado 的web分为2部分，HTTP server 接受请求， Application 处理请求。理论上这两部分都可以使用其他实现代替，但是Tornado建议使用它原生的搭配，所以我们不会刻意分离二者，
>
> Web程序中，我们更多的，是使用工具实现各样功能，常见的概念有：路由、处理程序、模板

### 主要内容

1. RequestHandler      1.3
2. Application         0.5
3. 彩蛋：HTTPServer    0.2 -> 2
4. 路由                0.7
5. 彩蛋：内置的Handler 0.3 -> 1
6. 设置                0.1
7. 模板                0.9 -> 1

#### 1. RequestHandler

##### 1.1 Hello Word

```python
from tornado import ioloop, web


class MainHandler(web.RequestHandler):  # 1.1 定义 Handler，处理请求
    def get(self):                      # 1.2 定义http请求发放同名的方法，根据http方法自动调用
        data = self.get_argument("name", "None")  # 1.3 接收访问的参数
        self.write(f"Hello, world, you input {data}")      # 1.4 调用父类方法，创建响应内容
        self.set_status(201)            # 1.5  调用父类方法，指定响应状态码


def make_app():
    router = [
        (r"/", MainHandler),             # 将处理器和 根路径 "/"关联起来，组成路由映射表
    ]
    return web.Application(router)       # 2.1 实例化Application ，并接收路由表


if __name__ == "__main__":
    app = make_app()  # 创建app
    app.listen(8888)  # 2.2 指定端口
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
  注意：如果要处理上面列表之外的方法，需要先扩展SUPPORTED\_METHODS属性,非则会响应 HTTP 405

  ```python
  class WebDAVHandler(RequestHandler):
    SUPPORTED_METHODS = RequestHandler.SUPPORTED_METHODS + ('PROPFIND',)

    def propfind(self):
        pass
  ```

1.2.3 此外，RequestHandler还有其他的方法在这个过程中被执行，都可以在子类中重写

| 执行时机 | 执行方法 |
| :--- | :--- |
| RequestHandler 实例化 | initialize |
| 找到处理方法 | prepare |
| 调用处理方法 | get/post/put/delete/…… |
| 请求处理完成 | on\_finish |

1.2.4 除了有框架调用的方法，还有些方法是可以在处理器中被主动调用，例如：

```python
def get(self): 
    self.write("Hello, world") # 创建响应内容
    self.set_status(201)  指定响应状态码
```

1. 获取输入  

   | 方法 | 用户 |  
   | :--- | :--- |  
   | get\_argument | 获取参数 （与get\_query\_argument 相同） |  
   | get\_arguments | 获取参数的多个值，返回值是列表 |  
   | get\_query\_argument | 获取查询字符串 （在url中） |  
   | get\_body\_argument | 获取请求参数（在body中） |  
   | get\_argument | 设置响应状态码 |


2. 生成输出 
 
   | 方法 | 用户 |  
   | :--- | :--- |  
   | set\_status | 设置响应状态码 |  
   | set\_header | 设置响应头 |  
   | clear\_header | 清除已设置的响应头 |  
   | write | 设置响应内容 |  
   | clear | 清除已设置的响应头和响应正文 |  
   | flush | 输出已设置的响应内容 |  
   | finish | 结束此次请求 |  
   | render | 调用模板生成响应内容，并结束会话 |  
   | redirect | 重定向 |

#### 1.3 RequestHandler 其他 属性 和 方法

参阅 [http://www.tornadoweb.org/en/stable/web.html\#other](http://www.tornadoweb.org/en/stable/web.html#other)

#### 1.4 异步的 Handler

> 课件 5.tornado\_handler\_async.py

在课件的实例代码中，同步和异步的区别体现在get方法中：

1. Tornado默认使用单进程、单线程、异步的方式执行
2. 在单线程的异步程序中，如果某个任务阻塞了，那么整个程序都会阻塞，要避免这种情况
3. 编写异步的Handler， 其实就是编写异步的Python程序，比如asycn / await 关键字，调用非阻塞模块，等等

#### 2. Application

Application 是请求处理的集合。  
此类的实例是可以被调用的，被调用时，接受【request】，并根据【路由】找到合适的【RequestHandler】处理请求  \(tornado.web.Application\#**call**\)

##### 2.1 如何实例化Application

另一版本的 make\_app：

```python
def make_app():
    router = [
        (r"/", MainHandler),             # 将 处理器 和 路径 关联起来，组成路由映射表
    ]
    host = "www.baidu.com"               # 为match host指定默认值，该值是为了配合application.add_handlers
    transforms = []                      # 类似于django中间件，可以对响应进行二次处理 ，参考 tornado.web.OutputTransform
    settings = {}                        # 设置选项

    app = web.Application(handlers=router,
                          default_host=host,
                          transforms=transforms,
                          **settings,
                          )       # 6. 实例化Application ，并接收路由表

    return app
```

##### 2.2 Application 的内置方法

Application 提供了一些内置方法，使得使用一些功能是调用方法就可以实现了。  
当然，我们也可以为Application 定义更多的方法，实现更多功能的便捷调用

* listen

    `实例化HTTPSerer，使application能够`
* add\_handlers

    `给Application 增加更多的处理器，可以根据host 解析到不同的处理器`
* reverse\_url

    `根据name，返回url。使用场景很多`

#### 3.彩蛋：HTTPServer

在Application 的listen 源代码如下（Tornado 5.1）：

```python
    def listen(self, port, address="", **kwargs):
        from tornado.httpserver import HTTPServer
        server = HTTPServer(self, **kwargs)
        server.listen(port, address)
        return server
```

可见，虽然表面是Application 的listen，然实际上是调用HTTPServer.listen

> 那么 HTTPServer 又是什么呢？

**HTTPServer ** 是非阻塞的单线程HTTP服务器。  
它监听端口、接受请求，并转交给Application ，看起来和nginx或apche做同样的工作。  
不过，因为GIL的原因，通常我们会启动多个tornado进程，运行在负载均衡后面

> HTTPServer 如何启动？

* 简单的启动 ,也就是Application.listen所使用的方式
  ```python
    server = HTTPServer(app)
    server.listen(8888)
  ```
* 多进程启动

  ```python
    server = HTTPServer(app)
    server.bind(8888)
    server.start(0)  # 给start方法的传参指定进程数，0表示根据CPU核心数确定，会forks多进程
  ```

* 更灵活的多进程启动

  ```python
    sockets = tornado.netutil.bind_sockets(8888)
    tornado.process.fork_processes(0) # 传参指定进程数，0表示根据CPU核心数确定
    server = HTTPServer(app)
    server.add_sockets(sockets) # 手动指定sockets
  ```

  ** 以上三种方式，看起来写法完全不同。但是背后是同一套执行逻辑。  
  只不过简单的方法将这些逻辑包装隐藏了，而复杂的方法重新将这些逻辑暴露出来，并允许我们手动设定**

通常情况下，我们不需要直接和HTTPServer打交道，而是通过Application间接的启动。  
不过，当你进行复杂的设定（比如用你改造过的HTTPServer启动应用）时，需要了解一下灵活的用法

---

#### 4. 路由

> 课件：5.路由器中的路由表

广义来说，路由就是将接收到的数据，转发给目标。在这个工作中包含两个基本的动作：  
1. 确定转发路径  
2. 把数据发送过去

很多时候**我们所说的路由**，只是指第一步，即“确定转发路径”。而这一步，通常是利用路由表实现的

> 课件：5.路由器中的路由表.jpg

在这样路由表中，记录两个关键信息： **要访问的目标 、能够处理这个目标的接口 **  
有了这样的两个信息，系统就能够知道该交给谁来处理了

在Tornado（以及其他的一些Web框架）中，也是利用“类似的路由表”告诉系统，用户访问的目标，交给谁来处理。
这样的路由表，作为Application的第一个参数，成用应用的一部分

> 课件 5.tornado_router.py


##### 4.1 静态路由
静态路由号称钢铁直男，简单、直接的 将某个url 指向 处理器
第一个参数是字符串，第二个参数是处理器
```python
router = [  # 定义路由映射
        (r"/sync", SyncHandler),  # 用户访问 /sync, 由SyncHandler来进行处理
]
```

##### 4.2 动态路由
动态路由是善变的，所以需要将满足某种情况的多个url 交给 同一个处理器
第一个参数是正则表达式，第二个参数是处理器

```python
router = [  # 定义路由映射
        (r"/sync([0-9]+)", SyncHandler),  
        # 用户访问 /sync+数字, 由SyncHandler来进行处理，并且处理器知道这个数字是几
]
```

需要注意，正则从url捕获的内容，会传递给处理方法，所以确保处理方法能够接受参数

```python
class SyncHandler(web.RequestHandler):

    def get(self, num=0): # 处理方法接受参数
        self.write(f"SyncHandler: {num}")

```

##### 4.3 处理器传递跟多的参数
动态路由是善变的，所以需要将满足某种情况的多个url 交给 同一个处理器
第三个参数指定要传递给处理器的【额外的参数】

```python
router = [  # 定义路由映射
        (r"/async", SyncHandler, {"is_async": True}),
        # 用户访问 /async, 同样是由SyncHandler来进行处理，但是处理前进行调整
]
```

同样需要注意，额外的参数是传递给处理器，而不是处理方法，所以需要确保处理器接受参数

```python
class SyncHandler(web.RequestHandler):
    def initialize(self, is_async=False):
        # 回顾第一节RequestHandler的知识
        # initialize方法是在处理器实例化的时候执行的，所以可以设置、修改实例的成用属性，实现移花接木
        if is_async:
            self.get = self._get

    async def _get(self, num=0):
        self.write(f"AsyncHandler: {num}")

```


#### 5. 彩蛋：内置的Handler
在上面得例子当中，通过传递额外得参数，实现了一个了不得得事情 ^_^。

就是，同一个处理器，可以实现完全不同得效果。
换言之，我们可以不需要创建新处理器，通过参数改变已有得处理器，就可以实现同样效果了。
减少了工作量。


Tornado 也提供了一些常用的，可以传参就直接使用的处理器
> 课件 5.tornado_builtin_handlers.py

1. ErrorHandler 抛出错误页面

    ```python
router = [  # 定义路由映射
        (r"/error", web.ErrorHandler, dict(status_code=403)),  # 指定错误页面的状态码
]
    ```

2. RedirectHandler

    ```python
router = [  # 定义路由映射
        (r"/go_url=(.*)", web.RedirectHandler, dict(url="/{0}", permanent=False)),  # 302 跳转，指定跳转的目标
        (r"/go_url_p=(.*)", web.RedirectHandler, dict(url="/{0}", permanent=True)),  # 301 跳转
]
    ```

3. StaticFileHandler

    ```python
router = [  # 定义路由映射
        (r"/img/(.*)", web.StaticFileHandler, dict({"path": os.path.join(BASE_DIR, "img")})),  # 访问img目录下的文件
]
    ```

4. FallbackHandler

    ```python
router = [  # 定义路由映射
        (r"/remote/(.*)", web.FallbackHandler, dict(fallback=callback)),  # 把request 交给其他程序执行,比如 其他的wsgi程序
]
    ```

---

#### 6. 设置
除了handlers、default_host 、transforms，其他的Application构造参数都会作为设置被保存起来

完整的设置选项请参阅：http://www.tornadoweb.org/en/stable/web.html#tornado.web.Application.settings
这里我们先列一下常用的

- cookie_secret   ：为cookie签名用的密钥，防止cookie被用户伪造
- static_path  ： 指定静态文件存放路径
- template_path： 指定模板文件存放路径
- debug ： 开启调试模式

```python
impor os 
BASE_DIR = os.path.dirname(__file__)
settings = {
    "cookie_secret": "sd*^%_Nn30NG%*BTq34905y7389vcnvbk,erjtherg",
    "template_path": os.path.join(BASE_DIR, "templates"),
    "static_path": os.path.join(BASE_DIR, "static"),
    # todo 其他设置
}
web.Application(router, **settings)
```


#### 7. 模板
模板一般指以及设置好格式，只差填充内容的数据，或者相似、相同内容已经准备好，只差差异性内容的数据。比如简历模板、合同模板、作文模板。

在Web项目中，很多时候输出的内容是很相似的（html或者json），会常用到模板。业务代码生成好差异性内容后，用模块快速的生成最终的完整页面。

创建的模板，保存在settings.template_path 指定的位置，好让tornado 找到

##### 7.1 处理器 中使用模板
> 课件 5.tornad_template

```python
class MainHandler(web.RequestHandler):

    def get(self):
        context = {"title": "MainHandler_Title"} # 通过模板渲染的内容
        self.render("index.html", **context)
```

##### 7.2 模板的基本用法
1. Python表达式

    `{{ python表达式 或变量名 }}`
    * 如果是变量名，会直接输出变量值
    * 表达式可以是任意的python代码，意味要慎重检查模板代码。
    * 最终输出的是表达式的返回值，注意`print(123)` 的返回值是None而不是123

2. 注释 

    `{# 注释内容 #}`
    * 可以多行注释

3. 标签

    `{% 标签 %}`

    * 可以看作一个函数在执行
    * 内置标签 [清单](http://www.tornadoweb.org/en/stable/template.html#syntax-reference)

4. 静态文件

    `<link rel="stylesheet" href="{{ static_url('my.css') }}">`

    * static_url是处理器的方法，用来根据文件名找到静态文件，并生成带hash的文件url
    * 为什么可以在模板直接使用？因为已经提前放入了模板
    * 实际上处理器的方法、属性，都可以在模板中使用，因为处理器也被放入模板了，只不过别名是handler
    * 所以上面代码也可以写作 `<link rel="stylesheet" href="{{ handler.static_url('my.css') }}">`

4. 模板继承

    `{% extends "base.html" %}`

5. 模板覆写

    `{% block title %}New Title{% end %}` # 覆写父模板title 块的内容， 前提是父模板有这个block

6. 模块引入

    `{% include "left-menu.html" %}` \# left-menu.html的内容引入到当前位置

    > 思考继承和引入的联系与区别：

    > 多个页面共有的内容，可以考虑通过继承或引入来减少模板代码量，方便维护

