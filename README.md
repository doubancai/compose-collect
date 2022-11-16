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

1.拷贝容器内配置文件到宿主机

```
docker run -it --rm  --name temp-kafka wurstmeister/kafka:2.13-2.8.1 /bin/bash

docker cp temp-kafka:/opt/kafka/config/ ./services/kafka/
```

2.进入容器

```
docker exec -it compose-collect-kafka-1 /bin/bash
```

3.创建topic

```
kafka-topics.sh --create --zookeeper compose-collect-zookeeper-1:2181 --replication-factor 1 --partitions 3 --topic test-flume
```

4.消费

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
# 从指定分区指定offset开始消费
kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic test-1 --partition 0 --offset 10
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

# 设置消费组从多个分区指定offset开始消费（关闭消费后执行）
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group console-consumer-9542 --reset-offsets –-topic test-1:0,1,2 --to-offset 10 --execute
# 设置消费组从某个分区指定offset开始消费（关闭消费后执行）
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group console-consumer-9542 --reset-offsets –-topic test-1:0 --to-offset 10 --execute

```



## metabase

创建数据库

```
CREATE DATABASE metabase CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```



## ClickHouse

1.拷贝容器内配置文件到宿主机

```
docker run -it --rm  --name temp-clickhouse-server --ulimit nofile=262144:262144 clickhouse/clickhouse-server:22.10.1.1877 /bin/bash

docker cp temp-clickhouse-server:/etc/clickhouse-server/ ./services/ch-server/
```

2.server启动后进入click

```
docker exec -it compose-collect-ch-server-1 clickhouse-client
```

3.建表

```
-- 创建数据库
create database test;

-- 创建消费kafka数据表
CREATE TABLE test.flume_consumer (
    id UInt64,
    msg String,
    dt UInt64
)
ENGINE = Kafka
SETTINGS kafka_broker_list = 'kafka:9092',
    kafka_topic_list = 'test-flume',
    kafka_group_name = 'flume_consumer',
    kafka_format = 'JSONEachRow',
    kafka_skip_broken_messages = 1,
    kafka_num_consumers = 2;

-- 创建存储数据表
CREATE TABLE test.flume (
    id UInt64,
    msg String,
    dt DateTime
) Engine = ReplacingMergeTree() PARTITION BY toYYYYMMDD(dt) ORDER BY (dt, id);


-- 创建视图从消费表导出到存储表
CREATE MATERIALIZED VIEW test.flume_view TO test.flume AS
SELECT
    id,
    msg,
    fromUnixTimestamp64Milli(dt) as dt
FROM test.flume_consumer;
```

4.source中向test_1.log文件新增内容

```
{"id":1,"msg":"1","dt":1666865462000}
{"id":1,"msg":"2","dt":1666865462000}
{"id":2,"msg":"2","dt":1666865462000}
```

5.查询

```
select * from test.flume;

# 强制分区合并后查询
optimize table test.flume FINAL;
select * from test.flume;
```

维护

```
1.停止kafka消息写入,避免大量消息堆积
2.卸载视图,停止接收kafka消息
DETACH TABLE test.flume_view;
恢复
1.恢复视图
ATTACH TABLE test.flume_view;
2.启动kafka
```

