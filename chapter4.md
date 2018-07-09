# 异步ORM

> 在程序开发过程中，一般十分需要使用数据库。而ORM则极大的提高了使用数据库的便利性
> 对于异步编程，我们是否一定需要异步ORM？
> 一般而言，这个答案是否定的。但是如果你需要使用异步的方式连接数据库的时候，可以关注一下

### 在Python 怎么使用数据
首先我们回顾一下，在Python 怎么使用数据，这里分别用redis 和mysql 作为演示

1. tornado 中使用redis
2. tornado 中使用mysql


### 如何使用异步的方式连接数据库，使用ORM
3. 使用异步 aioredis
4. 使用peewee_async

    1. ```pip install peewee-async aiomysql```


## 扩展阅读

[为何选用异步ORM？](http://gino.fantix.pro/en/latest/why.html)