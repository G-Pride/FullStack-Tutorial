# 安装

## 单机伪分布式搭建

版本：1.2.2

步骤：

1. tar -zvxf apache-storm-1.2.2.tar.gz

2. mv apache-storm-1.2.2 storm-1.2.2 

3. mkdir -p storm1 storm2 storm3

4. 将storm-1.2.2下的目录复制到新建目录下

   ```
   cd storm-1.2.2
   cp ./  -r  ../storm1/
   cp ./  -r  ../storm2/
   cp ./  -r  ../storm3/
   ```

5. 修改配置文件

   **vi storm1/conf/storm.yaml**

   ```
     storm.zookeeper.servers: 
      - "127.0.0.1"
     storm.zookeeper.port: 2181
     nimbus.host: "127.0.0.1"
     ui.port: 8085
     supervisor.slots.ports: 
      - 6700
      - 6701
      - 6702
      - 6703
     storm.local.dir: "/home/appuser/storm/storm-cluster/storm-1/workdir"
   ```

   **vi storm2/conf/storm.yaml**

   ```
     storm.zookeeper.servers: 
      - "127.0.0.1"
     storm.zookeeper.port: 2181
     nimbus.host: "127.0.0.1"
     ui.port: 8085
     supervisor.slots.ports: 
      - 7700
      - 7701
      - 7702
      - 7703
     storm.local.dir: "/home/appuser/storm/storm-cluster/storm-2/workdir"
   ```

   **vi storm3/conf/storm.yaml**

   ```
     storm.zookeeper.servers:  
      - "127.0.0.1"
     storm.zookeeper.port: 2183
     nimbus.host: "127.0.0.1"
     ui.port: 8085
     supervisor.slots.ports: 
      - 8700
      - 8701
      - 8702
      - 8703
     storm.local.dir: "/home/appuser/storm/storm-cluster/storm-3/workdir"
   
   ```

6. **启动主节点：**/home/appuser/storm/storm1/bin/storm nimbus &

7. **启动storm ui页面：**/home/appuser/storm/storm1/bin/storm ui &

8. **启动子节点：**/home/appuser/storm/storm2/bin/storm supervisor  &

9. **启动子节点：**home/appuser/storm/storm3/bin/storm supervisor  &

10. **storm ui页面地址：**http://49.234.27.151:8085/index.html

