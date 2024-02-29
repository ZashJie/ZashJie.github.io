---
title: Webserver 数据库连接池实现
date: 2023-04-01 10:34:00 +0800
categories: [计算机]
tags: [语言/CPP]
pin: false
author: ZashJie

toc: true
comments: true
typora-root-url: ../../ZashJie.github.io
math: false
mermaid: true


---

#项目/WebServer/单Reactor
连接池的实现是通过单例模式创建到一个对象，用RAII机制获取和释放连接
维护 **最大连接数**、**当前可用连接数** 以及 **当前已用连接数** 三个变量 ^44fb65

### [[单例模式]]

##### 实现
使用局部静态变量**懒汉模式**创建连接池
`static connection_pool *GetInstance();`
通过 `GetInstance()` 函数创建并返回一个连接池对象

    ```C++
    connection_pool *connection_pool::GetInstance() {
        static connection_pool connPool;
        return &connPool;
    }
    ```

##### static
表示程序只会在第一次调用 `GetInstance()` 函数时初始化，知道程序结束，所以程序中全局唯一的连接池对象
 
### [[RAII]]的实现
`connectionRAII` 在创建后的构造函数直接调用 `*SQL = connPool->GetConnection();` 获取数据库可用连接；然后RAII对象在局部函数结束后，自动调用析构函数，释放当前使用的连接。


### 信号与锁

##### `GetConnection()` 
获取可用连接时，等待信号量，对连接池进行加锁，获取连接完，当前可用连接数--，当前已用连接数++，处理完进行解锁

##### `ReleaseConnection(MYSQL *con)`
释放当前可用连接，加锁，将使用完的连接放到连接池中，当前可用连接数++，当前已用连接数--，处理完进行解锁

##### `DestroyPool()`
销毁数据库连接池，加锁，遍历连接池中每个连接，一个个进行关闭 `mysql_close(con)` ，最后`connList.clear()` 将连接池列表销毁，处理完进行解锁

### 登陆耗时如何解决？

由于 在目前的项目中，是将所有的用户名密码 `load` 到内存中，登陆时遍历，所以在数据量大的时候很非常耗时

##### 解决方案

利用 `hash` 建立多级索引，加快用户的验证
例如 有 10亿的用户信息，将用户信息进行缩小1000倍的hash，得到100万的hash数据，即一级用户的信息块，再将一级用户信息块再进行 `hash` ，最终得到1000的hash数据，即二级用户信息块。
在这种方式下，系统只需要遍历一次1000个数据的二级用户信息块，找到对应的一级信息块，再找到最终对应的用户数据就可以了。
