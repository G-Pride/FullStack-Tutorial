# Redis 5.0.5 

## 安装

步骤：

- 安装源码包

  ```
  yum -y install make gcc*
  ```

- 下载包

  ```
  wget http://download.redis.io/releases/redis-5.0.5.tar.gz
  ```

- 解压安装

  ```
  tar zvxf redis-5.0.5.tar.gz
  cd redis-5.0.5
  make
  ```

  `会报错：`

  `In file included from adlist.c:34:0:`

  `zmalloc.h:50:31: fatal error: jemalloc/jemalloc.h: No such file or directory`

  ```
  make MALLOC=libc
  
  #也可以不加PREFIX，这就默认将程序装在/usr/local/bin/目录下，这样可以在任意地方，而不用到指定目录下调用redis命令，这里我安装到自定义目录下
  make install PREFIX =/home/appuser/redis/redis-cluster 
  ```

  `会报错：`

  `make[1]: *** [install] Error 1`

  `make[1]: Leaving directory /home/appuser/redis/redis-5.0.5/src`

  `make: *** [install] Error 2`

  改用root账号执行make install

## 单机启动	

```
cd src
./redis-server --默认不需要加配置文件
./redis-cli -p 6379 --默认端口是6379
```

## 伪集群启动

- 解压完成后，在redis-cluster下新建集群节点

  ```
  mkdir redis7000
  cd redis7000
  mkdir -p data conf log
  cd ..
  cp -r ./redis7000  redis7001
  cp -r ./redis7000  redis7002
  cp -r ./redis7000  redis7003
  cp -r ./redis7000  redis7004
  cp -r ./redis7000  redis7005
  ```

- 将redis.conf复制到redis7000/conf下，修改配置文件 ：

  ```
  # 关闭保护模式
  protected-mode no
      
  # 以守护进程后台模式运行
  daemonize yes
  
  #它是集群节点自动维护的文件，主要用于记录集群中有哪些节点、他们的状态以及一些持久化参数等，方便在重启时恢复这些状态。通常是在收到请求之后这个文件就会被更新，去掉注释，默认生成nodes-6379.conf
  cluster-config-file nodes-7000.conf
  
  # 绑定本机ip
  bind 49.234.27.151
      
  # 修改端口
  port 7000
      
  # redis进程文件
  pidfile /usr/local/redis_cluster/redis7000/redis_7000.pid
      
  # 日志文件
  logfile /usr/local/redis_cluster/redis7000/log/redis_7000.log
      
  # 快照数据存放目录,一定是目录
  dir /usr/local/redis_cluster/redis7000/data/
      
  # 启用集群
  cluster-enabled yes
  ```

  redis7001~5操作如上，修改对应的参数值。

- 在redis-cluster下启动节点

  ```
  ./bin/redis-server ./redis7000/conf/redis.conf &
  ./bin/redis-server ./redis7001/conf/redis.conf &
  ./bin/redis-server ./redis7002/conf/redis.conf &
  ./bin/redis-server ./redis7003/conf/redis.conf &
  ./bin/redis-server ./redis7004/conf/redis.conf &
  ./bin/redis-server ./redis7005/conf/redis.conf &
  ```

- 启动集群

  ```
  /home/appuser/redis/redis-cluster/bin/redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1
  ```
  
- 进入对应节点的redis（ -c参数来启动集群模式 ，不然操作报错）

  ```
  /home/appuser/redis/redis-cluster/bin/redis-cli -c -p 7000
  ```

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

## HASH 哈希

Redis hash 是一个string类型的field和value的映射表，hash特别适合用于存储对象。

Redis 中每个 hash 可以存储 232 - 1 键值对（40多亿）。

### 示例

1. HDEL KEY_NAME FIELD1.. FIELDN

    用于删除哈希表 key 中的一个或多个指定字段，不存在的字段将被忽略 

   ```
   127.0.0.1:7002> HSET myhash field1 "foo"
   -> Redirected to slot [9295] located at 127.0.0.1:7001
   (integer) 1
   127.0.0.1:7001> HGET myhash field1
   "foo"
   127.0.0.1:7001> HDEL myhash field1
   (integer) 1
   127.0.0.1:7001> HGET myhash field1
   (nil)
   127.0.0.1:7001> HDEL myhash field2
   (integer) 0
   ```

2. HEXISTS KEY_NAME FIELD_NAME

    查看哈希表的指定字段是否存在 

   ```
   127.0.0.1:7001> HEXISTS myhash field1
   (integer) 0
   127.0.0.1:7001> HSET myhash field1 "foo"
   (integer) 1
   127.0.0.1:7001> HEXISTS myhash field1
   (integer) 1
   127.0.0.1:7001> HEXISTS myhash field2
   (integer) 0
   ```

3. HGET KEY_NAME FIELD_NAME

    返回哈希表中指定字段的值 

   ```
   127.0.0.1:7001> HSET myhash field1 "foo"
   (integer) 1
   127.0.0.1:7002> HGET myhash field1
   -> Redirected to slot [9295] located at 127.0.0.1:7001
   "foo"
   127.0.0.1:7001> HGET myhash field2
   (nil)
   ```

4. HGETALL KEY_NAME

   用于返回哈希表中，所有的字段和值。

   在返回值里，紧跟每个字段名(field name)之后是字段的值(value)，所以返回值的长度是哈希表大小的两倍。

   ```
   127.0.0.1:7001> HSET myhash field1 "Hello"
   (integer) 0
   127.0.0.1:7001> HSET myhash field2 "World"
   (integer) 1
   127.0.0.1:7001> HGETALL myhash
   1) "field1"
   2) "Hello"
   3) "field2"
   4) "World"
   ```

5. HINCRBY KEY_NAME FIELD_NAME INCR_BY_NUMBER

   用于为哈希表中的字段值加上指定增量值。增量也可以为负数，相当于对指定字段进行减法操作。如果哈希表的 key 不存在，一个新的哈希表被创建并执行 HINCRBY 命令。如果指定的字段不存在，那么在执行命令前，字段的值被初始化为 0 。对一个储存字符串值的字段执行 HINCRBY 命令将造成一个错误。本操作的值被限制在 64 位(bit)有符号数字表示之内。但是不能浮点数加减。

   ```
   127.0.0.1:7001> HSET myhash field 5
   (integer) 1
   127.0.0.1:7001> HGET myhash field
   "5"
   127.0.0.1:7000> HINCRBY myhash field 1
   -> Redirected to slot [9295] located at 127.0.0.1:7001
   (integer) 6
   127.0.0.1:7001> HGET myhash field
   "6"
   127.0.0.1:7001> HINCRBY myhash field -1
   (integer) 5
   127.0.0.1:7001> HGET myhash field
   "5"
   ##不存在时
   127.0.0.1:7001> HEXISTS myhash field3
   (integer) 0
   127.0.0.1:7001> HINCRBY myhash field3 1
   (integer) 1
   127.0.0.1:7001> HGET myhash field3
   "1"
   ##浮点数操作
   127.0.0.1:7001> HSET myhash field 5
   (integer) 0
   127.0.0.1:7001> HINCRBY myhash field 2.5
   (error) ERR value is not an integer or out of range
   ```

6. HINCRBYFLOAT key field increment

   用于为哈希表中的字段值加上指定浮点数增量值。如果指定的字段不存在，那么在执行命令前，字段的值被初始化为 0 。

   ```
   127.0.0.1:7001> HSET myhash field 5.5
   (integer) 0
   127.0.0.1:7001> HGET myhash field
   "5.5"
   127.0.0.1:7001> HINCRBYFLOAT myhash field 1
   "6.5"
   127.0.0.1:7001> HINCRBYFLOAT myhash field -2
   "4.5"
   127.0.0.1:7001> HINCRBYFLOAT myhash field -0.5
   "4"
   ```

7. HKEYS key

    用于获取哈希表中的所有域（field） 

   ```
   127.0.0.1:7001> HKEYS myhash
   1) "field1"
   2) "field2"
   3) "field"
   4) "field3"
   ```

8. HLEN KEY_NAME 

   用于获取哈希表中字段的数量 

   ```
   127.0.0.1:7001> HLEN myhash
   (integer) 4
   ```

9. HMGET KEY_NAME FIELD1...FIELDN 

   用于返回哈希表中，一个或多个给定字段的值。

   如果指定的字段不存在于哈希表，那么返回一个 nil 值。

   ```
   127.0.0.1:7001> HKEYS myhash
   1) "field1"
   2) "field2"
   3) "field"
   4) "field3"
   127.0.0.1:7001> HLEN myhash
   (integer) 4
   127.0.0.1:7001> HMGET myhash field1 field2
   1) "Hello"
   2) "World"
   127.0.0.1:7001> HMGET myhash field1 field2 field4
   1) "Hello"
   2) "World"
   3) (nil)
   ```

10. HMSET KEY_NAME FIELD1 VALUE1 ...FIELDN VALUEN

    用于同时将多个 field-value (字段-值)对设置到哈希表中。

    此命令会覆盖哈希表中已存在的字段。

    如果哈希表不存在，会创建一个空哈希表，并执行 HMSET 操作。

    ```
    127.0.0.1:7001> HMSET myhash field4 "good" field5 "luck"
    OK
    127.0.0.1:7001> HMGET myhash field4 field5
    1) "good"
    2) "luck"
    ```

11. HSET KEY_NAME FIELD VALUE

    用于为哈希表中的字段赋值 。

    如果哈希表不存在，一个新的哈希表被创建并进行 HSET 操作。

    如果字段已经存在于哈希表中，旧值将被覆盖。

    ```
    127.0.0.1:7001> HSET myhash field1 "foo"
    (integer) 0
    127.0.0.1:7001> HGET myhash field1
    "foo"
    ```

12. HSETNX KEY_NAME FIELD VALUE

    用于为哈希表中不存在的的字段赋值 。

    如果哈希表不存在，一个新的哈希表被创建并进行 HSET 操作。

    如果字段已经存在于哈希表中，操作无效。

    如果 key 不存在，一个新哈希表被创建并执行 HSETNX 命令

    ```
    ##key存在
    127.0.0.1:7001> HEXISTS myhash field
    (integer) 1
    127.0.0.1:7001> HGET myhash field
    "3.5"
    127.0.0.1:7001> HSETNX myhash field 5
    (integer) 0
    127.0.0.1:7001> HGET myhash field
    "3.5"
    ##key不存在
    127.0.0.1:7001> HEXISTS myhash field6
    (integer) 0
    127.0.0.1:7001> HSETNX myhash field6 666
    (integer) 1
    127.0.0.1:7001> HGET myhash field6
    "666"
    ```

13. HVALS KEY_NAME FIELD VALUE

     返回哈希表所有域(field)的值 

    ```
    127.0.0.1:7002> HMSET hashkey field1 good  field2 luck
    -> Redirected to slot [9649] located at 127.0.0.1:7001
    OK
    127.0.0.1:7001> HVALS hashkey
    1) "good"
    2) "luck"
    ```

14.  HSCAN key cursor [MATCH pattern] [COUNT count] 

     迭代哈希表中的键值对 

15.  HSTRLEN KEY_NAME FIELD

     返回 hash指定field的value的字符串长度 

    ```
    127.0.0.1:7001> HGET hashkey field1
    "good"
    127.0.0.1:7001> HSTRLEN hashkey field1
    (integer) 4
    ```

## List 列表

Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素导列表的头部（左边）或者尾部（右边）

一个列表最多可以包含 232 - 1 个元素 (4294967295, 每个列表超过40亿个元素)。

### 示例：

1.  LINDEX KEY_NAME INDEX_POSITION

    用法：用于通过索引获取列表中的元素。你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列			表的倒数第二个元素，以此类推。 

   返回：列表中下标为指定索引值的元素。 如果指定索引值不在列表的区间范围内，返回 nil  
   
   ```
redis 127.0.0.1:6379> LPUSH mylist "World"
   (integer) 1
    
   redis 127.0.0.1:6379> LPUSH mylist "Hello"
   (integer) 2
    
   redis 127.0.0.1:6379> LINDEX mylist 0
   "Hello"
    
   redis 127.0.0.1:6379> LINDEX mylist -1
   "World"
    
   redis 127.0.0.1:6379> LINDEX mylist 3        # index不在 mylist 的区间范围内
   (nil)
   ```
   
2.  RPUSH KEY_NAME VALUE1..VALUEN 

    用法：用于将一个或多个值插入到列表的尾部(最右边)。如果列表不存在，一个空列表会被创建并执行 RPUSH 			操作。 当列表存在但不是列表类型时，返回一个错误。

    返回： 执行 RPUSH 操作后，列表的长度。 

    ```
    127.0.0.1:7000> RPUSH mylist1 my
    (integer) 1
    127.0.0.1:7000> RPUSH mylist1 name
    (integer) 2
    ```

3.  LRANGE KEY_NAME START END 

    用法： 返回列表中指定区间内的元素，区间以偏移量 START 和 END 指定。 其中 0 表示列表的第一个元素， 			1 表示列表的第二个元素，以此类推。 你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表			示列表的倒数第二个元素，以此类推。 

    返回： 一个列表，包含指定区间内的元素。 

    ```
    127.0.0.1:7000> RPUSH mylist my
    (integer) 3
    127.0.0.1:7000> RPUSH mylist1 my
    (integer) 1
    127.0.0.1:7000> RPUSH mylist1 name
    (integer) 2
    127.0.0.1:7000> LRANGE mylist1 0 -1
    1) "my"
    2) "name"
    127.0.0.1:7000> LRANGE mylist1 0 0
    1) "my"
    ```

4.  RPOPLPUSH SOURCE_KEY_NAME DESTINATION_KEY_NAME 

    用法： 用于移除列表的最后一个元素，并将该元素添加到另一个列表并返回。 

    返回： 被弹出的元素。 

    ```
    redis 127.0.0.1:6379> RPUSH mylist "hello"
    (integer) 1
    redis 127.0.0.1:6379> RPUSH mylist "foo"
    (integer) 2
    redis 127.0.0.1:6379> RPUSH mylist "bar"
    (integer) 3
    redis 127.0.0.1:6379> RPOPLPUSH mylist myotherlist
    "bar"
    redis 127.0.0.1:6379> LRANGE mylist 0 -1
    1) "hello"
    2) "foo"
    ```

5.  BLPOP LIST1 LIST2 .. LISTN TIMEOUT 

    用法： 移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 

    返回： 如果列表为空，返回一个 nil 。 否则，返回一个含有两个元素的列表，第一个元素是被弹出元素所属			的 key ，第二个元素是被弹出元素的值。 

    ```
    127.0.0.1:7000> LRANGE mylist1 0 -1
    1) "my"
    2) "name"
    127.0.0.1:7000> BLPOP mylist1 10
    1) "mylist1"
    2) "my"
    127.0.0.1:7000> LRANGE mylist1 0 -1
    1) "name"
    ```

6.  BRPOP LIST1 LIST2 .. LISTN TIMEOUT 

     用法：移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为			止。

    返回： 假如在指定时间内没有任何元素被弹出，则返回一个 nil 和等待时长。 反之，返回一个含有两个元素			的列表，第一个元素是被弹出元素所属的 key ，第二个元素是被弹出元素的值。  

    ```
    127.0.0.1:7000> LPUSH mylist1 her
    (integer) 2
    127.0.0.1:7000> LRANGE mylist1 0 -1
    1) "her"
    2) "name"
    127.0.0.1:7000> BRPOP mylist1 10
    1) "mylist1"
    2) "name"
    127.0.0.1:7000> LRANGE mylist1 0 -1
    1) "her"
    ```

7.  BRPOPLPUSH LIST1 ANOTHER_LIST TIMEOUT 

    用法：从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表			直到等待超时或发现可弹出元素为止。

    返回： 假如在指定时间内没有任何元素被弹出，则返回一个 nil 和等待时长。 反之，返回一个含有两个元素			的列表，第一个元素是被弹出元素的值，第二个元素是等待时长。 

    ```
    #非空列表
    redis 127.0.0.1:6379> BRPOPLPUSH msg reciver 500
    "hello moto"                        # 弹出元素的值
    (3.38s)                             # 等待时长
     
    redis 127.0.0.1:6379> LLEN reciver
    (integer) 1
     
    redis 127.0.0.1:6379> LRANGE reciver 0 0
    1) "hello moto"
     
     
    # 空列表
     
    redis 127.0.0.1:6379> BRPOPLPUSH msg reciver 1
    (nil)
    (1.34s)
    <pre>
    (nil)
    (100.06s)
    ```
    
8.  LREM KEY_NAME COUNT VALUE 

    用法：根据参数 COUNT 的值，移除列表中与参数 VALUE 相等的元素。

    ​			COUNT 的值可以是以下几种：

    ​			count > 0 : 从表头开始向表尾搜索，移除与 VALUE 相等的元素，数量为 COUNT 。

    ​			count < 0 : 从表尾开始向表头搜索，移除与 VALUE 相等的元素，数量为 COUNT 的绝对值。

    ​			count = 0 : 移除表中所有与 VALUE 相等的值。

    返回： 被移除元素的数量。 列表不存在时返回 0 。 

    ```
    127.0.0.1:7000> LRANGE mylist1 0 -1
    1) "18"
    2) "is"
    3) "id"
    4) "age"
    5) "her"
    127.0.0.1:7000> LREM mylist1 1 id
    (integer) 1
    127.0.0.1:7000> LRANGE mylist1 0 -1
    1) "18"
    2) "is"
    3) "age"
    4) "her"
    127.0.0.1:7000> LREM mylist1 2 id
    (integer) 0
    127.0.0.1:7000> LRANGE mylist1 0 -1
    1) "18"
    2) "is"
    3) "age"
    4) "her
    ```

9.  LLEN KEY_NAME 

    用法： 用于返回列表的长度。 如果列表 key 不存在，则 key 被解释为一个空列表，返回 0 。 如果 key 不是列			表类型，返回一个错误。 

    返回： 列表的长度。 

    ```
    127.0.0.1:7000> LRANGE mylist1 0 -1
    1) "18"
    2) "is"
    3) "age"
    4) "her"
    127.0.0.1:7000> LLEN mylist1
    (integer) 4
    ```

10.  LTRIM KEY_NAME START STOP 

    用法：对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将			被删除。下标 0 表示列表的第一个元素，以 1 表示列表的第二个元素，以此类推。 你也可以使用负数			下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。

    返回： 命令执行成功时，返回 ok 。 

    ```
    redis 127.0.0.1:6379> RPUSH mylist "hello"
    (integer) 1
    redis 127.0.0.1:6379> RPUSH mylist "hello"
    (integer) 2
    redis 127.0.0.1:6379> RPUSH mylist "foo"
    (integer) 3
    redis 127.0.0.1:6379> RPUSH mylist "bar"
    (integer) 4
    redis 127.0.0.1:6379> LTRIM mylist 1 -1
    OK
    redis 127.0.0.1:6379> LRANGE mylist 0 -1
    1) "hello"
    2) "foo"
    3) "bar"
    ```

11.  Lpop   KEY_NAME 

      用于移除并返回列表的第一个元素。 

     ```
     127.0.0.1:7000> LRANGE mylist1 0 -1
     1) "18"
     2) "is"
     3) "age"
     4) "her"
     127.0.0.1:7000> LPOP mylist1
     "18"
     127.0.0.1:7000> LRANGE mylist1 0 -1
     1) "is"
     2) "age"
     3) "her"
     ```

12.  LPUSHX KEY_NAME VALUE1.. VALUEN 

      用法：将一个或多个值插入到已存在的列表头部，列表不存在时操作无效。 

     返回： 列表的长度 

     ```
     127.0.0.1:7000> LRANGE mylist1 0 -1
     1) "18"
     2) "is"
     3) "age"
     4) "her"
     127.0.0.1:7000> LPUSHX mylist1 !
     (integer) 5
     127.0.0.1:7000> LRANGE mylist1 0 -1
     1) "!"
     2) "18"
     3) "is"
     4) "age"
     5) "her"
     127.0.0.1:7000> LPUSHX mylist2 hello
     -> Redirected to slot [16360] located at 127.0.0.1:7002
     (integer) 0
     127.0.0.1:7002> LRANGE mylist2 0 -1
     (empty list or set)
     ```

13.  LINSERT KEY_NAME BEFORE EXISTING_VALUE NEW_VALUE 

     用法：用于在列表的元素前或者后插入元素。 当指定元素不存在于列表中时，不执行任何操作。 当列表不存			在时，   被视为空列表，不执行任何操作。 如果 key 不是列表类型，返回一个错误。 

     返回： 如果命令执行成功，返回插入操作完成之后，列表的长度。 如果没有找到指定元素 ，返回 -1 。 如果 			key 不存在或为空列表，返回 0 。 

     ```
     127.0.0.1:7002> LINSERT mylist1 before her my 
     -> Redirected to slot [3979] located at 127.0.0.1:7000
     (integer) 6
     127.0.0.1:7000> LRANGE mylist1 0 -1
     1) "!"
     2) "18"
     3) "is"
     4) "age"
     5) "my"
     6) "her"
     127.0.0.1:7000> LINSERT mylist1 after her he 
     (integer) 7
     127.0.0.1:7000> LRANGE mylist1 0 -1
     1) "!"
     2) "18"
     3) "is"
     4) "age"
     5) "my"
     6) "her"
     7) "he"
     127.0.0.1:7000> LINSERT mylist1 after name guozh
     (integer) -1
     127.0.0.1:7000> LINSERT mylist2 after her he 
     -> Redirected to slot [16360] located at 127.0.0.1:7002
     (integer) 0
     ```

14.  RPOP KEY_NAME  

      移除并返回列表的最后一个元素 

     ```
     127.0.0.1:7002> LRANGE mylist1 0 -1
     -> Redirected to slot [3979] located at 127.0.0.1:7000
     1) "!"
     2) "18"
     3) "is"
     4) "age"
     5) "my"
     6) "her"
     7) "he"
     127.0.0.1:7000> RPOP mylist1
     "he"
     127.0.0.1:7000> LRANGE mylist1 0 -1
     1) "!"
     2) "18"
     3) "is"
     4) "age"
     5) "my"
     6) "her"
     ```

15.  LSET KEY_NAME INDEX VALUE 

     用法：通过索引来设置元素的值。当索引参数超出范围，或对一个空列表进行 LSET 时，返回一个错误。

     返回： 操作成功返回 ok ，否则返回错误信息。 

     ```
     127.0.0.1:7000> LRANGE mylist1 0 -1
     1) "18"
     2) "is"
     3) "age"
     4) "my"
     5) "her"
     127.0.0.1:7000> lset mylist1 0 28
     OK
     127.0.0.1:7000> LRANGE mylist1 0 -1
     1) "28"
     2) "is"
     3) "age"
     4) "my"
     5) "her"
     ```

16.  LPUSH KEY_NAME VALUE1.. VALUEN 

      用法：将一个或多个值插入到列表头部。 如果 key 不存在，一个空列表会被创建并执行 LPUSH 操作。 当 			key 存在但不是列表类型时，返回一个错误。 

     返回： 列表的长度 

     ```
     redis 127.0.0.1:6379> LPUSH list1 "foo"
     (integer) 1
     redis 127.0.0.1:6379> LPUSH list1 "bar"
     (integer) 2
     redis 127.0.0.1:6379> LRANGE list1 0 -1
     1) "foo"
     2) "bar
     ```

17.  RPUSHX KEY_NAME VALUE1..VALUEN 

     用法： 将一个或多个值插入到已存在的列表尾部(最右边)。如果列表不存在，操作无效。 

     返回： 列表的长度 

     ```
     127.0.0.1:7002> LRANGE mylist3 0 -1
     1) "my"
     2) "name"
     3) "is"
     4) "guozh"
     127.0.0.1:7002> RPUSHX mylist3 !
     (integer) 5
     127.0.0.1:7002> LRANGE mylist3 0 -1
     1) "my"
     2) "name"
     3) "is"
     4) "guozh"
     5) "!"
     127.0.0.1:7002> RPUSHX mylist4 hello
     -> Redirected to slot [7982] located at 127.0.0.1:7001
     (integer) 0
     ```

### 应用场景:

1.取最新N个数据的操作

比如典型的取你网站的最新文章，通过下面方式，我们可以将最新的5000条评论的ID放在Redis的List集合中，并将超出集合部分从数据库获取

- 使用LPUSH latest.comments命令，向list集合中插入数据

- 插入完成后再用LTRIM latest.comments 0 5000命令使其永远只保存最近5000个ID

- 然后我们在客户端获取某一页评论时可以用下面的逻辑（伪代码）

  ```java
  FUNCTION get_latest_comments(start,num_items):
    id_list = redis.lrange("latest.comments",start,start+num_items-1)
    IF id_list.length < num_items
        id_list = SQL_DB("SELECT ... ORDER BY time LIMIT ...")
    END
    RETURN id_list
  ```

  如果你还有不同的筛选维度，比如某个分类的最新N条，那么你可以再建一个按此分类的List，只存ID的话，Redis是非常高效的。

#### 示例

取最新N个评论的操作

```bash
127.0.0.1:6379> lpush mycomment 100001
(integer) 1
127.0.0.1:6379> lpush mycomment 100002
(integer) 2
127.0.0.1:6379> lpush mycomment 100003
(integer) 3
127.0.0.1:6379> lpush mycomment 100004
(integer) 4
127.0.0.1:6379> LRANGE mycomment 0 -1
1) "100004"
2) "100003"
3) "100002"
4) "100001"
127.0.0.1:6379> LTRIM mycomment 0 1
OK
127.0.0.1:6379> LRANGE mycomment 0 -1
1) "100004"
2) "100003"
127.0.0.1:6379> lpush mycomment 100005
(integer) 3
127.0.0.1:6379> LRANGE mycomment 0 -1
1) "100005"
2) "100004"
3) "100003"
127.0.0.1:6379> LTRIM mycomment 0 1
OK
127.0.0.1:6379> LRANGE mycomment 0 -1
1) "100005"
2) "100004"
127.0.0.1:6379>
```

## Set 集合

Set 就是一个集合,集合的概念就是一堆不重复值、无序的组合。利用 Redis 提供的 Set 数据结构,可以存储一些集合性的数据。

> 比如在 微博应用中,可以将一个用户所有的关注人存在一个集合中,将其所有粉丝存在一个集合。

因为 Redis 非常人性化的为集合提供了 求交集、并集、差集等操作, 那么就可以非常方便的实现如共同关注、共同喜好、二度好友等功能, 对上面的所有集合操作,你还可以使用不同的命令选择将结果返回给客户端还是存集到一个新的集合中。

1.  SADD KEY_NAME VALUE1..VALUEN 

   用法：将一个或多个成员元素加入到集合中，已经存在于集合的成员元素将被忽略。假如集合 key 不存在，则创建一个只包含添加的元素作成员的集合。当集合 key 不是集合类型时，返回一个错误。

   返回： 被添加到集合中的新元素的数量，不包括被忽略的元素。 

   ```
   127.0.0.1:7001> SADD myset hello
   -> Redirected to slot [560] located at 127.0.0.1:7000
   (integer) 1
   127.0.0.1:7000> SADD myset hello redis set
   (integer) 2
   ```

2.  SCARD KEY_NAME

   用法： 返回集合中元素的数量 。

   返回： 集合的数量。 当集合 key 不存在时，返回 0 。 

   ```
   127.0.0.1:7001> SADD myset hello
   -> Redirected to slot [560] located at 127.0.0.1:7000
   (integer) 1
   127.0.0.1:7000> SADD myset hello redis set
   (integer) 2
   127.0.0.1:7000> SCARD myset
   (integer) 3
   ```

3.  SDIFF FIRST_KEY OTHER_KEY1..OTHER_KEYN 

    用法：返回给定集合之间的差集。不存在的集合 key 将视为空集。 

   返回： 包含差集成员的列表。 

   ```
   127.0.0.1:7000> SADD {myset} hello redis
   (integer) 2
   127.0.0.1:7000> SADD {myset}2 hello world
   (integer) 2
   127.0.0.1:7000> SDIFF {myset} {myset}2
   1) "redis"
   ```

4.  SDIFFSTORE DESTINATION_KEY KEY1..KEYN  

   用法： 将给定集合之间的差集存储在指定的集合中。如果指定的集合 key 已存在，则会被覆盖。 

   返回： 结果集中的元素数量。 

   ```
   127.0.0.1:7000> SADD {myset} hello redis
   (integer) 2
   127.0.0.1:7000> SADD {myset}2 hello world
   (integer) 2
   127.0.0.1:7000> SDIFFSTORE {myset}3 {myset} {myset}2
   (integer) 1
   127.0.0.1:7000> SMEMBERS {myset}3
   1) "redis"
   ```

5.  SINTER KEY KEY1..KEYN 

   用法：返回给定所有给定集合的交集。 不存在的集合 key 被视为空集。 当给定集合当中有一个空集时，结果也为空集(根据集合运算定律)。 

   返回： 交集成员的列表。 

   ```
   127.0.0.1:7000> SADD {myset} hello redis
   (integer) 2
   127.0.0.1:7000> SADD {myset}2 hello world
   (integer) 2
   127.0.0.1:7000> SINTER {myset} {myset}2
   1) "hello"
   ```

6.  SINTERSTORE DESTINATION_KEY KEY KEY1..KEYN

   用法： 将给定集合之间的交集存储在指定的集合中。如果指定的集合已经存在，则将其覆盖。 

   返回： 交集成员的列表。 

   ```
   127.0.0.1:7000> SADD {myset} hello redis
   (integer) 2
   127.0.0.1:7000> SADD {myset}2 hello world
   (integer) 2
   127.0.0.1:7000> SINTERSTORE {myset}3 {myset} {myset}2
   (integer) 1
   127.0.0.1:7000> SMEMBERS {myset}3
   1) "hello"
   ```

7.  SISMEMBER KEY VALUE 

   用法： 判断成员元素是否是集合的成员。 

   返回： 如果成员元素是集合的成员，返回 1 。 如果成员元素不是集合的成员，或 key 不存在，返回 0 。 

   ```
   127.0.0.1:7000> SADD myset4 hello redis and world
   -> Redirected to slot [16345] located at 127.0.0.1:7002
   (integer) 4
   127.0.0.1:7002> SISMEMBER  myset4 hello
   (integer) 1
   127.0.0.1:7002> SISMEMBER  myset4 hi
   (integer) 0
   ```

8.  SMEMBERS KEY VALUE 

   用法： 返回集合中的所有的成员。 不存在的集合 key 被视为空集合。 

   返回：  集合中的所有成员。 

   ```
   127.0.0.1:7000> SADD myset4 hello redis and world
   -> Redirected to slot [16345] located at 127.0.0.1:7002
   (integer) 4
   127.0.0.1:7002> SMEMBERS myset4
   1) "hello"
   2) "world"
   3) "redis"
   4) "and"
   ```

9.  SMOVE SOURCE DESTINATION MEMBER 

   用法： 将指定成员 member 元素从 source 集合移动到 destination 集合。 

   返回： 如果成员元素被成功移除，返回 1 。 如果成员元素不是 source 集合的成员，并且没有任何操作对 destination 集合执行，那么返回 0 。 

   ```
   ##成员元素是 source 集合的成员
   127.0.0.1:7000> SMEMBERS {myset}4
   1) "world"
   2) "hello"
   127.0.0.1:7000> SMEMBERS {myset}5
   1) "redis"
   2) "hello"
   127.0.0.1:7000> SMOVE {myset}4 {myset}5 world
   (integer) 1
   127.0.0.1:7000> SMEMBERS {myset}4
   1) "hello"
   127.0.0.1:7000> SMEMBERS {myset}5
   1) "world"
   2) "redis"
   3) "hello"
   ##成员元素不是 source 集合的成员
   127.0.0.1:7000> SMOVE {myset}4 {myset}5 redis
   (integer) 0
   ```

10.  SPOP KEY

     用法：用于移除并返回集合中的一个随机元素。

    返回： 被移除的随机元素。 当集合不存在或是空集时，返回 nil 。 

    ```
    127.0.0.1:7000> SMEMBERS {myset}5
     1) "3"
     2) "1"
     3) "7"
     4) "world"
     5) "2"
     6) "6"
     7) "4"
     8) "8"
     9) "5"
    10) "9"
    127.0.0.1:7000> SPOP {myset}5
    "1"
    127.0.0.1:7000>  SMEMBERS {myset}5
    1) "7"
    2) "world"
    3) "2"
    4) "6"
    5) "4"
    6) "8"
    7) "5"
    8) "3"
    9) "9"
    127.0.0.1:7000> SPOP {myset}5 2
    1) "5"
    2) "9"
    127.0.0.1:7000>  SMEMBERS {myset}5 
    1) "world"
    2) "4"
    3) "6"
    4) "8"
    5) "3"
    6) "7"
    7) "2"
    ```

11.  SRANDMEMBER KEY [count] 

    用法：

    - 如果 count 为正数，且小于集合基数，那么命令返回一个包含 count 个元素的数组，数组中的元素各不相同。如果 count 大于等于集合基数，那么返回整个集合。
    - 如果 count 为负数，那么命令返回一个数组，数组中的元素可能会重复出现多次，而数组的长度为 count 的绝对值。

    该操作和 SPOP 相似，但 SPOP 将随机元素从集合中移除并返回，而 Srandmember 则仅仅返回随机元素，而不对集合进行任何改动。

    返回： 只提供集合 key 参数时，返回一个元素；如果集合为空，返回 nil 。 如果提供了 count 参数，那么返			回一个数组；如果集合为空，返回空数组。 

    ```
    127.0.0.1:7000> SRANDMEMBER {myset}5 2
    1) "2"
    2) "3"
    127.0.0.1:7000> SRANDMEMBER {myset}5 8
    1) "8"
    2) "3"
    3) "7"
    4) "world"
    5) "2"
    6) "4"
    7) "6"
    127.0.0.1:7000> SRANDMEMBER {myset}5 -3
    1) "8"
    2) "8"
    3) "3"
    ```

12.  SREM KEY MEMBER1..MEMBERN 

    用法：用于移除集合中的一个或多个成员元素，不存在的成员元素会被忽略。当 key 不是集合类型，返回一个			错误。

    返回： 被成功移除的元素的数量，不包括被忽略的元素。 

    ```
    127.0.0.1:7000> SMEMBERS {myset}5
    1) "world"
    2) "4"
    3) "6"
    4) "8"
    5) "3"
    6) "7"
    7) "2"
    127.0.0.1:7000> SREM {myset}5 2 7
    (integer) 2
    127.0.0.1:7000> SMEMBERS {myset}5
    1) "world"
    2) "4"
    3) "6"
    4) "8"
    5) "3"
    127.0.0.1:7000> SREM {myset}5 1
    (integer) 0
    ```

13.  SUNION KEY KEY1..KEYN 

    用法：返回给定集合的并集。不存在的集合 key 被视为空集。 

    返回： 并集成员的列表。 

    ```
    127.0.0.1:7000> SADD {myset}6 1 2 3
    (integer) 3
    127.0.0.1:7000> SADD {myset}7 3 4 5
    (integer) 3
    127.0.0.1:7000> SUNION {myset}6 {myset}7
    1) "1"
    2) "2"
    3) "3"
    4) "4"
    5) "5"
    ```

14.  SUNIONSTORE DESTINATION KEY KEY1..KEYN 

    用法： 将给定集合的并集存储在指定的集合 destination 中。 

    返回： 结果集中的元素数量。 

    ```
    127.0.0.1:7000> SUNION {myset}6 {myset}7
    1) "1"
    2) "2"
    3) "3"
    4) "4"
    5) "5"
    127.0.0.1:7000> SUNIONSTORE {myset}8 {myset}6 {myset}7
    (integer) 5
    127.0.0.1:7000> SMEMBERS {myset}8
    1) "1"
    2) "2"
    3) "3"
    4) "4"
    5) "5"
    ```

15.  SSCAN KEY [MATCH pattern] [COUNT count] 

    用法： 迭代集合键中的元素。 

    返回：  数组列表。 

    ```
    redis 127.0.0.1:6379> SADD myset1 "hello"
    (integer) 1
    redis 127.0.0.1:6379> SADD myset1 "hi"
    (integer) 1
    redis 127.0.0.1:6379> SADD myset1 "bar"
    (integer) 1
    redis 127.0.0.1:6379> sscan myset1 0 match h*
    1) "0"
    2) 1) "hello"
       2) "h1"
    ```

## Sorted set 有序集合

Redis 有序集合和集合一样也是string类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

有序集合的成员是唯一的,但分数(score)却可以重复。

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。 集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。

### 示例

1.  ZADD KEY_NAME SCORE1 VALUE1.. SCOREN VALUEN 

   用法：用于将一个或多个成员元素及其分数值加入到有序集当中。如果某个成员已经是有序集的成员，那么更			新这个成员的分数值，并通过重新插入这个成员元素，来保证该成员在正确的位置上。分数值可以是整			数值或双精度浮点数。如果有序集合 key 不存在，则创建一个空的有序集并执行 ZADD 操作。当 key 			存在但不是有序集类型时，返回一个错误。

   返回： 被成功添加的新成员的数量，不包括那些被更新的、已经存在的成员。 

   ```
   127.0.0.1:7000> ZADD myzset 1 value 2 value2
   (integer) 2
   127.0.0.1:7000> ZRANGE myzset 0 -1 WITHSCORES
   1) "value"
   2) "1"
   3) "value2"
   4) "2"
   127.0.0.1:7000> ZADD myzset 2 value
   (integer) 0
   127.0.0.1:7000> ZRANGE myzset 0 -1 WITHSCORES
   1) "value"
   2) "2"
   3) "value2"
   4) "2"
   ```

2.  ZCARD KEY_NAME 

   用法： 用于计算集合中元素的数量。 

   返回： 当 key 存在且是有序集类型时，返回有序集的基数。 当 key 不存在时，返回 0 。 

   ```
   127.0.0.1:7000> ZRANGE myzset 0 -1 WITHSCORES
   1) "value"
   2) "2"
   3) "value2"
   4) "2"
   127.0.0.1:7000> ZCARD myzset
   (integer) 2
   ```

3.  ZCOUNT key min max 

   用法： 用于计算有序集合中指定分数区间（闭区间）的成员数量。 

   返回： 分数值在 min 和 max 之间的成员的数量。 

   ```
   127.0.0.1:7000> ZRANGE myzset 0 -1 WITHSCORES
   1) "value3"
   2) "1"
   3) "value"
   4) "2"
   5) "value2"
   6) "2"
   127.0.0.1:7000> ZCOUNT myzset 1 2
   (integer) 3
   ```

4.  ZINCRBY key increment member 

   用法：对有序集合中指定成员的分数加上增量 increment可以通过传递一个负数值 increment ，让分数减去			相应的值，比如 ZINCRBY key -5 member ，就是让 member 的 score 值减去 5 。当 key 不存在，或			分数不是 key 的成员时， ZINCRBY key increment member 等同于 ZADD key increment member 。			当 key 不是有序集类型时，返回一个错误。分数值可以是整数值或双精度浮点数。

   返回： 成员的新分数值，以字符串形式表示。 

   ```
   127.0.0.1:7000> ZRANGE myzset 0 -1 WITHSCORES
   1) "value3"
   2) "1"
   3) "value"
   4) "2"
   5) "value2"
   6) "2"
   127.0.0.1:7000> ZINCRBY myzset 1 value
   "3"
   127.0.0.1:7000> ZRANGE myzset 0 -1 WITHSCORES
   1) "value3"
   2) "1"
   3) "value2"
   4) "2"
   5) "value"
   6) "3"
   127.0.0.1:7000> ZINCRBY myzset -1 value2
   "1"
   127.0.0.1:7000> ZRANGE myzset 0 -1 WITHSCORES
   1) "value2"
   2) "1"
   3) "value3"
   4) "1"
   5) "value"
   6) "3"
   ```

5.  ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX] 

   用法：计算给定的一个或多个有序集的交集，其中给定 key 的数量必须以 numkeys 参数指定，并将该交集(结果集)储存到 destination 。默认情况下，结果集中某个成员的分数值是所有给定集下该成员分数值之和。

   返回： 保存到目标结果集的的成员数量。 

   ```
   127.0.0.1:7000> ZRANGE {myzset}1 0 -1 WITHSCORES
   1) "redis"
   2) "1"
   3) "mysql"
   4) "2"
   5) "oracle"
   6) "3"
   127.0.0.1:7000> ZRANGE {myzset}2 0 -1 WITHSCORES
   1) "redis"
   2) "2"
   3) "mysql"
   4) "3"
   5) "oracle"
   6) "4"
   127.0.0.1:7000> ZINTERSTORE {myzset}3 2 {myzset}2 {myzset}1
   (integer) 3
   127.0.0.1:7000> ZRANGE {myzset}3 0 -1 WITHSCORES
   1) "redis"
   2) "3"
   3) "mysql"
   4) "5"
   5) "oracle"
   6) "7"
   ```

6.  ZLEXCOUNT KEY MIN MAX 

    用法：在计算有序集合中指定字典区间内成员数量。 

    返回：指定区间内的成员数量。 

   ```
   redis 127.0.0.1:6379> ZADD myzset 0 a 0 b 0 c 0 d 0 e
   (integer) 5
   redis 127.0.0.1:6379> ZADD myzset 0 f 0 g
   (integer) 2
   redis 127.0.0.1:6379> ZLEXCOUNT myzset - +
   (integer) 7
   redis 127.0.0.1:6379> ZLEXCOUNT myzset [b [f
   (integer) 5
   ```

7.  ZRANGE key start stop [WITHSCORES] 

   用法：返回有序集中，指定区间内的成员。其中成员的位置按分数值递增(从小到大)来排序。具有相同分数值			的成员按字典序(lexicographical order )来排列。如果你需要成员按值递减(从大到小)来排列，请使用 			[ZREVRANGE](https://www.redis.net.cn/order/8748.html) 命令。下标参数 start 和 stop 都以 0 为底，也就是说，以 0 表示有序集第一个成员，以 			1 表示有序集第二个成员，以此类推。你也可以使用负数下标，以 -1 表示最后一个成员， -2 表示倒数			第二个成员，以此类推。

   返回： 指定区间内，带有分数值(可选)的有序集成员的列表。 

   ```
   redis 127.0.0.1:6379> ZRANGE salary 0 -1 WITHSCORES             # 显示整个有序集成员
   1) "jack"
   2) "3500"
   3) "tom"
   4) "5000"
   5) "boss"
   6) "10086"
    
   redis 127.0.0.1:6379> ZRANGE salary 1 2 WITHSCORES              # 显示有序集下标区间 1 至 2 的成员
   1) "tom"
   2) "5000"
   3) "boss"
   4) "10086"
    
   redis 127.0.0.1:6379> ZRANGE salary 0 200000 WITHSCORES         # 测试 end 下标超出最大下标时的情况
   1) "jack"
   2) "3500"
   3) "tom"
   4) "5000"
   5) "boss"
   6) "10086"
    
   redis > ZRANGE salary 200000 3000000 WITHSCORES                  # 测试当给定区间不存在于有序集时的情况
   (empty list or set)
   ```

8.  ZRANGEBYLEX key min max [LIMIT offset count] 

    用法：通过字典区间返回有序集合的成员。

   返回： 指定区间内的元素列表。 

   ```
   redis 127.0.0.1:6379> ZADD myzset 0 a 0 b 0 c 0 d 0 e 0 f 0 g
   (integer) 7
   redis 127.0.0.1:6379> ZRANGEBYLEX myzset - [c
   1) "a"
   2) "b"
   3) "c"
   redis 127.0.0.1:6379> ZRANGEBYLEX myzset - (c
   1) "a"
   2) "b"
   redis 127.0.0.1:6379> ZRANGEBYLEX myzset [aaa (g
   1) "b"
   2) "c"
   3) "d"
   4) "e"
   5) "f"
   ```

9.  ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count] 

   用法：返回有序集合中指定分数区间的成员列表。有序集成员按分数值递增(从小到大)次序排列。

   具有相同分数值的成员按字典序来排列(该属性是有序集提供的，不需要额外的计算)。

   默认情况下，区间的取值使用闭区间 (小于等于或大于等于)，你也可以通过给参数前增加 ( 符号来使用可选的开区间 (小于或大于)。

   举个例子：

   ```
   ZRANGEBYSCORE zset (1 5
   ```

   返回所有符合条件 1 `< score <= 5 的成员，而`

   ```
   ZRANGEBYSCORE zset (5 (10则返回所有符合条件 5 < score < 10 的成员。
   ```

   返回： 指定区间内，带有分数值(可选)的有序集成员的列表。 

   ```
   redis 127.0.0.1:6379> ZADD salary 2500 jack                        # 测试数据
   (integer) 0
   redis 127.0.0.1:6379> ZADD salary 5000 tom
   (integer) 0
   redis 127.0.0.1:6379> ZADD salary 12000 peter
   (integer) 0
    
   redis 127.0.0.1:6379> ZRANGEBYSCORE salary -inf +inf               # 显示整个有序集
   1) "jack"
   2) "tom"
   3) "peter"
    
   redis 127.0.0.1:6379> ZRANGEBYSCORE salary -inf +inf WITHSCORES    # 显示整个有序集及成员的 score 值
   1) "jack"
   2) "2500"
   3) "tom"
   4) "5000"
   5) "peter"
   6) "12000"
    
   redis 127.0.0.1:6379> ZRANGEBYSCORE salary -inf 5000 WITHSCORES    # 显示工资 <=5000 的所有成员
   1) "jack"
   2) "2500"
   3) "tom"
   4) "5000"
   redis 127.0.0.1:6379> ZRANGEBYSCORE salary (5000 400000            # 显示工资大于 5000 小于等于 400000 的成员
   1) "peter"
   ```

10.  ZRANK key member 

    用法：返回有序集中指定成员的排名。其中有序集成员按分数值递增(从小到大)顺序排列。 

    返回： 如果成员是有序集 key 的成员，返回 member 的排名。 如果成员不是有序集 key 的成员，返回 nil 。 

    ```
    127.0.0.1:7000>  ZRANGE {myzset}3 0 -1 WITHSCORES
    1) "redis"
    2) "3"
    3) "mysql"
    4) "5"
    5) "oracle"
    6) "7"
    127.0.0.1:7000> ZRANK {myzset}3  redis
    (integer) 0
    127.0.0.1:7000> ZRANK {myzset}3  mysql
    (integer) 1
    ```

11. ZREM  key member 

    用法：用于移除有序集中的一个或多个成员，不存在的成员将被忽略。当 key 存在但不是有序集类型时，返回一个错误。

    返回： 被成功移除的成员的数量，不包括被忽略的成员。 

    ```
    # 测试数据
     
    redis 127.0.0.1:6379> ZRANGE page_rank 0 -1 WITHSCORES
    1) "bing.com"
    2) "8"
    3) "baidu.com"
    4) "9"
    5) "google.com"
    6) "10"
     
     
    # 移除单个元素
     
    redis 127.0.0.1:6379> ZREM page_rank google.com
    (integer) 1
     
    redis 127.0.0.1:6379> ZRANGE page_rank 0 -1 WITHSCORES
    1) "bing.com"
    2) "8"
    3) "baidu.com"
    4) "9"
     
     
    # 移除多个元素
     
    redis 127.0.0.1:6379> ZREM page_rank baidu.com bing.com
    (integer) 2
     
    redis 127.0.0.1:6379> ZRANGE page_rank 0 -1 WITHSCORES
    (empty list or set)
     
     
    # 移除不存在元素
     
    redis 127.0.0.1:6379> ZREM page_rank non-exists-element
    (integer) 0
    ```

12.  ZREMRANGEBYLEX key min max 

    用法：用于移除有序集合中给定的字典区间的所有成员。 

    返回： 被成功移除的成员的数量，不包括被忽略的成员。 

    ```
    redis 127.0.0.1:6379> ZADD myzset 0 aaaa 0 b 0 c 0 d 0 e
    (integer) 5
    redis 127.0.0.1:6379> ZADD myzset 0 foo 0 zap 0 zip 0 ALPHA 0 alpha
    (integer) 5
    redis 127.0.0.1:6379> ZRANGE myzset 0 -1
    1) "ALPHA"
     2) "aaaa"
     3) "alpha"
     4) "b"
     5) "c"
     6) "d"
     7) "e"
     8) "foo"
     9) "zap"
    10) "zip"
    redis 127.0.0.1:6379> ZREMRANGEBYLEX myzset [alpha [omega
    (integer) 6
    redis 127.0.0.1:6379> ZRANGE myzset 0 -1
    1) "ALPHA"
    2) "aaaa"
    3) "zap"
    4) "zip"
    redis> 
    ```

13.  ZREMRANGEBYRANK key start stop 

    用法： 用于移除有序集中，指定排名(rank)区间（闭区间）内的所有成员。 

    返回： 被移除成员的数量。 

    ```
    127.0.0.1:7001> ZRANGE myset6 0 -1
    1) "kafka"
    2) "java"
    3) "python"
    127.0.0.1:7001> ZREMRANGEBYRANK myset6 0 1
    (integer) 2
    127.0.0.1:7001> ZRANGE myset6 0 -1
    1) "python"
    ```

14.  ZREMRANGEBYSCORE key min max 

     用法：用于移除有序集中，指定分数（score）区间内的所有成员。 

     返回： 被移除成员的数量。 

    ```
    127.0.0.1:7001> ZRANGE myset6 0 -1 WITHSCORES
    1) "kafka"
    2) "3"
    3) "java"
    4) "4"
    5) "python"
    6) "6"
    127.0.0.1:7001> ZREMRANGEBYSCORE myset6 3 4
    (integer) 2
    127.0.0.1:7001> ZRANGE myset6 0 -1 WITHSCORES
    1) "python"
    2) "6"
    ```

15.  ZREVRANGE key start stop [WITHSCORES] 

    用法：返回有序集中，指定区间内的成员。其中成员的位置按分数值递减(从大到小)来排列。具有相同分数值的成员按字典序的逆序(reverse lexicographical order)排列。除了成员按分数值递减的次序排列这一点外， ZREVRANGE 命令的其他方面和 [ZRANGE](https://www.redis.net.cn/order/8740.html) 命令一样。

    返回： 指定区间内，带有分数值(可选)的有序集成员的列表。 

    ```
    127.0.0.1:7001> ZRANGE myset6 0 -1 WITHSCORES
    1) "kafka"
    2) "3"
    3) "java"
    4) "4"
    5) "python"
    6) "6"
    127.0.0.1:7001> ZREVRANGE myset6 0 -1 WITHSCORES
    1) "python"
    2) "6"
    3) "java"
    4) "4"
    5) "kafka"
    6) "3"
    ```

16.  ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count] 

    用法：返回有序集中指定分数区间内的所有的成员。有序集成员按分数值递减(从大到小)的次序排列。具有相同分数值的成员按字典序的逆序(reverse lexicographical order )排列。除了成员按分数值递减的次序排列这一点外， ZREVRANGEBYSCORE 命令的其他方面和 [ZRANGEBYSCORE](https://www.redis.net.cn/order/8742.html) 命令一样。

    返回： 指定区间内，带有分数值(可选)的有序集成员的列表。 

    ```
    127.0.0.1:7001> ZREVRANGE myset6 0 -1 WITHSCORES
    1) "python"
    2) "6"
    3) "java"
    4) "4"
    5) "kafka"
    6) "3"
    #特定区间从高到低
    127.0.0.1:7001> ZREVRANGEBYSCORE myset6 4 3 WITHSCORES
    1) "java"
    2) "4"
    3) "kafka"
    4) "3"
    #全部从高到低
    127.0.0.1:7001> ZREVRANGEBYSCORE myset6 +inf  -inf  WITHSCORES
    1) "python"
    2) "6"
    3) "java"
    4) "4"
    5) "kafka"
    6) "3"
    ```

17.  ZREVRANK key member 

    用法：返回有序集中成员的排名。其中有序集成员按分数值递减(从大到小)排序。排名以 0 为底，也就是说， 分数值最大的成员排名为 0 。使用 ZRANK 命令可以获得成员按分数值递增(从小到大)排列的排名。

    返回： 如果成员是有序集 key 的成员，返回成员的排名。 如果成员不是有序集 key 的成员，返回 nil 。 

    ```
    #从低到高
    127.0.0.1:7001> ZRANGE myset6 0 -1 WITHSCORES
    1) "kafka"
    2) "3"
    3) "java"
    4) "4"
    5) "python"
    6) "6"
    127.0.0.1:7001> ZRANK myset6 kafka
    (integer) 0
    127.0.0.1:7001> ZRANK myset6 python
    (integer) 2
    #从高到低
    127.0.0.1:7001> ZREVRANGE myset6 0 -1 WITHSCORES
    1) "python"
    2) "6"
    3) "java"
    4) "4"
    5) "kafka"
    6) "3"
    127.0.0.1:7001> ZREVRANK myset6 python
    (integer) 0
    127.0.0.1:7001> ZREVRANK myset6 kafka
    (integer) 2
    ```

18.  ZSCORE key member 

    用法：返回有序集中，成员的分数值。 如果成员元素不是有序集 key 的成员，或 key 不存在，返回 nil 。 

    返回： 成员的分数值，以字符串形式表示。 

    ```
    127.0.0.1:7001> ZRANGE myset6 0 -1 WITHSCORES
    1) "kafka"
    2) "3"
    3) "java"
    4) "4"
    5) "python"
    6) "6"
    127.0.0.1:7001> ZSCORE myset6 python
    "6"
    ```

19.  ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX] 

    用法：计算给定的一个或多个有序集的并集，其中给定 key 的数量必须以 numkeys 参数指定，并将该并集(结果集)储存到 destination 。默认情况下，结果集中某个成员的分数值是所有给定集下该成员分数值之和 。

    返回：保存到 destination 的结果集的成员数量。 

    ```
    127.0.0.1:7002> ZRANGE {setkey}5 0 -1 WITHSCORES
    -> Redirected to slot [2440] located at 127.0.0.1:7000
    1) "java"
    2) "1"
    3) "sql"
    4) "10"
    5) "oracle"
    6) "20"
    127.0.0.1:7000> ZRANGE {setkey}6 0 -1 WITHSCORES
    1) "java"
    2) "10"
    3) "mysql"
    4) "20"
    5) "redis"
    6) "30"
    #关于权重weight：
    #如果含有同一个元素，该元素:{setkey}5*2+{setkey}6*3
    #如果没有含同一个元素:{setkey}5*2、{setkey}6*3
    127.0.0.1:7000> ZUNIONSTORE {setkey}7 2 {setkey}5 {setkey}6 WEIGHTS 2 3
    (integer) 5
    127.0.0.1:7000> ZRANGE {setkey}7 0 -1 WITHSCORES
     1) "sql"
     2) "20"
     3) "java"
     4) "32"
     5) "oracle"
     6) "40"
     7) "mysql"
     8) "60"
     9) "redis"
    10) "90"
    ```

20.  ZSCAN key cursor [MATCH pattern] [COUNT count] 

     用法：用于迭代有序集合中的元素（包括元素成员和元素分值）。

     返回： 返回的每个元素都是一个有序集合元素，一个有序集合元素由一个成员（member）和一个分值（score）组成。 

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

# Redis 持久化

目前Redis持久化的方式有两种： RDB 和 AOF

首先，我们应该明确持久化的数据有什么用，答案是:

> 用于重启后的数据恢复

Redis是一个内存数据库，无论是RDB还是AOF，都是其保证数据恢复的措施。

所以Redis在利用RDB和AOF进行恢复的时候，都会读取RDB或AOF文件，重新加载到内存中。

### RDB

RDB就是Snapshot快照存储，是默认的持久化方式。 可理解为半持久化模式，

> 即按照一定的策略周期性的将数据保存到磁盘。 对应产生的数据文件为dump.rdb，快照的周期通过配置文件中的save参数来定义。

下面是默认的快照设置：

```
dbfilename dump.rdb
# save <seconds> <changes>
save 900 1    #当有一条Keys数据被改变时，900秒刷新到Disk一次
save 300 10   #当有10条Keys数据被改变时，300秒刷新到Disk一次
save 60 10000 #当有10000条Keys数据被改变时，60秒刷新到Disk一次
```

Redis的RDB文件不会坏掉，因为其写操作是在一个新进程中进行的。 当生成一个新的RDB文件时，Redis生成的子进程会先将数据写到一个临时文件中，然后通过原子性rename系统调用将临时文件重命名为RDB文件。

这样在任何时候出现故障，Redis的RDB文件都总是可用的。

同时，Redis的RDB文件也是Redis主从同步内部实现中的一环。

> 第一次Slave向Master同步的实现是： Slave向Master发出同步请求，Master先dump出rdb文件，然后将rdb文件全量传输给slave，然后Master把缓存的命令转发给Slave，初次同步完成。
>
> 第二次以及以后的同步实现是： Master将变量的快照直接实时依次发送给各个Slave。 但不管什么原因导致Slave和Master断开重连都会重复以上两个步骤的过程。

Redis的主从复制是建立在内存快照的持久化基础上的，只要有Slave就一定会有内存快照发生。可以很明显的看到，RDB有它的不足，就是一旦数据库出现问题，那么我们的RDB文件中保存的数据并不是全新的。

从上次RDB文件生成到Redis停机这段时间的数据全部丢掉了。

## AOF（Append-only file）方式

AOF(Append-Only File)比RDB方式有更好的持久化性。

1. 在使用AOF持久化方式时，Redis会将每一个收到的写命令都通过Write函数追加到文件中，类似于MySQL的binlog。

2. 当Redis重启是会通过重新执行文件中保存的写命令来在内存中重建整个数据库的内容。

   > 在Redis重启时会逐个执行AOF文件中的命令来将硬盘中的数据载入到内存中,所以说，载入的速度相较RDB会慢一些

3. 默认情况下,Redis没有开启AOF方式的持久化,可以在redis.conf中通过`appendonly`参数开启:

   ```bash
    appendonly yes         #启用aof持久化方式
    # appendfsync always   #每次收到写命令就立即强制写入磁盘，最慢的，但是保证完全的持久化，不推荐使用
    appendfsync everysec     #每秒钟强制写入磁盘一次，在性能和持久化方面做了很好的折中，推荐
    # appendfsync no #完全依赖OS的写入，一般为30秒左右一次，性能最好但是持久化最没有保证，不被推荐。
   ```

4. AOF文件和 RDB文件的保存文件夹位置相同,都是通过dir参数设置的,默认的文件名是`appendonly.aof`,可以通过`appendfilename`参数修改

   ```bash
    appendfilename appendonly.aof
   ```

5. AOF的完全持久化方式同时也带来了另一个问题，持久化文件会变得越来越大。

   > 比如: 我们调用`INCR test` 命令100次，文件中就必须保存全部的100条命令，但其实`99`条都是多余的。 因为要恢复数据库的状态其实文件中保存一条`SET test 100`就够了。
   >
   > 为了压缩AOF的持久化文件，Redis提供了`bgrewriteaof`命令。收到此命令后Redis将使用与快照类似的方式将内存中的数据以命令的方式保存到临时文件中，最后替换原来的文件，以此来实现控制AOF文件的增长。

   配置redis自动重写AOF文件的参数

   ```bash
    no-appendfsync-on-rewrite yes   #在AOF重写时，不进行命令追加操作，而只是将其放在缓冲区里，避免与命令的追加造成`DISK IO`上的冲突。
    auto-aof-rewrite-percentage 100 #当前AOF文件大小是上次日志重写得到AOF文件大小的二倍时，自动启动新的日志重写过程。
    auto-aof-rewrite-min-size 64mb #当前AOF文件启动新的日志重写过程的最小值，避免刚刚启动Reids时由于文件尺寸较小导致频繁的重写。
   ```

## 到底选择什么呢？

下面是来自官方的建议：

通常，如果你要想提供很高的数据保障性，那么建议你同时使用两种持久化方式。

> 如果你可以接受灾难带来的几分钟的数据丢失，那么你可以仅使用RDB。
>
> 很多用户仅使用了AOF，但是我们建议，既然RDB可以时不时的给数据做个完整的快照，并且提供更快的重启，所以建议也使用RDB。

因此，我们希望可以在未来（长远计划）统一AOF和RDB成一种持久化模式。

在数据恢复方面：RDB的启动时间会更短，原因有两个：

> 一是RDB文件中每一条数据只有一条记录，不会像AOF日志那样可能有一条数据的多次操作记录。所以每条数据只需要写一次就行了。
>
> 另一个原因是RDB文件的存储格式和Redis数据在内存中的编码格式是一致的，不需要再进行数据编码工作，所以在CPU消耗上要远小于AOF日志的加载。

二、灾难恢复模拟

既然持久化的数据的作用是用于重启后的数据恢复，那么我们有必要进行一次这样的灾难恢复模拟了。

> 如果数据要做持久化又想保证稳定性，则建议留空一半的物理内存。因为在进行快照的时候，fork出来进行dump操作的子进程会占用与父进程一样的内存，真正的copy-on-write，对性能的影响和内存的耗用都是比较大的。

目前，通常的设计思路是利用`Replication`机制来弥补aof、snapshot性能上的不足，达到了数据可持久化。

> 即Master上Snapshot和AOF都不做，来保证Master的读写性能，而Slave上则同时开启Snapshot和AOF来进行持久化，保证数据的安全性。

首先，修改Master上的如下配置：

```bash
$ sudo vim /redis/etc/redis.conf
#save 900 1 #禁用Snapshot
#save 300 10
#save 60 10000

appendonly no #禁用(注释)AOF
```

接着，修改Slave上的如下配置：

```bash
$ sudo vim /redis/etc/redis.conf
save 900 1 #启用Snapshot
save 300 10
save 60 10000

appendonly yes #启用AOF
appendfilename appendonly.aof #AOF文件的名称
# appendfsync always
appendfsync everysec #每秒钟强制写入磁盘一次
# appendfsync no  

no-appendfsync-on-rewrite yes   #在日志重写时，不进行命令追加操作
auto-aof-rewrite-percentage 100 #自动启动新的日志重写过程
auto-aof-rewrite-min-size 64mb  #启动新的日志重写过程的最小值
```

分别启动Master与Slave

```bash
$ redis-server /etc/redis/redis.conf
```

启动完成后在Master中确认未启动Snapshot参数

```bash
redis 127.0.0.1:6379> CONFIG GET save
1) "save"
2) ""
```

然后通过以下脚本在Master中生成25万条数据：

```bash
python@redis:$ cat redis-cli-generate.temp.sh
#!/bin/bash

REDISCLI="redis-cli -a slavepass -n 1 SET"
ID=1

while(($ID<50001))
do
  INSTANCE_NAME="i-2-$ID-VM"
  UUID=`cat /proc/sys/kernel/random/uuid`
  PRIVATE_IP_ADDRESS=10.`echo "$RANDOM % 255 + 1" | bc`.`echo "$RANDOM % 255 + 1" | bc`.`echo "$RANDOM % 255 + 1" | bc`\
  CREATED=`date "+%Y-%m-%d %H:%M:%S"`

  $REDISCLI vm_instance:$ID:instance_name "$INSTANCE_NAME"
  $REDISCLI vm_instance:$ID:uuid "$UUID"
  $REDISCLI vm_instance:$ID:private_ip_address "$PRIVATE_IP_ADDRESS"
  $REDISCLI vm_instance:$ID:created "$CREATED"

  $REDISCLI vm_instance:$INSTANCE_NAME:id "$ID"

  ID=$(($ID+1))
done
python@redis:$ ./redis-cli-generate.temp.sh
```

在数据的生成过程中，可以很清楚的看到Master上仅在第一次做Slave同步时创建了dump.rdb文件，之后就通过增量传输命令的方式给Slave了。 dump.rdb文件没有再增大。

```bash
python@redis:/opt/redis/data/6379$ ls -lh
total 4.0K
-rw-r--r-- 1 root root 10 Sep 27 00:40 dump.rdb
```

而Slave上则可以看到dump.rdb文件和AOF文件在不断的增大，并且AOF文件的增长速度明显大于dump.rdb文件。

```bash
python@redis-slave:$ ls -lh
total 24M
-rw-r--r-- 1 root root 15M Sep 27 12:06 appendonly.aof
-rw-r--r-- 1 root root 9.2M Sep 27 12:06 dump.rdb
```

等待数据插入完成以后，首先确认当前的数据量。

```bash
redis 127.0.0.1:6379> info
...
...

used_memory:33055824
used_memory_human:31.52M
used_memory_rss:34717696
used_memory_peak:33055800
used_memory_peak_human:31.52M
...

# Keyspace
db0:keys=1,expires=0,avg_ttl=0
db1:keys=250000,expires=0
```

当前的数据量为`db0:keys=1 db1:keys=250000`条，占用内存31.52M。

然后我们直接Kill掉Master的Redis进程，模拟灾难。

```bash
python@redis:$ sudo killall -9 redis-server
```

我们到Slave中查看状态：

```bash
redis 127.0.0.1:6379> info
...
...

used_memory:33047696
used_memory_human:31.52M
used_memory_rss:34775040
used_memory_peak:33064400
used_memory_peak_human:31.53M
...

master_host:10.6.1.143
master_port:6379
master_link_status:down
master_last_io_seconds_ago:-1
master_sync_in_progress:0
...

# Keyspace
db0:keys=1,expires=0,avg_ttl=0
db1:keys=250000,expires=0
```

可以看到`master_link_status`的状态已经是down了，Master已经不可访问了。 而此时，Slave依然运行良好，并且保留有AOF与RDB文件。

下面我们将通过Slave上保存好的AOF与RDB文件来恢复Master上的数据。

> 首先，将Slave上的同步状态取消，避免主库在未完成数据恢复前就重启，进而直接覆盖掉从库上的数据，导致所有的数据丢失。

如果Redis服务器已经充当从站命令`SLAVEOF NO ONE` 会关掉复制，转Redis服务器为主

```
redis 127.0.0.1:6379> SLAVEOF NO ONE
OK
```

确认一下已经没有了master相关的配置信息：

```bash
redis 127.0.0.1:6379> INFO
...
role:master
...
```

在Slave上复制数据文件：

```bash
python@redis-slave:$ tar cvf data.tar *
appendonly.aof
dump.rdb
```

将data.tar上传到Master上，尝试恢复数据: 可以看到Master目录下有一个初始化Slave的数据文件，很小，将其删除。

```bash
python@redis:$ ls -l
total 4
-rw-r--r-- 1 root root 10 Sep 27 00:40 dump.rdb
python@redis:/opt/redis/data/6379$ sudo rm -f dump.rdb
```

然后解压缩数据文件：

```bash
python@redis:$ sudo tar xf data.tar
python@redis:$ ls -lh
total 29M
-rw-r--r-- 1 root root 18M Sep 27 01:22 appendonly.aof
-rw-r--r-- 1 root root 12M Sep 27 01:22 dump.rdb
```

启动Master上的Redis；

```bash
python@redis:$ redis-server /etc/redis/redis.conf
Starting Redis server...
```

查看数据是否恢复：

```bash
redis 127.0.0.1:6379> INFO
...
db0:...
db1:keys=250000,expires=0
```

可以看到25万条数据已经完整恢复到了Master上。 此时，可以放心的恢复Slave的同步设置了

```bash
redis 127.0.0.1:6379> SLAVEOF 10.6.1.143 6379
OK
```

查看同步状态：

```
redis 127.0.0.1:6379> INFO
...
role:slave
master_host:10.6.1.143
master_port:6379
master_link_status:up
...
```

## 恢复数据策略

Redis允许同时开启AOF和RDB,既保证了数据安全又使得进行备份等操作十分容易, 那么到底是哪一个文件完成了数据的恢复呢？

实际上，当Redis服务器挂掉时，重启时将按照以下优先级恢复数据到内存：

1. 如果只配置AOF,重启时加载AOF文件恢复数据；
2. 如果同时 配置了RDB和AOF,启动是只加载AOF文件恢复数据;
3. 如果只配置RDB,启动是将加载dump文件恢复数据。

也就是说，AOF的优先级要高于RDB，这也很好理解，因为AOF本身对数据的完整性保障要高于RDB。

#### 总结

目前的线上环境中，由于数据都设置有过期时间，采用AOF的方式会不太实用，因为过于频繁的写操作会使AOF文件增长到异常的庞大，大大超过了我们实际的数据量，这也会导致在进行数据恢复时耗用大量的时间。

因此，可以在Slave上仅开启Snapshot来进行本地化，同时可以考虑将save中的频率调高一些或者调用一个计划任务来进行定期bgsave的快照存储，来尽可能的保障本地化数据的完整性。

在这样的架构下，如果仅仅是Master挂掉，Slave完整，数据恢复可达到100%。

如果Master与Slave同时挂掉的话，数据的恢复也可以达到一个可接受的程度。

## Redis的同步机制

**主同步过程：**

- Salve发送sync命令到Master
- Master会启动一个后台进程，将Redis中的数据快照保存到文件中
- 同时Master将保存数据快照期间接收到的写命令（增量数据）缓存起来
- Master完成写文件操作后，将该文件发送给Salve
- Salve将文件保存到磁盘中，加载文件到内存中，加载数据快照，使用新的AOF文件替换掉旧的AOF文件
- Salve完成数据快照之后，Master将这期间收集的增量写命令发送给Salve端

**增量同步的过程：**

- Master接收到用户的操作指令，判断是否需要传播到Salve
- 将操作记录追加到AOF文件
- 将操作传播到其他的Salve：1、对齐主从库；2、往响应缓存写入指令
- 将缓存中的数据发送给salve

## Redis Sentinel

解决主从同步Master宕机后的主从切换问题：

- 监控：监测主从服务器是否运行正常
- 提醒：通过API向管理员或者其他应用程序发送故障通知
- 自动故障迁移：主从切换（redis sentinel是一个分布式的系统，可以在架构中运行多个sentinel进程，这些进程使用流言协议来接收主服务器是否下线的信息，并使用投票协议决定是否使用自动故障迁移）

