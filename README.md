# kafka
How to use and install kafka

[kafka documentation](https://kafka.apache.org/documentation/)

## 1 Deployment with dokcer compose
```
version: '3.8'

services:
  zookeeper:
    image: docker.1ms.run/confluentinc/cp-zookeeper:7.7.0
    container_name: zookeeper
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"

  kafka:
    image: docker.1ms.run/confluentinc/cp-kafka:7.7.0
    container_name: kafka
    depends_on:
      - zookeeper
    ports:
      - "19092:19092"
      - "29092:29092"
      - "19999:9999"   # JMX（可选）
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      # 监听器定义（必须不同端口或 IPv4/IPv6）
      KAFKA_LISTENERS: PLAINTEXT://localhost:19092,PLAINTEXT_HOST://0.0.0.0:29092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:19092,PLAINTEXT_HOST://10.0.0.16:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_JMX_PORT: 9999
    healthcheck:
      test: ["CMD", "nc", "-z", "localhost", "19092"]
      interval: 10s
      timeout: 5s
      retries: 5
```

## 2 Test kafka in docker container
```
docker exec -it kafka bash
```

- list all topics
```
kafka-topics --list \
  --bootstrap-server 10.0.0.16:29092
```

- create topics
```
kafka-topics --create \
  --topic test1 \
  --bootstrap-server 10.0.0.16:29092 \
  --partitions 3 \
  --replication-factor 1
```

- **producer**
```
# **生产消息**
kafka-console-producer \
  --topic test1 \
  --bootstrap-server 10.0.0.16:29092
# type hello kafka then press ctl+c
```

- *customer*
```
# 消费消息（另开终端）
kafka-console-consumer \
  --topic test1 \
  --from-beginning \
  --bootstrap-server 10.0.0.16:29092
```

```
# *消费消息（另开终端）*
docker exec -it kafka kafka-console-consumer \
  --topic test1 \
  --from-beginning \
  --bootstrap-server 10.0.0.16:29092
```



