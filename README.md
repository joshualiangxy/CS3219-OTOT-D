# Instructions on setting up Kafka cluster

1. Ensure you have Docker installed

2. Run the following command:
```bash
$ docker compose up
```

3. Verify that ZooKeeper is up. Enter the following command:
```bash
$ docker compose logs zookeeper | grep Created
```

4. There should be logs that look similar to the following line:
```bash
zookeeper_1 | [2021-09-25 14:02:41,259] INFO Created server with tickTime 2000 minSessionTimeout 4000 maxSessionTimeout 40000 datadir /var/lib/zookeeper/log/version-2 snapdir /var/lib/zookeeper/data/version-2 (org.apache.zookeeper.server.ZooKeeperServer)
```

5. Next, verify that the Kafka nodes are up. Enter the following command:
```bash
$ docker compose logs kafka-1 | grep started
$ docker compose logs kafka-2 | grep started
$ docker compose logs kafka-3 | grep started
```

6. You should see the similar logs for each respective command:
```bash
kafka-1_1 | [2021-09-25 14:02:47,163] INFO [KafkaServer id=1] started (kafka.server.KafkaServer)
kafka-2_1 | [2021-09-25 14:02:47,172] INFO [KafkaServer id=2] started (kafka.server.KafkaServer)
kafka-3_1 | [2021-09-25 14:02:47,087] INFO [KafkaServer id=3] started (kafka.server.KafkaServer)
```

7. To send a message, we first need to create a topic. Enter the following command:
```bash
$ docker run --net=host --rm confluentinc/cp-kafka:latest kafka-topics --create --topic my-topic --partitions 3 --replication-factor 3 --if-not-exists --zookeeper localhost:22181
```

8. Next, we want to send the message using the following command:
```bash
$ docker run --net=host --rm confluentinc/cp-kafka:latest bash -c "echo 'Hello World!' | kafka-console-producer --broker-list localhost:19092 --topic my-topic"
```

9. To receive the message, use the following command:
```bash
$ docker run --net=host --rm confluentinc/cp-kafka:latest kafka-console-consumer --bootstrap-server localhost:29092 --topic my-topic --from-beginning --max-messages 1
```

10. You should see the following messages:
```bash
Hello World!
Processed a total of 1 messages
```

# Demonstration of management of master node failure

1. We first obtain information on which node is the master node. We do so by getting information about the topic using this command:
```bash
$ docker run --net=host --rm confluentinc/cp-kafka:latest kafka-topics --describe --topic my-topic --zookeeper localhost:22181
```

2. You should see something similar to the following:

![1.png](/images/1.png)

3. From this screenshot, we see that each partition has a leader telling us which node is the master node for the partition. In this case, we see that
    * Partition 0 has node 2 as its leader
    * Partition 1 has node 3 as its leader
    * Partition 2 has node 1 as its leader

4. Next, choose to stop any Kafka container. In this demonstration, I will be stopping node 2.

![2.png](/images/2.png)

5. Use the same command to fetch information about the topic. Now, we see something like this:

![3.png](/images/3.png)

6. Note how the leader of partition 0 has changed from node 2 to node 3.
