聊天室项目1: 实现主要功能
===


### 0. 需求分析
1. 用户界面，显示昵称和输入聊天内容
2. 用A发送的消息，用户B即时收到
3. 屏幕上展示大家的聊天内容
4. 用户加入和退出时有消息通知

### 1. 用户页面
准备html页面

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>我的聊天室</title>

    <!-- csss 写在这里 -->
</head>
<body id="body">

<div id="user" class="left">
    <span>在线用户</span>
    <ul id="user_list">
        <li>AAA</li>
        <li>BBB</li>

    </ul>
</div>

<div class="right">
    <div id="msg_log">
    </div>
    <hr/>

    <div class="input">
        昵称：XXXYYYZZZ <br/>
        内容：<input type="text" id="msg">
        <input type="submit" value="发送" id="send">
    </div>

</div>
<!--  js 写在这里 -->

</body>
</html>
```
在浏览器中显示
```python
class MainHandler(web.RequestHandler):
    def get(self):
        self.render("index.html")
        
```



### 2. websocket回音壁
通过websocket实时通信
1. 前端js代码，发起ws请求
    ```js
        let url = 'ws://' + window.location.host + '/ws'
    var ws = new WebSocket(url)
    ws.onopen = function(event){
       ws.send("hello")
    }
    ws.onmessage = function (event) {
        let msg = JSON.parse(event.data)
        let node = $(msg.html)
        let msg_type = msg.type

        switch (msg_type) {
            default:
                console.log(msg)

        }

    }
    ws.onclose = function (event) {
        alert("ws链接失败")

    }
    ```
2. 后端使用WebSocketHandler 处理ws请求

    ```python
    class WSHandler(websocket.WebSocketHandler):
    
    def on_message(self, message):
        self.write_message(message)
    ```

### 3. websocket群发

```python
clients = set()  # 保存全部在线用户
_todo_remove = set() # 已经离线的会话

class WSHandler(websocket.WebSocketHandler):

    def open(self, *args, **kwargs):
        clients.add(self) # 新会话加入到全局变量

    def on_message(self, message):
        global _todo_remove
        for client in clients:  # 给每个会话发送消息
            try:
                client.write_message(message)
            except websocket.WebSocketClosedError:
                _todo_remove.add(client)

        for _ in _todo_remove: # 从全局变量中清除断线的会话
            clients.remove(_)

        _todo_remove = set()
```

### 4.  用户进入和离开通知

1. 区分不同会话
    ```python
    class UserMixin:

    __user_ip = None

    @property
    def user_ip(self):
        if self.__user_ip:
            return self.__user_ip
        ip = self.request.remote_ip

        try:
            ip = ip.split(".")
            return ".".join(ip[:2] + ["*", "*"])
        except:
            return ".".join(["*", "*", "*", "*", ])

    @property
    def current_user(self):
        user = self.get_secure_cookie("username")
        if user:
            return user.decode("utf-8")
        else:
            return ""
    ```
2. 处理新建立链接和断开链接的情况
    ```python
    class WSHandler(UserMixin, websocket.WebSocketHandler):
    
        def open(self, *args, **kwargs):
            clients.add(self)
    
            for client in clients:  # 消息广播
                try:
                    client.write_message(f"{self.current_user} ({self.user_ip}) 进入了聊天室")
                except websocket.WebSocketClosedError:
                    pass
    
        def on_message(self, message):
            global _todo_remove
            message = f"{self.current_user} ({self.user_ip}): {message}"
            for client in clients:  # 消息广播
                try:
                    client.write_message(message)
                except websocket.WebSocketClosedError:
                    pass
    
        def on_close(self):
    
            clients.remove(self)  # 从全局变量中清除断线的会话
            for client in clients:  # 消息广播
                try:
                    client.write_message(f"{self.current_user} ({self.user_ip}) 离开了聊天室")
                except websocket.WebSocketClosedError:
                    pass
    
    ```
3. 设置cookies密钥
    ```python
    app = web.Application([
        (r"/", MainHandler),
        (r"/ws", WSHandler),
    ],
        cookie_secret= "sd*^%_Nn30NG%*BTq34905y7389vcnvbk,erjtherg"
    )
    ```

### 5. 在页面上发送和显示消息

1. 配置静态文件，引入jquery
    ```python
    app = web.Application([
        (r"/", MainHandler),
        (r"/ws", WSHandler),
    ],
        cookie_secret="sd*^%_Nn30NG%*BTq34905y7389vcnvbk,erjtherg",
        static_path=os.path.join(BASE_DIR, "static"),
    )
    ```

    ```html
    <script type="application/javascript" src="{{ static_url('jquery.min.js') }}"></script>
    <script type="application/javascript" src="{{ static_url('chat.js') }}"></script>
    ```

2. 编写js代码
    ```js
    function send_msg(ws) {
        msg = $("#msg").val()
        ws.send(msg)
        $("#msg").val("")
    }
    
    $(document).ready(function () { // 页面加载完成，执行的js
            let url = 'ws://' + window.location.host + '/ws'
            var ws = new WebSocket(url)
            ws.onmessage = function (event) { //收到消息
                let msg = event.data
                let node = $("<p>" + msg + "</p>")
                node.hide()
                $("#msg_log").append(node)
                node.slideDown()
    
            }
            ws.onclose = function (event) { //断开链接
                console.log("ws断开链接")
                alert("与服务端断开链接，请重新打开网页")
                location.reload();
            }
      
            $("#send").click(function () { //点击发送
                send_msg(ws)
            })
    
            $("#msg").live("keypress", function (e) {
                if (e.keyCode == '13') { //按下了回车
                    send_msg(ws)
                }
            })
 
        }
    )
    ```