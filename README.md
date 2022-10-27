# compose-collect

## Flume

1.构建容器

```
docker-compose build
```

2.启动

```
docker-compose up
```

source中新建test_1.log，向文件中新增内容，将打印到控制台中



直接运行容器启动flume

```
docker run -it --rm --name flume_test -v $PWD/code/localhost/flume/:/var/flume/ compose-collect-flume:latest /bin/bash
```

```
flume-ng agent -n a1 -c /opt/flume/conf -f /var/flume/conf/test.conf -Dflume.root.logger=INFO,console
```



## Kafka

1.进入容器

```
docker exec -it compose-collect-kafka-1 /bin/bash
```

2.创建topic

```
kafka-topics.sh --create --zookeeper compose-collect-zookeeper-1:2181 --replication-factor 1 --partitions 3 --topic test-flume
```

3.消费

```
kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --from-beginning --topic test-flume
```

source中向test_1.log文件新增内容，将打印到消费客户端



常用命令

```
# 创建topic 
kafka-topics.sh --create --zookeeper compose-collect-zookeeper-1:2181 --replication-factor 1 --partitions 3 --topic test-1
# 查看topic列表
kafka-topics.sh --zookeeper compose-collect-zookeeper-1:2181 --list
# 查看topic详情
kafka-topics.sh --zookeeper compose-collect-zookeeper-1:2181 --describe --topic test-1
# 删除topic
kafka-topics.sh --delete --zookeeper compose-collect-zookeeper-1:2181 --topic test-1
```

```
# 生产
kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic test-1
# 消费
kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --from-beginning --topic test-1
```

```
全局配置topic清除时间 server.properties 中 log.retention.hours=168

# 设置topic清空配置 
# 不会在retention.ms后直接删除，而是由server.properties里面的log.retention.check.interval.ms轮询检测清空
kafka-configs.sh --bootstrap-server 127.0.0.1:9092 --entity-type topics --entity-name test-1 --alter --add-config retention.ms=10000
# 查看topic清空配置
kafka-configs.sh --bootstrap-server 127.0.0.1:9092 --describe --entity-type topics --entity-name test-1
# 删除topic清空时间配置
kafka-configs.sh --bootstrap-server 127.0.0.1:9092 --entity-type topics --entity-name test-1 --alter --delete-config retention.ms
```

```
# 查看消费者组列表
kafka-consumer-groups.sh --bootstrap-server 127.0.0.1:9092 --list
# 查看消费者组详情
kafka-consumer-groups.sh --bootstrap-server 127.0.0.1:9092 --describe --group console-consumer-9542
```

