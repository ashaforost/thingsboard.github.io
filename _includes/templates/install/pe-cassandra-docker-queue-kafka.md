
[Apache Kafka](https://kafka.apache.org/) is an open-source stream-processing software platform.

Configuration environment file for ThingsBoard queue service:

```text
nano tb-node.env
```
{: .copy-code}

Add the following line to the file.

```bash
TB_QUEUE_TYPE=kafka
TB_KAFKA_SERVERS=kafka:9092
```
{: .copy-code}

Check docker-compose.yml and configure ports if you need:

```yml
services:
  kafka:
    restart: always
    image: bitnami/kafka:3.5.2
    ports:
      - 9092:9092 #to kafka:9092 from host machine
      - 9093 #for Kraft
      - 9094 #to kafka:9094 from within Docker network
    environment:
      ALLOW_PLAINTEXT_LISTENER: "yes"
      KAFKA_CFG_LISTENERS: "OUTSIDE://:9092,CONTROLLER://:9093,INSIDE://:9094"
      KAFKA_CFG_ADVERTISED_LISTENERS: "OUTSIDE://localhost:9092,INSIDE://kafka:9094"
      KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP: "INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT,CONTROLLER:PLAINTEXT"
      KAFKA_CFG_INTER_BROKER_LISTENER_NAME: "INSIDE"
      KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE: "false"
      KAFKA_CFG_LOG_RETENTION_BYTES: "1073741824"
      KAFKA_CFG_SEGMENT_BYTES: "268435456"
      KAFKA_CFG_LOG_RETENTION_MS: "300000"
      KAFKA_CFG_LOG_CLEANUP_POLICY: "delete"
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: "1"
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: "1"
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: "1"
      KAFKA_CFG_PROCESS_ROLES: "controller,broker" #KRaft
      KAFKA_CFG_NODE_ID: "0" #KRaft
      KAFKA_CFG_CONTROLLER_LISTENER_NAMES: "CONTROLLER" #KRaft
      KAFKA_CFG_CONTROLLER_QUORUM_VOTERS: "0@kafka:9093" #KRaft
    volumes:
      - kafka-data:/bitnami
  tbpe:
    restart: always
    image: "${DOCKER_REPO}/${TB_NODE_DOCKER_NAME}:${TB_VERSION}"
    ports:
      - "8080:8080"
      - "1883:1883"
      - "7070:7070"
      - "5683-5688:5683-5688/udp"
    depends_on:
      - kafka
volumes:
  kafka-data:
    driver: local
```
{: .copy-code}
