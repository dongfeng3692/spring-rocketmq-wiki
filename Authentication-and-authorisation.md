We need two more configurations for support ACL in producer.

```properties
## application.properties
rocketmq.name-server=127.0.0.1:9876
rocketmq.producer.group=my-group

rocketmq.producer.access-key=AK
rocketmq.producer.secret-key=SK
```
Transaction Message should configure AK/SK in `@RocketMQTransactionListener`. 

```
@RocketMQTransactionListener(
    txProducerGroup = "test,
    accessKey = "AK",
    secretKey = "SK"
)
class TransactionListenerImpl implements RocketMQLocalTransactionListener {
    ...
}
```

> Note:
> 
> You do not need to configure AK/SK for each `@RocketMQTransactionListener`, you could configure `rocketmq.producer.access-key` and `rocketmq.producer.secret-key` as default value

The ACL feature in consumer should configure AK/SK in `@RocketMQMessageListener`.

```
@Service
@RocketMQMessageListener(
    topic = "test-topic-1", 
    consumerGroup = "my-consumer_test-topic-1",
    accessKey = "AK",
    secretKey = "SK"
)
public class MyConsumer implements RocketMQListener<String> {
    ...
}
```

> Note:
> 
> You do not need to configure AK/SK for each `@RocketMQMessageListener`, you could configure `rocketmq.consumer.access-key` and `rocketmq.consumer.secret-key` as default value