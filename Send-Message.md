Modify application.properties
```properties
## application.properties
rocketmq.name-server=127.0.0.1:9876
rocketmq.producer.group=my-group
```

> Note:
> 
> Maybe you need change `127.0.0.1:9876` with your real NameServer address for RocketMQ

```java
@SpringBootApplication
public class ProducerApplication implements CommandLineRunner{
    @Resource
    private RocketMQTemplate rocketMQTemplate;
    
    public static void main(String[] args){
        SpringApplication.run(ProducerApplication.class, args);
    }
    
    public void run(String... args) throws Exception {
        //send message synchronously
        rocketMQTemplate.convertAndSend("test-topic-1", "Hello, World!");
      	//send spring message
        rocketMQTemplate.send("test-topic-1", MessageBuilder.withPayload("Hello, World! I'm from spring message").build());
        //send messgae asynchronously
      	rocketMQTemplate.asyncSend("test-topic-2", new OrderPaidEvent("T_001", new BigDecimal("88.00")), new SendCallback() {
            @Override
            public void onSuccess(SendResult var1) {
                System.out.printf("async onSucess SendResult=%s %n", var1);
            }

            @Override
            public void onException(Throwable var1) {
                System.out.printf("async onException Throwable=%s %n", var1);
            }

        });
      	//Send messages orderly
      	rocketMQTemplate.syncSendOrderly("orderly_topic",MessageBuilder.withPayload("Hello, World").build(),"hashkey")
        
        //rocketMQTemplate.destroy(); // notes:  once rocketMQTemplate be destroyed, you can not send any message again with this rocketMQTemplate
    }
    
    @Data
    @AllArgsConstructor
    public class OrderPaidEvent implements Serializable{
        private String orderId;
        
        private BigDecimal paidMoney;
    }
}
```

> More relevant configurations for producing:
>
> ```properties
> rocketmq.producer.send-message-timeout=3000
> rocketmq.producer.compress-message-body-threshold=4096
> rocketmq.producer.max-message-size=4194304
> rocketmq.producer.retry-times-when-send-async-failed=0
> rocketmq.producer.retry-next-server=true
> rocketmq.producer.retry-times-when-send-failed=2
> ```