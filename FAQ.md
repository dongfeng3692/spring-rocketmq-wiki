1. How to connected many `nameserver` on production environment？

    `rocketmq.name-server` support the configuration of multiple `nameserver`, separated by `;`. For example: `172.19.0.1:9876; 172.19.0.2:9876`

1. When was `rocketMQTemplate` destroyed?

    Developers do not need to manually execute the `rocketMQTemplate.destroy ()` method when using `rocketMQTemplate` to send a message in the project, and` rocketMQTemplate` will be destroyed automatically when the spring container is destroyed.

1. start exception：`Caused by: org.apache.rocketmq.client.exception.MQClientException: The consumer group[xxx] has been created before, specify another name please`

   RocketMQ in the design do not want a consumer to deal with multiple types of messages at the same time, so the same `consumerGroup` consumer responsibility should be the same, do not do different things (that is, consumption of multiple topics). Suggested `consumerGroup` and` topic` one correspondence.
   
1. How is the message content body being serialized and deserialized?

    RocketMQ's message body is stored as `byte []`. When the business system message content body if it is `java.lang.String` type, unified in accordance with` utf-8` code into `byte []`; If the business system message content is not `java.lang.String` Type, then use [jackson-databind](https://github.com/FasterXML/jackson-databind) serialized into the `JSON` format string, and then unified in accordance with` utf-8` code into `byte [] `.
    
1. How do I specify the `tags` for topic?

    RocketMQ best practice recommended: an application as much as possible with one Topic, the message sub-type with `tags` to identify,` tags` can be set by the application free.
    
    When you use `rocketMQTemplate` to send a message, set the destination of the message by setting the` destination` parameter of the send method. The `destination` format is `topicName:tagName`, `:` Precedes the name of the topic, followed by the `tags` name.
    
    > Note:
    >
    > `tags` looks a complex, but when sending a message , the destination can only specify one topic under a `tag`, can not specify multiple.
    
1. How do I set the message's `key` when sending a message?

    You can send a message by overloading method like `xxxSend(String destination, Message<?> msg, ...)`, setting `headers` of `msg`. for example:
    
    ```java
    Message<?> message = MessageBuilder.withPayload(payload).setHeader(MessageConst.PROPERTY_KEYS, msgId).build();
    rocketMQTemplate.send("topic-test", message);
    ```

    Similarly, you can also set the message `FLAG`,` WAIT_STORE_MSG_OK` and some other user-defined other header information according to the above method.
    
    > Note:
    >
    > In the case of converting Spring's Message to RocketMQ's Message, to prevent the `header` information from conflicting with RocketMQ's system properties, the prefix `USERS_` was added in front of all `header` names. So if you want to get a custom message header when consuming, please pass through the key at the beginning of `USERS_` in the header.
    
1. When consume message, in addition to get the message `payload`, but also want to get RocketMQ message of other system attributes, how to do?

    Consumers in the realization of `RocketMQListener` interface, only need to be generic for the` MessageExt` can, so in the `onMessage` method will receive RocketMQ native 'MessageExt` message.
    
    ```java
    @Slf4j
    @Service
    @RocketMQMessageListener(topic = "test-topic-1", consumerGroup = "my-consumer_test-topic-1")
    public class MyConsumer2 implements RocketMQListener<MessageExt>{
        public void onMessage(MessageExt messageExt) {
            log.info("received messageExt: {}", messageExt);
        }
    }
    ```
    
1. How do I specify where consumers start consuming messages?

    The default consume offset please refer: [RocketMQ FAQ](http://rocketmq.apache.org/docs/faq/).
    To customize the consumer's starting location, simply add a `RocketMQPushConsumerLifecycleListener` interface implementation to the consumer class. Examples are as follows:
    
    ```java
    @Slf4j
    @Service
    @RocketMQMessageListener(topic = "test-topic-1", consumerGroup = "my-consumer_test-topic-1")
    public class MyConsumer1 implements RocketMQListener<String>, RocketMQPushConsumerLifecycleListener {
        @Override
        public void onMessage(String message) {
            log.info("received message: {}", message);
        }
    
        @Override
        public void prepareStart(final DefaultMQPushConsumer consumer) {
            // set consumer consume message from now
            consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_TIMESTAMP);
            consumer.setConsumeTimestamp(UtilAll.timeMillisToHumanString3(System.currentTimeMillis()));
        }
    }
    ```
    
    Similarly, any other configuration on `DefaultMQPushConsumer` can be done in the same way as above.
    
1. How do I send transactional messages?

    It needs two steps on client side: 

    a) Define a class which is annotated with @RocketMQTransactionListener and implements RocketMQLocalTransactionListener interface, in which, the executeLocalTransaction() and checkLocalTransaction() methods are implemented;

    b) Invoke the sendMessageInTransaction() method with the RocketMQTemplate API.

    **Note: After rocketmq-spring version 2.1.0, the annotation @RocketmqTransactionListener cannot set txProducerGroup, AK, SK. And these values are consistent with the corresponding rocketMQTemplate**
    
1. How do I create more than one RocketMQTemplate with a different name-server or other specific properties?

    Step1. Define an extra RocketMQTemplate with required properties. You can define different nameservers, groupnames from the standard rocketMQTemplate. If you do not define, it will use the global configuration definition by default. 
    ```java
    // The RocketMQTemplate's Spring Bean name is 'extRocketMQTemplate', same with the simplified class name (Initials lowercase)
      @ExtRocketMQTemplateConfiguration(nameServer="127.0.0.1:9876"    
         , ... // override other specific properties if needed
      )
      public class ExtRocketMQTemplate extends RocketMQTemplate {
      // keep the body empty
      }
    ```
    Step2. Use the extra RocketMQTemplate. e.g.
    ```java
    @Resource(name = "extRocketMQTemplate") // Must define the name to qualify to extra-defined RocketMQTemplate bean.
    private RocketMQTemplate extRocketMQTemplate;
    ```
    Then you can use the template as normal.

1. How to use extra rocketmqtemplate to send transactional messages？

    First, define a class which is annotated with @RocketMQTransactionListener and implements RocketMQLocalTransactionListener interface. The rocketMQTemplateBeanName of the annotation field indicates the bean name of the extra RocketMQTemplate (if it is not set, the default is the standard rocketmptemplate). For example, if the extra RocketMQTemplate bean name is "extRocketMQTemplate", the code As follows:

    ```java
    @RocketMQTransactionListener(rocketMQTemplateBeanName = "extRocketMQTemplate")
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
    ```

    Then use extRocketMQTemplate to call sendTessagentransaction to send transactional messages.

12. How do I create a consumer Listener with different name-server other than the global Spring configuration 'rocketmq.name-server' ?  

    ```java
    @Service
    @RocketMQMessageListener(
       nameServer = "NEW-NAMESERVER-LIST", // define new nameServer list
       topic = "test-topic-1", 
       consumerGroup = "my-consumer_test-topic-1",
       enableMsgTrace = true,
       customizedTraceTopic = "my-trace-topic"
    )
    public class MyNameServerConsumer implements RocketMQListener<String> {
       ...
    }
    ```