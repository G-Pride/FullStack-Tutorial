# 安装

## **单机伪分布式搭建**

版本：kafka_2.12-2.0.0.tgz

步骤：

1. 解压：tar zvxf kafka_2.12-2.0.0.tgz

2. 进入kafka_2.12-2.0.0/config/目录下

   ```
   cp server.properties server1.properties
   cp server.properties server2.properties
   ```

3. vi server.properties

   ```
   broker.id=1
   
   #搭建集群必须指定，不然默认指向9092
   port=9091
   
   log.dirs=/home/appuser/kafka/kafka_2.12-2.0.0/log1
   
   #因为单机所以都是127.0.0.1，如果非单机则除了本机是127.0.0.1，其他为各自服务器对应的ip地址
   zookeeper.connect=127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183
   ```
   
   vi server1.properties

   ```
broker.id=2
   
   #搭建集群必须指定，不然默认指向9092
   port=9092
   
   log.dirs=/home/appuser/kafka/kafka_2.12-2.0.0/log2
   
   #因为单机所以都是127.0.0.1，如果非单机则除了本机是127.0.0.1，其他为各自服务器对应的ip地址
   zookeeper.connect=127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183
   ```
   
   vi server2.properties
   
   ```
broker.id=3
   
#搭建集群必须指定，不然默认指向9092
   port=9093
   
   log.dirs=/home/appuser/kafka/kafka_2.12-2.0.0/log3
   
   #因为单机所以都是127.0.0.1，如果非单机则除了本机是127.0.0.1，其他为各自服务器对应的ip地址
   zookeeper.connect=127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183
   ```
   
   *log1~3文件夹不存在，需要手动创建*
   
4. 进入kafka_2.12-2.0.0/目录，vi kafka-start-all.sh

   ```
   ./bin/kafka-server-start.sh ./config/server.properties &
   ./bin/kafka-server-start.sh ./config/server1.properties &
   ./bin/kafka-server-start.sh ./config/server2.properties &
   ```

**注意：要确保9091~9093跟2181~2183能成功联通。**

