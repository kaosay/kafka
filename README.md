# kafka
How to use and install kafka


## Deployment with dokcer compose

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
        #KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:29092,PLAINTEXT_HOST://localhost:19092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:19092,PLAINTEXT_HOST://10.0.0.161:29092
        #KAFKA_LISTENERS: 'CONTROLLER://kafka-kraft:29093,PLAINTEXT_HOST://0.0.0.0:19092,PLAINTEXT://kafka-kraft:29092'
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_JMX_PORT: 9999
    healthcheck:
      test: ["CMD", "nc", "-z", "localhost", "19092"]
      interval: 10s
      timeout: 5s
      retries: 5
