#### Kafka 消息队列

```
传统定义：分布式的基于分布/订阅模式的消息队列
最新定义：开源的分布式流平台
```

#### Docker 安装zookeeper和Kafka

```text
zookeeper的安装
docker pull wurstmeister/zookeeper //拉取镜像
docker run -d --name zookeeper -p 2181:2181 -t wurstmeister/zookeeper//启动镜像zookeeper
Kafka的安装
docker pull wurstmeister/kafka//拉取镜像
启动镜像
docker run -d --restart=always --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=1 -e KAFKA_auto_create_topics_enable=true -e KAFKA_ZOOKEEPER_CONNECT=192.168.1.101:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.1.101:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -t wurstmeister/kafka 
```

```
测试Kafka
进入容器
docker exec -it kafka /bin/bash
进入Kafka执行命令的文件下
cd /opt/kafka_2.13-2.8.1/bin
创建话题
 ./kafka-topics.sh --create --zookeeper 192.168.1.101:2181 --replication-factor 1 --partitions 1 --topic mykafka
运行一个消费者
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic mykafka --from-beginning
运行一个生产者
./kafka-console-producer.sh --broker-list localhost:9092 --topic mykafka
查看话题
bin/kafka-topics.sh --list --zookeeper 192.168.1.101:2181　
```

```
在docker 运行zookeeper容器时会报错
bin/zkServer.sh: 81: /home/ljj/software/zookeeper-3.4.13/bin/zkEnv.sh: Syntax error: "(" unexpected (expecting "fi")
错误原因：
　　Ubuntu的默认shell为dash
解决方法：
　　将dash 改为　bash　
　　ls -l /bin/sh
　　ln -sf bash /bin/sh
　　ls -l /bin/sh
然后重启zookeeper
```

