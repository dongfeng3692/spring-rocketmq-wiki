Modify application.properties
```properties
## application.properties
rocketmq.name-server=127.0.0.1:9876
rocketmq.producer.group=my-group
```

> Note:
> 
> Maybe you need change `127.0.0.1:9876` with your real NameServer address for RocketMQ

Send message in transaction and implement local check Listener

~~~java
@SpringBootApplication
public class ProducerApplication implements CommandLineRunner{
    @Resource
    private RocketMQTemplate rocketMQTemplate;

    public static void main(String[] args){
        SpringApplication.run(ProducerApplication.class, args);
    }

    public void run(String... args) throws Exception {
        try {
            // Build a SpringMessage for sending in transaction
            Message msg = MessageBuilder.withPayload(..)...;
            // In sendMessageInTransaction(), the first parameter transaction name ("test")
            // must be same with the @RocketMQTransactionListener's member field 'transName'
            rocketMQTemplate.sendMessageInTransaction("test", "test-topic", msg, null);
        } catch (MQClientException e) {
            e.printStackTrace(System.out);
        }
    }

    // Define transaction listener with the annotation @RocketMQTransactionListener
    @RocketMQTransactionListener
    class TransactionListenerImpl implements RocketMQLocalTransactionListener {
          @Override
          public RocketMQLocalTransactionState executeLocalTransaction(Message msg, Object arg) {
            // ... local transaction process, return bollback, commit or unknown
            return RocketMQLocalTransactionState.UNKNOWN;
          }

          @Override
          public RocketMQLocalTransactionState checkLocalTransaction(Message msg) {
            // ... check transaction status and return bollback, commit or unknown
            return RocketMQLocalTransactionState.COMMIT;
          }
    }
}
~~~