# 支持异步代码的单元测试



好的程序员应该为自己自己的代码写好测试。所以我们需要编写单元测试，验证之前完成的功能，这也为日后重构打下基础。

为了掌握这一节内容，我们需要先了解两个的知识点：

1. 单元测试
2. 协程

关于异步的主要是内容是：事件循环、协程，更多内容可以回顾chapter 3，
这里补充一些单元测测试的内容，以便对聊天室项目进行测试。

**UnitTest** 不仅仅是软件测试中一种方法，也是Python标准库提供的测试框架。

### 1. 简单的测试用例



```python
import unittest     # 1. 导入模块


class MyTestCase(unittest.TestCase): # 2. 创建测试用例

    def test_upper(self):  # 3. 定义测试方法
        self.assertEqual('foo'.upper(), 'FOO')  # 4. 检查预期结果


if __name__ == '__main__':
    unittest.main()  # 5. 执行测试

```
在这个简单的例子中，unittest框架其实做了很多工作，但是我们只需要关注测试方法中的断言。
一般来说，测试方法在执行是出现断言异常（AssertionError） 说明测试失败，如果正常执行结束表示测试通过。

> 如果出现AssertionError之外异常，属于什么？

所以我们可以直接执行测试用法，得到测试结果

```python
MyTestCase().test_upper()
```

这样的演示是为了让大家清除测试用例的执行原理，不推荐大家抛弃框架手动执行用例，比如有多个测试用例，怎么样在出现断言异常后，继续执行下一次测试用例？还是应当充分利用框架的各样功能和优势。

### 2. 简单的协程

协程是Python异步解决方案之一，特点是轻量、灵活，适用于IO密集型的业务场景，比如网络请求、文件读写。

> 除了协程，还有哪些异步方案？

在Python 3.7中，可以使用下面的快速创建和运行协程

```python
import asyncio  # 1. 导入模块


async def hello():  # 2.1 async 关键字，定义协程
    print("hello")
    await asyncio.sleep(1) # 2.2 await 关键字，等待其他任务完成
    print("world")

if __name__ == '__main__':
    asyncio.run(hello())   # asyncio.run 启动协程
```

当然，这样的代码看不出什么效果。作为异步方案，当任务数大于1的时候，才会显示效果，比如下面的例子，打印多个"hello word"，也会1秒完成

```python
async def main():
    # await hello()
    # await hello() # 两个任务串行

    await asyncio.gather(hello(), hello(),)  # 两个任务并


if __name__ == '__main__':
    print(f"started at {time.strftime('%X')}")
    asyncio.run(main())
    print(f"finished at {time.strftime('%X')}")
```

### 3. 协程和函数的区别

从新的 async/await这样的关键字，可以隐约发觉协程和函数是有区别的。

不仅有区别，而且区别很大
首先用函数的运行方式运行main
```python
main()
```
会产生什么效果呢？什么效果效果都米有看到。只是创建了一个协程，但是协程没有运行

如果要在屏幕上出现打印的内容（实际上就是hello执行了）需要

```python
    coroutine = main()  # 创建协程
    asyncio.run(coroutine) # 执行协程代码
    
    # 以下为单步细节，不要求掌握
    loop = asyncio.get_event_loop() # 获取事件循环

    future = asyncio.ensure_future(coroutine, loop=loop)
    # 通过创建一个future，把协程和事件循环联系起来

    loop._run_once() # 循环运行一次，执行需要完成的步骤
    loop._run_once()
```

**区别在于：**
1. 函数调用时执行内部代码
2. 协程调用之后产生协程对象（coroutine object ），但不会执行内部代码
3. 通过事件循环运行协程里的代码

所以下面这个本应该测试失败的用例的，结果却是**测试通过**，因为测试代码根本没有执行，没有进行断言，没有异常，框架“误以为”测试通过，这个前面第一部分介绍过
```python
class MyTestCase(unittest.TestCase): # 2. 创建测试用例

    async def test_upper(self):  # 3. 定义测试方法
        assert 2 ==3  # 4. 检查预期结果
```

### 4. 在测试框架中使用协程

```python
from tornado.testing import * # 1. 导入异步测试框架


class MyTestCase(AsyncTestCase):  # 2. 创建测试用例

    @gen_test    # 4.9 运行协程
    async def test_upper(self):  # 3. 定义测试方法
        assert 2 ==3  # 4. 检查预期结果


if __name__ == '__main__':
    unittest.main()  # 5. 执行测试
```

### 5. 测试Tornado程序的功能

前面解释了测试框架中使用异步方法，你可能会有这样的疑惑：**为什么一定要在测试中使用异步代码（协程）？**

两个理由：

1. 为Tornado 项目在单元测试时，需要启动项目，很显然，这需要异步的支持

2. Tornado 提供了HTTP Client、TCP Client、WebSocket Client等工具，这些工具是异步的，如果普通项目做测试时用到了这些工具，就需要异步来驱动

   

哈哈，所以Tornado开发必须掌握够在测试中使用异步 :)

来看看怎么为聊天室项目添加单元测试吧

#### 5.1 AsyncTestCase

UnitTest的子类，增加了异步的支持

#### 5.1 AsyncHTTPTestCase

AsyncTestCase的子类，增加了异步http server 和http client，
其中:

http serve 需要重写 `get_app`方法，并返回web.Application 的实例 

http client 通过`self.fetch` 调用

```python
class TestHelloApp(AsyncHTTPTestCase):
    def get_app(self):
        return hello.make_app()

    def test_homepage(self):
        response = self.fetch('/')
        self.assertEqual(response.code, 200)
        self.assertEqual(response.body, 'Hello, world')
  
    @gen_test    
    async def test_homepage_async(self):
        response = await self.fetch('/')
        self.assertEqual(response.code, 200)
        self.assertEqual(response.body, 'Hello, world')
```

**注意**：http client 不是ws client，所以测试ws接口时 self.fetch 不适用

#### 5.1 AsyncHTTPSTestCase

AsyncHTTPTestCase的子类，将通讯协议改为**HTTPS**