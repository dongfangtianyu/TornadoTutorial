# 异步ORM

> 在程序开发过程中，一般十分需要使用数据库。而ORM则极大的提高了使用数据库的便利性
> 对于异步编程，我们是否一定需要异步ORM？
> 一般而言，这个答案是否定的。但是如果你需要使用异步的方式连接数据库的时候，可以关注一下

### 在Python 怎么使用数据
首先我们回顾一下，在Python非异步编程中 怎么使用数据，这里分别用redis 和mysql 举例
> 课件：4.sync_db.py

1. tornado 中使用redis
    1. ```pip install redis-py```
2. tornado 中使用mysql
    1. `pip install mysqlclient`


### 如何使用异步的方式连接数据库，使用ORM
> 课件：4.async_db.py

3. 使用异步 aioredis

    1. ```pip install  aioredis```

4. 使用peewee_async

    1. ```pip install aiomysql```


## 扩展阅读

[为何选用异步ORM？](http://gino.fantix.pro/en/latest/why.html)