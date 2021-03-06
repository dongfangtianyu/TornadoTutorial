# 异步HTTPClient

1. HTTPClient

   发起HTTP请求，并且获取到响应

2. 什么是异步  
    在一次请求完成之前，另一个请求已经开始

3. 有什么好处  
    假设有非常多大的网址需要访问，异步可以在同一时间可以发起大量的请求，快速完成任务

4. Python3

   * yield
   * 生成器
   * 协程
   * 原生协程
   * 异步迭代、异步生成器、异步上下文

---

### 第一个例子

#### 代码
[http://tornado-zh-cn.readthedocs.io/zh\_CN/latest/guide/queues.html](http://tornado-zh-cn.readthedocs.io/zh_CN/latest/guide/queues.html)

```python
    import time
    from datetime import timedelta
    import asyncio
    try:
        from HTMLParser import HTMLParser
        from urlparse import urljoin, urldefrag
    except ImportError:
        from html.parser import HTMLParser
        from urllib.parse import urljoin, urldefrag

    from tornado import httpclient, gen, ioloop, queues

    base_url = 'http://www.tornadoweb.org/en/stable/'
    concurrency = 10


    @gen.coroutine
    def get_links_from_url(url):
        """Download the page at `url` and parse it for links.

        Returned links have had the fragment after `#` removed, and have been made
        absolute so, e.g. the URL 'gen.html#tornado.gen.coroutine' becomes
        'http://www.tornadoweb.org/en/stable/gen.html'.
        """
        try:
            response = yield httpclient.AsyncHTTPClient().fetch(url) # 发送http请求
            print('fetched %s' % url)

            html = response.body if isinstance(response.body, str) \
                else response.body.decode()
            urls = [urljoin(url, remove_fragment(new_url))
                    for new_url in get_links(html)]
        except Exception as e:
            print('Exception: %s %s' % (e, url))
            raise gen.Return([])

        raise gen.Return(urls)


    def remove_fragment(url):
        pure_url, frag = urldefrag(url)
        return pure_url


    def get_links(html):
        class URLSeeker(HTMLParser):
            def __init__(self):
                HTMLParser.__init__(self)
                self.urls = []

            def handle_starttag(self, tag, attrs):
                href = dict(attrs).get('href')
                if href and tag == 'a':
                    self.urls.append(href)

        url_seeker = URLSeeker()
        url_seeker.feed(html)
        return url_seeker.urls


    @gen.coroutine
    def main():
        q = queues.Queue()  # 1. 定义一个队列
        start = time.time()
        fetching, fetched = set(), set()  # 2. 分别保存需要抓取的url，抓取过的url ，思考：为什么用set？

        @gen.coroutine
        def fetch_url():
            current_url = yield q.get()  # 4.1 从队列中取任务目标，这里用了yield，所以取目标的时候交出cpu权限，另外一个任务也开始取
            try:
                if current_url in fetching: # 4.2 等到合适的时候（cpu空闲，或是某个任务的目标获取成功），开始处理
                    return

                print('fetching %s' % current_url)
                fetching.add(current_url)
                urls = yield get_links_from_url(current_url) # 4.3 等到合适的时候，开始从url中获取url（暂时不关注怎么获取的）
                fetched.add(current_url)  # 并且把目标放进已完成列表

                for new_url in urls:
                    # Only follow links beneath the base URL
                    if new_url.startswith(base_url):
                        yield q.put(new_url)   # 4.4 等到合适的时候，把新url放入队列，等待下一次循环的时候处理

            finally:
                q.task_done()

        @gen.coroutine
        def worker():
            while True:
                a = fetch_url()
                yield a

        q.put(base_url)  # 3.给队列放入第一个url

        # Start workers, then wait for the work queue to be empty.
        for _ in range(concurrency):
            worker() # 4. 执行concurrency个任务，任务的内容是执行fetch_url()

        yield q.join(timeout=timedelta(seconds=300))  # 5.等待队列情况（也就是任务完成了再进入下一步）
        assert fetching == fetched
        print('Done in %d seconds, fetched %s URLs.' % (
            time.time() - start, len(fetched)))


    if __name__ == '__main__':
        import logging
        logging.basicConfig()
        io_loop = ioloop.IOLoop.current()
        io_loop.run_sync(main)
```
#### 内容详解 
1. 执行 平均每秒完成10个url的抓取

   > Done in 11 seconds, fetched 127 URLs.

2. async / await

   async 和 await 关键字是PEP 492提出来的，主要目标是解决“生成器（generator）”和“协程（Coroutine ）”，都依赖yield、区别不明确、容易搞糊涂的问题。

   * 首先把所有的yield 替换成await ，
   * 在使用了await 的函数使前加async 
   * 讲raise gen.Return 替换成return

3. Future

   修改结束后发现，程序变成同步了！  
    发生了什么？？ 下面是修改后的代码

   ```python
        async def worker():
            while True:
                await fetch_url()
   ```

   恢复成

   ```python
        @gen.coroutine
        def worker():
            while True:
                yield fetch_url()
   ```

   重新执行，发现程序重新恢复了异步的能力，那么worker 究竟是什么？

   ```python
    print(worker())
   ```

   ```bash
    <Future pending cb=[_make_coroutine_wrapper.<locals>.wrapper.<locals>.<lambda>() at tornado\gen.py:336]>
   ```

4. Task

   ```python
   async def worker():
   while True:
       a = fetch_url()
       await a
   for _ in range(concurrency):
       loop.create_task(worker())
   ```

   恢复异步效果

   > Done in 12 seconds, fetched 127 URLs.

---

### 自己动手编写异步代码

>课件：3.fech_jobbole.py
>课件：3.fech_jobbole_async.py


1. 编写同步代码 （课件）
2. 改成异步
3. 对比执行速度、代码差异


### 练习
使用tornado.httpclient.AsyncHTTPClient 编写一个异步爬虫，
并统计平均每秒爬取的网址数
