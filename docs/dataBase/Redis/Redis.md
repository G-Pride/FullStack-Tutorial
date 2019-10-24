# Redis 简介

Redis是一个高性能的 `key-value` 内存数据库。

它的三个特点有别于其他竞争对手：

- 支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用。
- 不仅仅支持简单的 `key-value` 类型的数据，同时还提供 `list，set，zset，hash，string` 等数据结构的存储。
- 支持数据的备份，即 `master-slave` 模式的数据备份。

**Redis优点**

- 性能极高，能读的速度是110000次/s，写的速度是81000次/s。
- 支持丰富的数据类型：`list，set，zset，hash，string`。
- 操作都是原子的——所有Redis的操作都是原子，从而确保当两个客户同时访问Redis服务器得到的是更新后的值（最新值）。`原子性（atomicity）：一个事务是一个不可分割的最小工作单位，事务当中的诸多操作，要么都做，要么都不做。Redis所有单个命令的执行都是原子性的，这与它的单线程机制有关。`

# Redis 基本数据类型

Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset（sorted set：有序集合）。

| redis类型  | 含义     |
| ---------- | -------- |
| String     | 字符串   |
| Hash       | 哈希     |
| List       | 列表     |
| Set        | 集合     |
| Sorted set | 有序集合 |

## String 字符串

 string是redis最基本的类型，一个key对应一个value。 

string类型是二进制安全的，意思是redis的string可以包含任何数据，比如jpg图片或者序列化的对象。

string类型是Redis最基本的数据类型，一个键最大能存储512MB。

 **常用的 redis 字符串命令 ：**

**1、SET KEY_NAME VALUE**

设置指定 key 的值

```
172.18.230.13:9001> set name guozh
-> Redirected to slot [5798] located at 172.18.230.14:9003
OK
```

**2、GET KEY_NAME**

获取指定 key 的值

```
172.18.230.14:9003> get name
guozh
```

**3、GETRANGE KEY_NAME start end**

返回 key 中字符串值的子字符，索引从0开始，start和end是闭区间

```
172.18.230.14:9003> GETRANGE name 0 1
gu
172.18.230.14:9003> GETRANGE name 0 -1
guozh
```

**4、GETSET KEY_NAME VALUE**

将给定 key 的值设为 value ，并返回 key 的旧值(old value)

```
172.18.230.14:9003> GETSET name guojihoo
guozh
172.18.230.14:9003> get name
guojihoo
```

**5、 GETBIT KEY_NAME OFFSET** 

对 key 所储存的字符串值，获取指定偏移量上的位(bit)

*在redis中，存储的字符串都是以二级制的形式存在的， 二进制中的每一位就是offset值 ，比如 ”a“ 的 ASCII码是 97 ，转换为二进制是：01100001 ，offset是从左往右计数的 ，即offset 0 = 0，offset 7 = 1。*

```
172.18.230.14:9003> set bitTest a
-> Redirected to slot [15563] located at 172.18.230.15:9005
OK
172.18.230.15:9005> GETBIT bitTest 0
0
172.18.230.15:9005> GETBIT bitTest 7
1
```

**6、MGET KEY1 KEY2 .. KEYN**

 返回所有(一个或多个)给定 key 的值。 如果给定的 key 里面，有某个 key 不存在，那么这个 key 返回特殊值 nil  

```
redis 127.0.0.1:6379> SET key1 "hello"
OK
redis 127.0.0.1:6379> SET key2 "world"
OK
redis 127.0.0.1:6379> MGET key1 key2 someOtherKey
1) "Hello"
2) "World"
3) (nil)
```

**7、Setbit KEY_NAME OFFSET**

对 key 所储存的字符串值，获取指定偏移量上的位(bit)

*现通过 SETBIT  命令将 ”a“ （二进制：01100001）变成 ”b“（二进制：01100010）*

```
172.18.230.14:9003> set danci a
OK
172.18.230.14:9003> get danci
a
172.18.230.14:9003> SETBIT danci 6 1
0
172.18.230.14:9003> SETBIT danci 7 0
1
172.18.230.14:9003> get danci
b
```

**8、SETEX KEY_NAME SECOND VALUE**

 为指定的 key 设置值及其过期时间。如果 key 已经存在， SETEX 命令将会替换旧的值 ，SETEX 必须大写

```
172.18.230.13:9001> SETEX test1 10 test
OK
172.18.230.13:9001> TTL  test1
6
172.18.230.13:9001> get test1
test
172.18.230.13:9001> TTL  test1
-2
172.18.230.13:9001> get test1

```

**9、SETNX KEY_NAME VALUE**

 命令在指定的 key 不存在时，为 key 设置指定的值。

```
172.18.230.13:9001> EXISTS a1
-> Redirected to slot [7785] located at 172.18.230.14:9003
0
172.18.230.14:9003> SETNX a1 aa
1
172.18.230.14:9003> get a1
aa
172.18.230.13:9001> EXISTS sex
1
172.18.230.13:9001> get sex
boy
172.18.230.13:9001> SETNX sex girl
0
172.18.230.13:9001> get sex
boy
```

**10、SETRANGE KEY_NAME OFFSET VALUE**

 指定的字符串覆盖给定 key 所储存的字符串值，覆盖的位置从偏移量 offset 开始 

```
172.18.230.13:9001> SET key1 "Hello World"
-> Redirected to slot [9189] located at 172.18.230.14:9003
OK
172.18.230.14:9003> get key1
Hello World
172.18.230.14:9003> SETRANGE key1 6 "Redis"
11
172.18.230.14:9003> get key1
Hello Redis
```

**11、STRLEN KEY_NAME**

 用于获取指定 key 所储存的字符串值的长度。当 key 储存的不是字符串值时，返回一个错误 

```
172.18.230.13:9001> HSET hashkey key1 value1
-> Redirected to slot [9649] located at 172.18.230.14:9003
1
172.18.230.14:9003> STRLEN hashkey
WRONGTYPE Operation against a key holding the wrong kind of value

172.18.230.14:9003> get sex
-> Redirected to slot [2584] located at 172.18.230.13:9001
boy
172.18.230.13:9001> STRLEN sex
3
```

**12、MSET key1 value1 key2 value2 .. keyN valueN**

 用于同时设置一个或多个 key-value 对 

```
redis 127.0.0.1:6379> MSET key1 "Hello" key2 "World"
OK
redis 127.0.0.1:6379> GET key1
"Hello"
redis 127.0.0.1:6379> GET key2
1) "World"
```

**13、MSETNX key1 value1 key2 value2 .. keyN valueN** 

 用于所有给定 key 都不存在时，同时设置一个或多个 key-value 对 

```
# 对不存在的 key 进行 MSETNX

redis> MSETNX rmdbs "MySQL" nosql "MongoDB" key-value-store "redis"
(integer) 1

redis> MGET rmdbs nosql key-value-store
1) "MySQL"
2) "MongoDB"
3) "redis"


# MSET 的给定 key 当中有已存在的 key

redis> MSETNX rmdbs "Sqlite" language "python"  # rmdbs 键已经存在，操作失败
(integer) 0

redis> EXISTS language                          # 因为 MSET 是原子性操作，language 没有被设置
(integer) 0

redis> GET rmdbs                                # rmdbs 也没有被修改
"MySQL"
```

**14、PSETEX key1 EXPIRY_IN_MILLISECONDS value1** 

 以毫秒为单位设置 key 的生存时间 

```
172.18.230.14:9003> PSETEX key5 30000 values5
OK
172.18.230.14:9003> get key5
values5
172.18.230.14:9003>  pttl key5
20032
172.18.230.14:9003> ttl key5
14
172.18.230.14:9003> ttl key5
-2
172.18.230.14:9003> get key5

```

**15、INCR KEY_NAME** 

将 key 中储存的数字值增一。

如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 INCR 操作。

如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

本操作的值限制在 64 位(bit)有符号数字表示之内

```
172.18.230.14:9003> set key6 valuse6
-> Redirected to slot [4866] located at 172.18.230.13:9001
OK
172.18.230.13:9001> INCR key6
ERR value is not an integer or out of range

172.18.230.13:9001> set key6 6
OK
172.18.230.13:9001> INCR key6 
7
172.18.230.13:9001> get key6
7
```

**16、INCRBY KEY_NAME INCR_AMOUNT**

将 key 中储存的数字加上指定的增量值。

如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 INCRBY 命令。

如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

本操作的值限制在 64 位(bit)有符号数字表示之内

```
# key 存在且是数字值

redis> SET rank 50
OK

redis> INCRBY rank 20
(integer) 70

redis> GET rank
"70"


# key 不存在时

redis> EXISTS counter
(integer) 0

redis> INCRBY counter 30
(integer) 30

redis> GET counter
"30"


# key 不是数字值时

redis> SET book "long long ago..."
OK

redis> INCRBY book 200
(error) ERR value is not an integer or out of range
```

**17、INCRBYFLOAT KEY_NAME INCR_AMOUNT**

为 key 中所储存的值加上指定的浮点数增量值。

如果 key 不存在，那么 INCRBYFLOAT 会先将 key 的值设为 0 ，再执行加法操作

```
# 值和增量都不是指数符号

redis> SET mykey 10.50
OK

redis> INCRBYFLOAT mykey 0.1
"10.6"


# 值和增量都是指数符号

redis> SET mykey 314e-2
OK

redis> GET mykey                # 用 SET 设置的值可以是指数符号
"314e-2"

redis> INCRBYFLOAT mykey 0      # 但执行 INCRBYFLOAT 之后格式会被改成非指数符号
"3.14"


# 可以对整数类型执行

redis> SET mykey 3
OK

redis> INCRBYFLOAT mykey 1.1
"4.1"


# 后跟的 0 会被移除

redis> SET mykey 3.0
OK

redis> GET mykey                                    # SET 设置的值小数部分可以是 0
"3.0"

redis> INCRBYFLOAT mykey 1.000000000000000000000    # 但 INCRBYFLOAT 会将无用的 0 忽略掉，有需要的话，将浮点变为整数
"4"

redis> GET mykey
"4"
```

**18、DECR KEY_NAME** 

将 key 中储存的数字值减一。

如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 DECR 操作。

如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

本操作的值限制在 64 位(bit)有符号数字表示之内

```
# 对存在的数字值 key 进行 DECR

redis> SET failure_times 10
OK

redis> DECR failure_times
(integer) 9


# 对不存在的 key 值进行 DECR

redis> EXISTS count
(integer) 0

redis> DECR count
(integer) -1


# 对存在但不是数值的 key 进行 DECR

redis> SET company YOUR_CODE_SUCKS.LLC
OK

redis> DECR company
(error) ERR value is not an integer or out of range
```

**19、DECRBY KEY_NAME DECREMENT_AMOUNT**

将 key 所储存的值减去指定的减量值。

如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 DECRBY 操作。

如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

本操作的值限制在 64 位(bit)有符号数字表示之内

```
# 对已存在的 key 进行 DECRBY

redis> SET count 100
OK

redis> DECRBY count 20
(integer) 80


# 对不存在的 key 进行DECRBY

redis> EXISTS pages
(integer) 0

redis> DECRBY pages 10
(integer) -10
```

**20、APPEND KEY_NAME NEW_VALUE**

用于为指定的 key 追加值。

如果 key 已经存在并且是一个字符串， APPEND 命令将 value 追加到 key 原来的值的末尾。

如果 key 不存在， APPEND 就简单地将给定 key 设为 value ，就像执行 SET key value 一样

```
# 对不存在的 key 执行 APPEND

redis> EXISTS myphone               # 确保 myphone 不存在
(integer) 0

redis> APPEND myphone "nokia"       # 对不存在的 key 进行 APPEND ，等同于 SET myphone "nokia"
(integer) 5                         # 字符长度


# 对已存在的字符串进行 APPEND

redis> APPEND myphone " - 1110"     # 长度从 5 个字符增加到 12 个字符
(integer) 12

redis> GET myphone
"nokia - 1110"
```

# Redis 订阅发布模式

 Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。 Redis 客户端可以订阅任意数量的频道。 下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2、client5 和 client1 之间的关系： 

![img](./images/04_pub.png)

当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：

![img](./images/04_pub2.png)

**SUBSCRIBE channel [channel ...]**

订阅给指定频道的信息。

**PUBLISH channel message**

将信息 message 发送到指定的频道 channel

## 应用场景

在Redis中，你可以设定对某一个key值进行消息发布及消息订阅，当一个key值上进行了消息发布后，所有订阅它的客户端都会收到相应的消息。这一功能最明显的用法就是用作实时消息系统，比如普通的即时聊天，群聊等功能。

1. 今日头条订阅号、微信订阅公众号、新浪微博关注、邮件订阅系统
2. 即时通信系统（QQ、微信）
3. 群聊部落系统（微信群）

## 案例

微信班级群`class:20170101`，发布订阅模型

**学生 A B C:**

------

**学生C:**

订阅一个`主题`名叫： `chatroom`

```bash
172.18.230.13:9001> SUBSCRIBE chatroom
subscribe
chatroom
1
```

**学生A:**

针对 ``chatroom`` 主题发送 消息，那么所有订阅该主题的用户都能够收到该数据。

```cpp
172.18.230.13:9001> PUBLISH chatroom hi
1
```

**学生B:**

针对 `chatroom`主题发送 消息，那么所有订阅该主题的用户都能够收到该数据。

```cpp
172.18.230.13:9001> PUBLISH chatroom hello
1
```

最后学生C会收到 A 和 B 发送过来的消息。

```cpp
172.18.230.13:9001> SUBSCRIBE chatroom
subscribe
chatroom
1
message
chatroom
hi
message
chatroom
hello
```

# Redis 事务

Redis事务允许一组命令在单一步骤中执行。事务有两个属性，说明如下：

- 事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
- Redis事务是原子的。原子意味着要么所有的命令都执行，要么都不执行；

一个事务从开始到执行会经历以下三个阶段：

- 开始事务
- 命令入队
- 执行事务

```bash
redis 127.0.0.1:6379> MULTI
OK
List of commands here
redis 127.0.0.1:6379> EXEC
```

## 案例

银行转账，邓超给宝强转账1万元：

```bash
127.0.0.1:6379> set dengchao 60000
OK
127.0.0.1:6379> set wangbaoqiang 200
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> INCRBY dengchao -10000
QUEUED
127.0.0.1:6379> INCRBY wangbaoqiang 10000
QUEUED
127.0.0.1:6379> EXEC
1) (integer) 50000
2) (integer) 10200
127.0.0.1:6379>
```

# Redis 使用场景

1.取最新N个数据的操作

比如典型的取你网站的最新文章,我们可以将最新的5000条评论的ID放在Redis的List集合中,并将超出集合部分从数据库获取。

------

2.排行榜应用,取TOP N操作

这个需求与上面需求的不同之处在于,前面操作以时间为权重,这个是以某个条件为权重,比如按顶的次数排序, 这时候就需要我们的sorted set出马了,将你要排序的值设置成sorted set的score, 将具体的数据设置成相应的value,每次只需要执行一条ZADD命令即可。

------

3.需要精准设定过期时间的应用

比如你可以把上面说到的sorted set的score值设置成过期时间的时间戳,那么就可以简单地通过过期时间排序, 定时清除过期数据了,不仅是清除Redis中的过期数据,你完全可以把Redis里这个过期时间当成是对数据库中数据的索引, 用Redis来找出哪些数据需要过期删除,然后再精准地从数据库中删除相应的记录。

------

4.计数器应用

Redis的命令都是原子性的,你可以轻松地利用INCR,DECR命令来构建计数器系统。

------

5.uniq操作,获取某段时间所有数据排重值

这个使用Redis的set数据结构最合适了,只需要不断地将数据往set中扔就行了,set意为集合,所以会自动排重。

------

6.Pub/Sub构建实时消息系统

Redis的Pub/Sub系统可以构建实时的消息系统,比如很多用Pub/Sub构建的实时聊天系统的例子。

------

7.构建队列系统

使用list可以构建队列系统,使用sorted set甚至可以构建有优先级的队列系统。

------

8.缓存

最常用,性能优于Memcached(被libevent拖慢),数据结构更多样化。

------



# Redis 数据备份与恢复

Redis SAVE 命令用于创建当前数据库的备份。

## 语法

redis Save 命令基本语法如下：

```
redis 127.0.0.1:6379> SAVE
```

## 实例

```
redis 127.0.0.1:6379> SAVE 
OK
```

该命令将在 redis 安装目录中创建dump.rdb文件。

## 恢复数据

如果需要恢复数据，只需将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可。获取 redis 目录可以使用 CONFIG 命令，如下所示：

```
redis 127.0.0.1:6379> CONFIG GET dir
1) "dir"
2) "/home/python"
```

以上命令 CONFIG GET dir 输出的 redis 安装目录为 /home/python。

## Bgsave

创建 redis 备份文件也可以使用命令 BGSAVE，该命令在后台执行。

## 实例

```
127.0.0.1:6379> BGSAVE

Background saving started
```

# Redis 配置文件

在启动Redis服务器时,我们需要为其指定一个配置文件,缺省情况下配置文件在Redis的源码目录下,文件名为**redis.conf。**

redis配置文件使用`#######################`被分成了几大块区域,

主要有:

1. 通用(general)
2. 快照(snapshotting)
3. 复制(replication)
4. 安全(security)
5. 限制(limits)
6. 追加模式(append only mode)
7. LUA脚本(lua scripting)
8. REDIS集群(REDIS CLUSTER)
9. 慢日志(slow log)
10. 事件通知(event notification)
11. ADVANCED CONFIG

为了对Redis的系统实现有一个直接的认识,我们首先来看一下Redis的配置文件中定义了哪些主要参数以及这些参数的作用。

------

- daemonize no

默认情况下,redis不是在后台运行的。如果需要在后台运行,把该项的值更改为yes;

------

- pidfile /var/run/redis_6379.pid

当Redis在后台运行的时候,Redis默认会把pid文件放在/var/run/redis.pid, 你可以配置到其他地址。当运行多个redis服务时,需要指定不同的pid文件和端口;

------

- port 6379

指定redis运行的端口,默认是6379;

作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字

------

- bind 127.0.0.1

指定redis只接收来自于该IP地址的请求,如果不进行设置,那么将处理所有请求。在生产环境中最好设置该项;

------

- loglevel notice

指定日志记录级别,

其中Redis总共支持四个级别: debug 、verbose 、notice 、warning, 默认为 notice 。

- debug表示记录很多信息,用于开发和测试。
- verbose表示记录有用的信息, 但不像debug会记录那么多。
- notice表示普通的verbose,常用于生产环境。
- warning 表示只有非常重要或者严重的信息会记录到日志;

------

- logfile 

配置log文件地址,默认值为stdout。若后台模式会输出到/dev/null("黑洞"); （可以自定义配置：/var/log/redis/redis.log）

------

- databases 16

可用数据库数,默认值为16,默认数据库为0,数据库范围在0­ 15之间切换,彼此隔离;

------

- save

保存数据到磁盘,格式为 `save  `,指出在多长时间内,有多少次更新操作, 就将数据同步到数据文件rdb。相当于条件触发抓取快照,这个可以多个条件配合。

保存数据到磁盘:

- save 900 1 #900秒（15分钟）内至少有1个key被改变
- save 300 10 #300秒（5分钟）内至少有10个key被改变
- save 60 10000 #60秒内至少有10000个key被改变

------

- rdbcompression yes

存储至本地数据库时(持久化到rdb文件)是否压缩数据,默认为yes;

如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大

------

- dbfilename dump.rdb

本地持久化数据库文件名,默认值为dump.rdb;

------

- dir ./

工作目录,数据库镜像备份的文件放置的路径。

------

- slaveof

主从复制,设置该数据库为其他数据库的从数据库。设置当本机为slave服务时,设置master服务的IP地址及端口。 在Redis启动时,它会自动从master进行数据同步;

------

- masterauth

￼当master服务设置了密码保护时(用requirepass制定的密码)slave服务连接master的密码;

------

- slave-serve-stale-data yes

当一个slave失去和master的连接，或者同步正在进行中，slave的行为有两种可能：

1) 如果 slave-serve-stale-data 设置为 "yes" (默认值)，slave会继续响应客户端请求，可能是正常数据，也可能是还没获得值的空数据。

2) 如果 slave-serve-stale-data 设置为 "no"，slave会回复"正在从master同步（SYNC with master in progress）"来处理各种请求，除了 INFO 和 SLAVEOF 命令。

------

- repl-ping-slave-period 10

从库会按照一个时间间隔向主库发送PING,可以通过repl-ping-slave-period设置这个时间间隔,默认是10秒;

------

- repl-timeout 60

设置主库批量数据传输时间或者ping回复时间间隔,默认值是60秒,一定要确保repl-timeout大于repl-ping-slave-period ;不然会经常检测到超时。

master检测到slave上次发送的时间超过repl-timeout，即认为slave离线，清除该slave信息。

slave检测到上次和master交互的时间超过repl-timeout，则认为master离线

------

- requirepass foobared

设置客户端连接后进行任何其他指定前需要使用的密码。因为redis速度相当快,所以在一台比较好的服务器下, 一个外部的用户可以在一秒钟进行150K次的密码尝试,这意味着你需要指定非常强大的密码来防止暴力破解;

------

- rename­command CONFIG ""

命令重命名,在一个共享环境下可以重命名相对危险的命令,比如把CONFIG重名为一个不容易猜测的字符: rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52。 如果想删除一个命令,直接把它重命名为一个空字符""即可:rename-command CONFIG "";

------

- maxclients 128

设置最多同时连接客户端数量。

默认没有限制，这个关系到Redis进程能够打开的文件描述符数量。

特殊值"0"表示没有限制。

一旦达到这个限制，Redis会关闭所有新连接并发送错误"达到最大用户数上限（max number of clients reached）"

------

- maxmemory

指定Redis最大内存限制。Redis在启动时会把数据加载到内存中,达到最大内存后,Redis会先尝试清除已到期或即将到期的Key, Redis同时也会移除空的list对象。当此方法处理后,仍然到达最大内存设置,将无法再进行写入操作,但仍然可以进行读取操作。

注意:Redis新的vm机制,会把Key存放内存,Value会存放在swap区;

------

- maxmemory-policy volatile-lru

当内存达到最大值的时候Redis会选择删除哪些数据呢?有五种方式可供选择:

- volatile-lru 代表利用LRU算法移除设置过期时间的key(LRU:最近使用 LeastRecentlyUsed)
- allkeys-lru 代表利用LRU算法移除任何key
- volatile-random 代表移除设置过过期时间的随机key
- allkeys_random 代表移除一个随机的key,
- volatile-ttl 代表移除即将过期的key(minor TTL)
- noeviction 代表不移除任何key,只是返回一个写错误。

注意:对于上面的策略,如果没有合适的key可以移除,写的时候Redis会返回一个错误;

------

- appendonly no

是否开启aof功能

默认情况下,redis会在后台异步的把数据库镜像备份到磁盘,但是该备份是非常耗时的,而且备份也不能很频繁。 如果发生诸如拉闸限电、拔插头等状况,那么将造成比较大范围的数据丢失,所以redis提供了另外一种更加高效的数据库备份及灾难恢复方式。

开启appendonly模式之后,redis会把所接收到的每一次写操作请求都追加到appendonly.aof文件中。

当redis重新启动时,会从该文件恢复出之前的状态

------

- appendfilename appendonly.aof

AOF文件名称,默认为"appendonly.aof";

------

- appendfsync everysec

Redis支持三种同步AOF文件的策略:

- no 代表不进行同步,系统去操作,
- always 代表每次有写操作都进行同步,
- everysec 代表对写操作进行累积,每秒同步一次

默认是"everysec",按照速度和安全折中这是最好的。

------

- slowlog-log-slower-than 10000

记录超过特定执行时间的命令。执行时间不包括I/O计算,比如连接客户端,返回结果等,只是命令执行时间。

可以通过两个参数设置slow log:一个是告诉Redis执行超过多少时间被记录的参数slowlog-log-slower-than(微妙), 另一个是slow log 的长度。当一个新命令被记录的时候最早的命令将被从队列中移除,

下面的时间以微秒(百万分之一秒,1000 * 1000)单位, 因此1000000代表一分钟。

```
1秒=1000毫秒
1秒=1000000微秒
```

注意制定一个负数将关闭慢日志,而设置为0将强制每个命令都会记录;

------

- slowlog-max-len 128

慢操作日志"保留的最大条数

"记录"将会被队列化,如果超过了此长度,旧记录将会被移除。可以通过`SLOWLOG  args`查看慢记录的信息(SLOWLOG get 10,SLOWLOG reset)，通过"SLOWLOG get num"指令可以查看最近num条慢速记录，其中包括"记录"操作的时间/指令/K-V等信息。

------

参考：[https://github.com/linli8/cnblogs/blob/master/redis%E5%89%AF%E6%9C%AC.conf](https://github.com/linli8/cnblogs/blob/master/redis副本.conf)

## 总结

### Select 命令

Redis Select 命令用于切换到指定的数据库，数据库索引号 index 用数字值指定，以 0 作为起始索引值。

```bash
python@ubuntu:/dev$ redis-cli
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> set name 'itcast.cn'
OK
127.0.0.1:6379[1]> select 2
OK
127.0.0.1:6379[2]> set web 'itcast.com'
OK
127.0.0.1:6379[2]> get web
"itcast.com"
127.0.0.1:6379[2]> select 1
OK
127.0.0.1:6379[1]> get name
"itcast.cn"
127.0.0.1:6379[1]>
```

### Shutdown 命令

Redis Shutdown 命令执行以下操作：

- 停止所有客户端
- 如果有至少一个保存点在等待，执行 SAVE 命令
- 如果 AOF 选项被打开，更新 AOF 文件
- 关闭 redis 服务器(server)

```
redis 127.0.0.1:6379> PING
PONG
redis 127.0.0.1:6379> SHUTDOWN
$ redis
```

### Redis 认证

```
redis-cli
AUTH "password"
```

或者

```
redis-cli -h 127.0.0.1 -p 6379 -a myPassword
```

### Redis Showlog

Redis Showlog 是 Redis 用来记录查询执行时间的日志系统。

查询执行时间指的是不包括像客户端响应(talking)、发送回复等 IO 操作，而单单是执行一个查询命令所耗费的时间。

另外，slow log 保存在内存里面，读写速度非常快，因此你可以放心地使用它，不必担心因为开启 slow log 而损害 Redis 的速度。

语法 redis Showlog 命令基本语法如下：

```bash
redis 127.0.0.1:6379> SLOWLOG subcommand [argument]
```

查看日志信息：

```bash
redis 127.0.0.1:6379> slowlog get 2
1) 1) (integer) 14
   2) (integer) 1309448221
   3) (integer) 15
   4) 1) "ping"
2) 1) (integer) 13
   2) (integer) 1309448128
   3) (integer) 30
   4) 1) "slowlog"
      2) "get"
      3) "100"
```

> 每一个Showlog都是由四个字段组成的
>
> 慢日志标识符
>
> 记录的命令进行处理的Unix时间戳
>
> 执行所需的时间，以微秒
>
> 命令、参数。

查看当前日志的数量：

```bash
redis 127.0.0.1:6379> SLOWLOG LEN
(integer) 14
使用命令 SLOWLOG RESET 可以清空 slow log 。

redis 127.0.0.1:6379> SLOWLOG LEN
(integer) 14

redis 127.0.0.1:6379> SLOWLOG RESET
OK

redis 127.0.0.1:6379> SLOWLOG LEN
(integer) 0
```