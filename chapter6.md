# Websocket

### 主要内容
- 为什么要使用Websocket
- Websocket 是什么
- Toronado中怎么使用Websocket
- 彩蛋：Websocket客户端
- Websocket的负载均衡

----


### 一、为什么要使用Websocket

我们之前使用的**HTTP**协议，通信方式是：

1. 客户端发起请求给服务端
2. 服务器接到请求后生成响应给客户端

这种方式有一个弊端：**每次通信，必须由客户端发起**。

这意味着：

1. 如果有新内容产生，必须要等客户端请求才能获取（更新不及时）
2. 如果没有新内容产生，客户端也会发送请求，但绝大多数请求之后发现没有新内容 （浪费系统资源）

而**Websocket**协议的最大特点是**服务端可以主动向客户端发送消息**


#### 举个例子
> HTTP轮询:

- 你：请问成绩出来了吗？
- 对方：没有
- 你：请问成绩出来了吗？
- 对方：没有  
- 你：请问成绩出来了吗？
- 对方：没有  
- 你：请问成绩出来了吗？
- 对方：出来了，你考了666分

>Websocket

- 对方：你的成绩出来了，考了666分


##### 效果演示
> 课件 6.websocket_demo1.py 。

### 二、Websocket 是什么


WebSocket是一种浏览器与服务器全双工通信协议，
在2011年成为国际标准，现在被所以的主流浏览器支持。

> 课件 6.Websocket.pptx 

#### 要搞清楚的是，WebSocket的整个过程，分为2个部分：

1. 握手：HTTP 
	- 既然是HTTP，自然有request/response

    > 课件 6.Websocket的请求响应.jpg
	
2. 通信：TCP 
	- 既然是TCP，自然有各种socket的概念
	- 比如：
		- 连接成功
		- 收到消息
		- 对方断开连接
		- 发送消息
		- 主动断开连接


#### 同时需要清楚，浏览器只是承担了【websocket客户端】的角色

websocket客户端还有很多，比如tornado就提供了一个

### 三、Toronado中怎么使用Websocket

#### 在Toronado提供Websocket 和提供HTTP非常相似


```python

router = [
    (r'/', web.RequestHandler()), # HTTP 
    (r'/ws', websocket.WebSocketHandler()), # Websocket
]

```
实际上，websocket.WebSocketHandler是web.RequestHandler的子类。


#### 在使用的时候，稍有区别
1. 不需要处理HTTP请求

	WebSocketHandler只接受`GET`请求。并且`GET`请求只是用来进行websocket握手的,一般不需要修改

2. 收、发WebSocket消息

	在Tornado中，如果收到了WebSocket消息，会把消息传递给`on_message`方法，所以我们只有重写`on_message`方法就可以处理收到的消息了

	```python
	def on_message(self, message):
        print(f"收到新消息: {message}")

	```
	如果想发送消息给客户端，通过调用`write_message`方法实现

	```python
	self.write_message(message)
	```

3. 能不能做更多？ 处理“连接建立”、 “连接断开”的情况 ，甚至主动断开连接
	 - 建立连接，调用 `open`方法
	 - 断开连接，调用 `on_close`方法
	 - 主动断开连接，调用 `close`方法

课件 6.tornado_websocket.py 实现了一个websocket 的“回音壁”程序，并处理了“连接建立”、 “连接断开”的情况
>课件 6.tornado_websocket.py




### 四、 彩蛋：Python版Websocket客户端

在章节”二、Websocket 是什么”的结尾，我们提到了“浏览器只是承担了【websocket客户端】的角色”

只要可以实现TCP编程，都可以websocket客户端还有很多，比如Android、iOS、或者其他服务端语言


##### Tornado 提供了一个Websocket客户端
主要用来测试自己的Websocket功能，
如果你做爬虫的时候发现的信息是用ws实现的话，也可以拿来用用

> 课件 6.tornado_websocket_client.py

因为ws是双全工协议，所以客户端和服务端用法非常相似，但也有一些差别:

1. 由客户端发起握手

	```
	conn = await websocket_connect(url)
	```

2. 如果服务端主动关闭连接，客户端会收到一个None（这个None不是服务端发的）

	```
	msg = await conn.read_message()
	if msg is None:
		print("连接已断开")
	```

3. 协程的方式执行





#### 另一个Python版Websocket客户端

如果你只用ws客户端而不会tornado的其他功能，可以看看这个，单纯的ws客户端，
使用的方法非常的相似，这里不赘述了

https://github.com/websocket-client/websocket-client/



### 五、Websocket的负载均衡

**移动到Tornado部署章节**



### 练习

1. 使用Tonrado实现一个websocket服务，为客户端发送北京时间
2. 使用Tonrado实现一个websocket客户端，连接到练习1的服务端，获取时间并打印出来

