
RocketMQ-Spring provide request/reply semantics support.

- Producer     

Send request messages using sendAndReceice method.

> Note:
>
> Synchronous sending needs to indicate the return value type in the parameter of the method.
>
> Asynchronous sending needs to indicate the return value type in the callback's interface.

```java
@SpringBootApplication
public class ProducerApplication implements CommandLineRunner{
    @Resource
    private RocketMQTemplate rocketMQTemplate;
    
    public static void main(String[] args){
        SpringApplication.run(ProducerApplication.class, args);
    }
    
    public void run(String... args) throws Exception {
        // Send request in sync mode and receive a reply of String type.
        String replyString = rocketMQTemplate.sendAndReceive(stringRequestTopic, "request string", String.class);
        System.out.printf("send %s and receive %s %n", "request string", replyString);

        // Send request in async mode and receive a reply of User type.
        rocketMQTemplate.sendAndReceive(objectRequestTopic, new User("requestUserName",(byte) 9), new RocketMQLocalRequestCallback<User>() {
            @Override public void onSuccess(User message) {
                System.out.printf("send user object and receive %s %n", message.toString());
            }

            @Override public void onException(Throwable e) {
                e.printStackTrace();
            }
        }, 5000);
    }
    
    @Data
    @AllArgsConstructor
    public class User implements Serializable{
        private String userName;
    		private Byte userAge;
    }
}
```
- Consumer

It is neccessary to implement the RocketMQReplyListener\< T, R \> interface.  T represents the type of received value and R represents the type of returned value.

```java
@SpringBootApplication
public class ConsumerApplication{
    
    public static void main(String[] args){
        SpringApplication.run(ConsumerApplication.class, args);
    }
    
     /**
      * The consumer that replying String
      */
      @Service
      @RocketMQMessageListener(topic = "stringRequestTopic", consumerGroup = "stringRequestConsumer")
      public class StringConsumerWithReplyString implements RocketMQReplyListener<String, String> {
        @Override
        public String onMessage(String message) {
          System.out.printf("------- StringConsumerWithReplyString received: %s \n", message);
          return "reply string";
        }
      }
   
   /**
    * The consumer that replying Object
    */
    @Service
    @RocketMQMessageListener(topic = "objectRequestTopic", consumerGroup = "objectRequestConsumer")
    public class ObjectConsumerWithReplyUser implements RocketMQReplyListener<User, User>{
        public User onMessage(User user) {
          	System.out.printf("------- ObjectConsumerWithReplyUser received: %s \n", user);
          	User replyUser = new User("replyUserName",(byte) 10);	
          	return replyUser;
        }
    }

    @Data
    @AllArgsConstructor
    public class User implements Serializable{
        private String userName;
    		private Byte userAge;
    }
}
```