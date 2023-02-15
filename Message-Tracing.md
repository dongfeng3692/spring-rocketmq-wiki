We need two more configurations for support message trace in producer.

```properties
## application.properties
rocketmq.name-server=127.0.0.1:9876
rocketmq.producer.group=my-group

rocketmq.producer.enable-msg-trace=true
rocketmq.producer.customized-trace-topic=my-trace-topic
```

The message trace in consumer should configure in `@RocketMQMessageListener`.

```
@Service
@RocketMQMessageListener(
    topic = "test-topic-1", 
    consumerGroup = "my-consumer_test-topic-1",
    enableMsgTrace = true,
    customizedTraceTopic = "my-trace-topic"
)
public class MyConsumer implements RocketMQListener<String> {
    ...
}
```
> Note:
> 
> Maybe you need change `127.0.0.1:9876` with your real NameServer address for RocketMQ

> By default, the message track feature of Producer and Consumer is turned on and the trace-topic is RMQ_SYS_TRACE_TOPIC
> The topic of message trace can be configured with `rocketmq.consumer.customized-trace-topic` configuration item, not required to be configured in each `@RocketMQMessageListener`

For Alibaba Cloud message traces, you need to set the `accessChannel` configuration in `@RocketMQMessageListener` to `CLOUD`.