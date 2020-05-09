

之前介绍 [Logstash 的用法](http://mp.weixin.qq.com/s?__biz=Mzg4MzAyOTE5Ng==&mid=2247483755&idx=1&sn=91bbe86bdd4e19966ee75ffb70d5bb49&chksm=cf4ce4c8f83b6ddee73bfed38f06aa087b16ee52971b2b1029f8e5dddc15063231f9f0804367&scene=21#wechat_redirect)，这次整合 Kafka 和 Logstash（2.4.1版本）：

*（适合有 Logstash 和 Kafka 搭建使用经验的人群，不同版本参数有所不同，具体以官方对应版本的文档为主）*

![img](C:/Users/49353/Desktop/640-1589004262850.jpg)



## **Logstash -> Kafka**

将 Logstash 的 output 作为 Kafka 的product（生产者），并成功 consume（消费）Logstash 采集的文本数据。



先了解 logstash-output-kafka 插件，主要用到两个参数：

**topic_id**

**bootstrap_servers**

*虽然bootstrap_servers非必须，但是不指定kafka无法消费到数据*



logstash.conf 文件如下：

```
input{  
	file{    
		path => "/home/appuser/logstash/txt/logstashTest.txt"    
	}
}
filter {  
	csv {    
		separator => "?"  
	} 
}
output{  
	kafka{    
		topic_id => "logstash_kafka_topic"    
		bootstrap_servers => "127.0.0.1:9091"   
	}
}
```



根据配置启动 Logstash

```
 ./logstash -f ../../conf/logstash.conf
```



启动 Kafka 消费

```
./kafka-console-consumer.sh --zookeeper 127.0.0.1:2181 --topic logstash_kafka_topic
```



往 logstashTest.txt 输入数据

```
echo "this is my first logstash_kafka?Hello Logstash and Kafka" >> logstashTest.txt
```



查看 Kafka 消费情况

```
{"message":"this is my first logstash_kafka?Hello Logstash and Kafka","@version":"1","@timestamp":"2020-04-26T13:34:51.132Z","path":"/home/appuser/logstash/txt/logstashTest.txt","host":"VM_0_15_centos","column1":"this is my first logstash_kafka","column2":"Hello Logstash and Kafka"}
```



## **Kafka -> Logstash **

将 Kafka 的 produce 作为 Logstash 的数据源，Logstash input接收数据并用 output 输出到控制台**


先了解 logstash-input-kafka 插件主要参数

**topic_id**

**zk_connect**



启动 kafka 的生产者

```
/home/appuser/kafka/kafka_2.11-0.9.0.0/bin/kafka-console-producer.sh --broker-list 127.0.0.1:9091 --topic logstash_kafka_topic
```



新建kafka_to_logstash.conf 

```
input{        
	kafka{                
		topic_id => "logstash_kafka_topic"                
		zk_connect => "127.0.0.1:2181"        
	}
}
filter {     
	csv {       
		separator => "?"      
	}
}
output{        
	stdout{               
		codec => "json"        
	}
}
```



开启 logstash管道

```
./logstash -f ./kafka_to_logstash.conf 
```



在生产端输入信息

```
this is my first kafka_to_logstash test ? hello kafka_to_logstash
```



在 logstash 端查看

```
{"message":"this is my first kafka_to_logstash test ? hello kafka_to_logstash","tags":["_jsonparsefailure"],"@version":"1","@timestamp":"2020-04-27T02:53:59.305Z","column1":"this is my first kafka_to_logstash test ","column2":" hello kafka_to_logstash"}
```



整合成功。