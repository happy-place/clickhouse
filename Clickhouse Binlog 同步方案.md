# Clickhouse Binlog 同步方案

## 简介

```shell
mysql -> 
	maxwell -> 
		kafka -> stream_table (engine = kafka) ->  
			material view -> binlog_table (engine=MergeTree) ->
				select from distribute_table (engine=Distribute)
```

## 部署

### 规划

| 节点        | IP           | 备注                                   |
| ----------- | ------------ | -------------------------------------- |
| fix-net     | 192.168.0.1  | 自定义网关                             |
| mysql5.7    | 192.168.0.3  | Ver 14.14 Distrib 5.7.32、端口：3307   |
| zk          | 192.168.0.9  | apache-zookeeper-3.6.2-bin、端口：2181 |
| kafka | 192.168.0.11 | kafka_2.12、端口：9092    |
| maxwell | 192.168.0.12 | maxwell-1.29.0 |
| ck-node1    | 192.168.0.5  | 20.12.3.3、端口：9001、8124、9010      |
| ck-node2    | 192.168.0.6  | 20.12.3.3、端口：9002、8125、9011      |
| ck-node3    | 192.168.0.7  | 20.12.3.3、端口：9003、8126、9012      |
| ck-node4    | 192.168.0.8  | 20.12.3.3、端口：9004、8127、9013      |

### mysql

```shell
docekr start mysql5.7
```

### zookeeper

```shell
docker start zk
```

### kafka

```shell
docker pull whohow20094702/kafka_2.12-2.5.0:v1.0

docker run -itd \
--name kafka \
--net fix-net \
--ip 192.168.0.11 \
-p 9092:9092 \
-e ZOOKEEPER_HOST=192.168.0.9:2181 \
-e LOG_RETENTION_HOURS=200 \
whohow20094702/kafka_2.12-2.5.0:v1.0

docker ps

# 测试 kafka 是否能够正常运行
docker exec -it kafka bash -c 'cat $KAFKA_HOME/config/server.properties | grep zookeeper.connect='
# zookeeper.connect=192.168.0.9:2181
docker exec -it kafka bash -c 'kafka create_topic test 3 1'
# Created topic test.
```

### maxwell

```shell
docker pull zendesk/maxwell

# 拷贝配置（--rm Automatically remove the container when it exits）
docker run -itd --name maxwell --rm zendesk/maxwell bash -c 'sleep 1000000000'
docker cp maxwell:/app/config.properties.template $DOCKERPLACE/maxwell-1.29.0/conf/config.properties

docker run -idt \
--name maxwell \
--net fix-net \
--ip 192.168.0.12 \
-v $DOCKERPLACE/maxwell-1.29.0/conf/config.properties:/app/config.properties \
zendesk/maxwell bin/maxwell --producer=kafka

docker logs maxwell
# Using kafka version: 1.0.0
# 10:04:01,440 INFO  Maxwell - Starting Maxwell. maxMemory: 524288000 bufferMemoryUsage: 0.25
# .....
# 10:04:13,965 INFO  BinaryLogClient - Connected to mysql5.7:3306 at mysql5.7-bin/7610 (sid:6379, cid:57)
# 10:04:13,966 INFO  BinlogConnectorReplicator - Binlog connected.
```

```
ka
```

