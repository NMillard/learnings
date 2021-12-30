# Spring Boot Basics

## Application Settings Configuration
Prefer yaml-based configuration over the traditional key-pair style in application.properties. Any configuration that may be changed in any environment must be configurable from start

__Never__ include secrets in the configuration files.

Initializing values directly in components may be okay in some instances.

```java
@Component
public class MyBean {

    @Value("${name}") // <- read from yaml configuration file
    private String name;

    // ...
}
```

[Read more about spring configurations here](https://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/html/boot-features-external-config.html)


## Setting up messaging

In-process messages may be wired up by annotating a component class' method with `@ServiceActivator(inputChannel = "inProcessRegistration")` where inProcessRegistration is the channel to listen on.  
In order to sent messages to the channel you need to configure a bean with a method name corresponding to the channel name.  

```java
@Configuration
public class IntegrationConfig {

    @Bean
    public MessageChannel inProcessRegistration() {
        return new DirectChannel();
    }

    // ...
}
```

Listening to e.g. a RabbitMQ queue is somewhat more involved. You'll need to register a couple of classes with the dependency container, such as below.  

```java
// Configuring the queue and exchange.
@Configuration
public class IntegrationConfig {
    /** Setup the queue. */
    @Bean
    public Queue getQueue() {
        return new Queue("message-test");
    }

    /** Setup the exchange */
    @Bean
    public DirectExchange getExchange() {
        return new DirectExchange("message-test-exchange");
    }

    /**
     * A queue is required to be bound to an exchange. Messages are sent to an exchange and then routed
     * to a queue based on the routing-key.
     */
    @Bean
    public Binding getBinding(Queue queue, DirectExchange exchange) {
        return BindingBuilder
                .bind(queue)
                .to(exchange)
                .with("message-test-key");
    }

    /** Converter for a message sent to a queue and received by a listener. */
    @Bean
    public MessageConverter getMessageConverter() {
        return new Jackson2JsonMessageConverter();
    }

    /**
     * Template used to send and recieve messages - this is optional as one AmqpTemplate is already
     * provided by the maven dependency.
     */
    @Bean
    public AmqpTemplate getTemplate(ConnectionFactory connectionFactory) {
        var template = new RabbitTemplate(connectionFactory);
        template.setMessageConverter(getMessageConverter());

        return template;
    }
}
// ------

// How to sent a message to the queue
@Component
public class RegisterParticipant {

    public RegisterParticipant(@Qualifier("getTemplate") AmqpTemplate template) {
        this.template = template;
    }

    private final AmqpTemplate template;

    public void sendMessage() {
        ParticipantMessage nicm = new ParticipantMessage();
        nicm.setParticipantName("nicm");
        template.convertAndSend("message-test-exchange", "message-test-key", nicm);
    }

    /** Process messages coming from the specified queue. */
    @RabbitListener(queues = "message-test")
    public void recieveMessage(ParticipantMessage message) {
        System.out.println("Received the message");
    }
}
```