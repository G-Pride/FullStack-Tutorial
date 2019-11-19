## O(1), O(n), O(logn), O(nlogn) 的区别

![](./images/20180928135003419.png)

## 索引

**为什么需要索引：**快速查询数据。

**什么样的信息能成为索引：**主键、唯一键以及普通键。

**索引的数据结构：**

- 生成索引，建立二叉查找树进行二分查找
- 生成索引，建立B-Tree结构进行查找
- 生成索引，建立B+Tree结构进行查找
- 生成索引，建立Hash结构进行查找

**密集索引和稀疏索引的区别：**

- 密集索引文件中的每个搜索码值都对应一个索引值
- 稀疏索引文件只为索引码的某些值建立索引项

InnoDB：若一个主键被定义，该主键则作为密集索引；若没有主键被定义，该表的第一个唯一非空索引则作为密集索引；若不满足以上条件，innodb内部会生成一个隐藏主键（密集索引）

**索引越多与好？**

- 数据量小的表不需要建立索引，建立会增加额外的索引开销
- 数据变更需要维护索引，因此更多的索引意味着更多的维护成本
- 更多的索引意味着需要更多的空间

## 锁

**MyISAM与InnoDB关于锁方面的区别是什么：**

- MyISAM默认用的是表级锁，不支持行级锁
- InnoDB默认用的是行级锁，也支持表级锁

**MyISAM引擎，MyISAM读操作时会给表加表级的读锁（共享锁），写操作时会给表加表级的写锁（排斥锁）：**

- 当对一张表的1~10行进行读操作时，同时对这张表的10行以外的数据进行写操作时会被block，直到读操作完成为止。

 操作命令: 

```
//加锁
lock table 表名 read

//解锁
unlock tables
```

 实战场景: 

```
clientA: 
    lock table roles write; //写锁
    select * from roles where id = 1; //查询成功
    update roles set name = 'admin' where id = 1; //更新成功

clientB: 
    select * from roles where id = 1; //卡住,等待锁释放
ClientA:
    unlock tables; //解锁
    
clientB: 
    select * from roles where id = 1; //查询成功
```

- 当对一张表的1~10行进行读操作时，同时对这张表的10行以外的数据进行读操作是不会被block。

 操作命令: 

```
//加锁
lock table 表名 read

//解锁
unlock tables
```

 实战场景: 

```
clientA: 
    lock table roles read; //读锁
    select * from roles where id = 1; //查询成功

clientB: 
    select * from roles where id = 1; //查询成功
    update roles set name = 'root'; //卡住,等待锁释放
    
ClientA:
    unlock tables; //解锁
    
clientB: 
    update roles set name = 'root2'; //更新成功
```

**InnoDB引擎，InnoDB 只有通过索引检索数据，才会使用行锁，否则的话使用表锁 :**

- 当对一张表的1~10行进行读操作时，同时对这张表的10行以外的数据进行写操作时不会被block。
- 当对一张表的1~10行进行读操作时，同时对这张表的10行以外的数据进行读操作是不会被block；同时对同一行进行读操作时也不会受影响。

![](.\images\1574178093(1).jpg)

X是增删改、for update操作，S是读操作。

**MyISAM适合的场景：**

- 频繁执行全表count语句
- 没有大量的增删改操作频率不高，查询非常频繁
- 无事务

**InnoDB适合的场景：**

- 数据增删改相当频繁，只是对行进行加锁
- 可靠性要求比较高，要求支持事务

