# 异步ORM

> 在程序开发过程中，一般十分需要使用数据库。而ORM则极大的提高了使用数据库的便利性
> 对于异步编程，我们是否一定需要异步ORM？
> 一般而言，这个答案是否定的。但是如果你需要使用异步的方式连接数据库的时候，可以关注一下

### 在Python中 非异步的方式连接数据库
首先我们回顾一下，在Python非异步编程中 怎么使用数据，这里分别用redis 和mysql 举例
> 课件：4.sync_db.py

1. python中使用redis
    1. ```pip install redis-py```
    
2. python中使用mysql
    1. `pip install mysqlclient`


### 在Python中 异步的方式连接数据库

> 课件：4.async_db.py

1. python中使用redis
    1. ```pip install aioredis```
    
2. python中使用mysql
    1. `pip install aiomysql`


### 在Tornado中 非异步的方式连接数据库
> 课件：4.async_db_tornado.py
> 课件：4.Request.http

注意事项：
1. 异步库需要asyncio的loop驱动
2. Tornado 5.0之后的版本。默认使用asyncio的loop
3. 数据库在使用之前，建立连接


### 练习 

修改之前的抓取程序，抓取到的文章标题和链接，存入数据库


## 扩展阅读

[为何选用异步ORM？](http://gino.fantix.pro/en/latest/why.html)