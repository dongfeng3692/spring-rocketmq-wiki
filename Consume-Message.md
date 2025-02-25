## Push mode

Modify application.properties
```properties
## application.properties
rocketmq.name-server=127.0.0.1:9876
```

> Note:
> 
> Maybe you need change `127.0.0.1:9876` with your real NameServer address for RocketMQ

```java
@SpringBootApplication
public class ConsumerApplication{
    
    public static void main(String[] args){
        SpringApplication.run(ConsumerApplication.class, args);
    }
    
    @Slf4j
    @Service
    @RocketMQMessageListener(topic = "test-topic-1", consumerGroup = "my-consumer_test-topic-1")
    public class MyConsumer1 implements RocketMQListener<String>{
        public void onMessage(String message) {
            log.info("received message: {}", message);
        }
    }
    
    @Slf4j
    @Service
    @RocketMQMessageListener(topic = "test-topic-2", consumerGroup = "my-consumer_test-topic-2")
    public class MyConsumer2 implements RocketMQListener<OrderPaidEvent>{
        public void onMessage(OrderPaidEvent orderPaidEvent) {
            log.info("received orderPaidEvent: {}", orderPaidEvent);
        }
    }
}
```

> More relevant configurations for consuming:
>
> see: [RocketMQMessageListener](rocketmq-spring-boot/src/main/java/org/apache/rocketmq/spring/annotation/RocketMQMessageListener.java)

## Pull mode

Starting from RocketMQ Spring 2.2.0, RocketMQ Spring supports consume with pull mode.

Modify application.properties
```properties
## application.properties
rocketmq.name-server=127.0.0.1:9876
# When set rocketmq.pull-consumer.group and rocketmq.pull-consumer.topic, rocketmqTemplate will start lite pull consumer
# If you do not want to use lite pull consumer, please do not set rocketmq.pull-consumer.group and rocketmq.pull-consumer.topic
rocketmq.pull-consumer.group=my-group1
rocketmq.pull-consumer.topic=test
```

> Note:
> 
> Maybe you need change `127.0.0.1:9876` with your real NameServer address for RocketMQ
>
> The effective configuration of lite pull consumer before is rocketmq.consumer.group and rocketmq.consumer.topic, but since it is very easy to be confused with push-consumer, it is changed to rocketmq.pull-consumer.group and rocketmq.pull-consumer.topic after version 2.2.3

```java
@SpringBootApplication
public class ConsumerApplication implements CommandLineRunner {

    @Resource
    private RocketMQTemplate rocketMQTemplate;

    @Resource(name = "extRocketMQTemplate")
    private RocketMQTemplate extRocketMQTemplate;

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        //This is an example of pull consumer using rocketMQTemplate.
        List<String> messages = rocketMQTemplate.receive(String.class);
        System.out.printf("receive from rocketMQTemplate, messages=%s %n", messages);

        //This is an example of pull consumer using extRocketMQTemplate.
        messages = extRocketMQTemplate.receive(String.class);
        System.out.printf("receive from extRocketMQTemplate, messages=%s %n", messages);
    }
}
```