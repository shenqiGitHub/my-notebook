# This will have no impact if delete.topic.enable is not set to true以及删除kafka中的topic

$KAFKA/config/server.properties中

delete.topic.enble=true

 

然后删除topic

$ ./kafka-topics.sh --zookeeper Desktop:2181,Laptop:2182,Laptop:2183 --delete --topic my-topic
Topic my-topic is marked for deletion.





# How to delete multiple topics in Apache Kafka

```
./bin/kafka-topics.sh --zookeeper localhost:2181 --delete --topic 'giorgos-.*'
```