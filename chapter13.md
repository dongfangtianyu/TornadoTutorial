# 服务端部署

当测试通过之后，可以将代码部署到服务器上，进行项目上线了。

代码在上线时，和在本地运行时，有些许不同：

比如代码运行在更稳定的的linux系统上，程序在长久运行过程中可能会奔溃，用户量大时对性能有更高要求



Linux + nginx + tonroado + redis

> 课件：13.部署.pptx

### 1. Linux
   1. CentOS  包管理器 yum

   2. **Ubuntu  包管理器** apt
      搜索 包 `https://launchpad.net/ubuntu/+ppas`
      添加 第三方源

      ```bash
      sudo add-apt-repository ppa:chris-lea/redis-server
      sudo apt-get update
      ```

      
### 2. Python
   1. 编译安装

   2. 第三方源

   3. **conda**

      ``` bash
      wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
      bash Miniconda3-latest-Linux-x86_64.sh
      ```

   4. 设置国内镜像
       ```bash
       # cat ~/.pip/pip.conf
       [global]
       timeout = 6000
       index-url = https://mirrors.aliyun.com/pypi/simple/
       ```

   5. **安装依赖，启动项目**
### 3. 负载均衡代理
   1. HAProxy

   2. **Nginx**
      2.1 安装
      
      ```bash
      sudo apt install nginx -y
      ```
      2.2 配置
      
      ```bash
      touch /etc/nginx/conf.d/chat.conf
      ```
      **需要注意，分别反代http 和websocket，**
      
      **websocket 需要增加请求头才有效**
      
      > 课件 13.chat.conf
      
      2.3 检查配置
      
      ```bash
      sudo nginx -t
      ```
      2.4 应用配置
      ```bash
      sudo nginx -s reload
      ```
      
      2.5 部署静态文件
      ```bash
      cp -R static /var/www
      ```

### 4. 进程管控
   1. Supervisord
      1.1 安装
      ```bash
      pip install supervisor
      mkdir /etc/supervisor
sudo sh -c "echo_supervisord_conf  > /etc/supervisor/supervisord.conf"
      supervisord
      ```
      1.2 配置
      
      ```bash
      vim /etc/supervisor/supervisord.conf
      ;[include]
      files = ini.d/*.ini    ;从目录中加载其他配置
      ```
      ```bash
      mkdir /etc/supervisor/ini.d
      touch /etc/supervisor/ini.d/chat.ini 
      ```
      >课件：13.chat.ini

      1.3 应用配置，启动程序
      
      ```bash
      supervisorctl
      supervisor> update     # 更新配置并执行
      supervisor> status
      supervisor> restart chat:tornado_port=8001  # 重启某个程序
      supervisor> restart chat:          # 重启全部程序
      ```
      
      1.4 自启动
      https://github.com/Supervisor/initscripts
      
      ```bash
      sudo bash -c " wget https://raw.githubusercontent.com/Supervisor/initscripts/master/ubuntu -O /etc/init.d/supervisord"
      
      vim /etc/init.d/supervisord
      # 修改文件具体路径
      
      sudo chmod +x /etc/init.d/supervisord
      sudo update-rc.d supervisord defaults
      service supervisord restar
      ```
      
      1.5 修改nginx上游列表
      
### 5. 项目修改

1. 关掉debug 
   ```python
   {"debug":False}
   ```
3. 修改密钥
   ```python
   {"cookie_secret": "xxxxxxxxxx"}
   ```
   
2. 修改日志等级
   ```python
   logging.getLogger("").setLevel(logging.WARNING)
   ```
