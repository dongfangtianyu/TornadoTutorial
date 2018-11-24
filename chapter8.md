 聊天室项目2: 完善细节
 ===

### 要求用户设定昵称
```python
class MainHandler(UserMixin, web.RequestHandler):
    def get(self):
        if not self.current_user: # 未设定昵称的用户显示login页面
            self.render("login.html")
        else:
            self.render("index.html", username=self.current_user)

    def post(self):
        username = self.get_body_argument("username") # 用户提交的用户名
        self.set_secure_cookie("username", username)
        self.render("index.html", username=username)
```
修改模板，渲染昵称

```html
昵称：{{ username }} <br/>
```

###  区分自己消息和其他人消息
1. 后端

    ```python
       def on_message(self, message):
           html = f'<div class="msg">{self.current_user} ({self.user_ip}): {message}</div>'
           html_self = f'<div class="self">{self.current_user} ({self.user_ip}): {message}</div>'
        
           for client in clients:  # 消息广播
               if client is self: # 群发时不发给自己
                   continue
               try:
                   client.write_message(html)
               except websocket.WebSocketClosedError:
                   pass
       
           self.write_message(html_self) # 群发后重新发送给自己
    ```

2. 前端
    ```js
    ws.onmessage = function (event) {
            let msg = event.data
            let node = $( msg )  // 接收html
            node.hide()
            $("#msg_log").append(node)
            node.slideDown()
    
    }
    ```

### 在线用户列表

1. 发送当前得在线用户列表
```python

def open(self, *args, **kwargs):
    clients.add(self)
    message_live = {
        "type": 'user',
        "html": "".join([f"<li>{client.current_user}</li>" for client in clients])
    }

    message_sys = {
        "type": "sys",
        "html": f'<div class="sys">{self.current_user} ({self.user_ip}) 进入了聊天室</div>'
    }
    for client in clients:  # 消息广播
        try:
            client.write_message(message_sys)
            client.write_message(message_live)
            except websocket.WebSocketClosedError:
                pass

```
2. 为所有得消息添加type

   1. on_message
   2. on_close


3. 前端根据type 选择消息处理方式

 ```javascript
   ws.onmessage = function (event) {
       let msg = JSON.parse(event.data)
       let node = $(msg.html)
       let msg_type = msg.type
   
       console.log(msg)
       node.hide()  //先隐藏
       
       switch (msg_type) {
           case 'user':  // 在线用户列表
               $("#user_list").html(node)
               break;
   
           case 'sys': //系统消息
           case "chat": // 聊天内容
               $("#msg_log").append(node)
               break;
   
           default:
       }
       node.slideDown() //显示html
   
   }
 ```


### 保存聊天记录
```python
chat_logs = [] # 聊天记录

def open(self, *args, **kwargs):
    ...
    message_chat_logs = {
        "type":"chat",
       "html": "".join(chat_logs)
    }
    self.write_message(message_chat_logs)  # 发送之前的聊天记录
    ...

def on_message(self, message):
    ....
    chat_logs.append(message_chat['html'])  # 保存聊天记录
    
```

### 模板文件移动到templates目录

```python
settings = {
    "cookie_secret": "sd*^%_Nn30NG%*BTq34905y7389vcnvbk,erjtherg",
    "template_path": os.path.join(BASE_DIR, "templates"),
    "static_path": os.path.join(BASE_DIR, "static"),# 指定模板目录
    # todo 其他设置
}

app = web.Application(urls, **settings)
```
移动后的文件结构
```bash
$ tree
.
|-- app.py
|-- static
|   |-- chat.js
|   |-- jquery.min.js
|-- templates
    |-- index.html
    |-- login.html
```



### 编写页面样式

1. 在模板中引入css

    ```html
    <link rel="stylesheet"  href="{{ static_url('css.css') }}">
    ```
2. static下创建css文件
    ```css
    .left {
        width: 200px;
        position: absolute;
        bottom: 15px;
    }
    
    .right {
        position: absolute;
        left: 300px;
        bottom: 15px;
        width: 400px;
    }
    
    .msg {
        font-size: 14px;
        color: #41aaff;
    }
    
    .self {
        font-size: 14px;
        color: #34b577
    }
    
    .sys {
        font-size: 12px;
        color: #ff667e;
    }
    ```
