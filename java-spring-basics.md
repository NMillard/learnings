# Spring Boot Basics

## Dependency Injection
In Spring DI is also referred to as "Configuration" which is slightly misleading if you have a .NET background.  

Spring uses a convention over configuration approach by utilizing "stereotypes". You wire up components by annoating them with `@Component`, `@Service`, and `@Repository`.  
Or, you can use "configuration classes" that acts as a collection of factory methods for different types, such as the example below.  

```java
@Configuration
public class LoggerConfiguration {

    /** Get a new instance of Logger every time it's requested from the container
     *  and instantiate it with the calling class as argument  */
    @Bean
    @Scope("prototype")
    public Logger getLogger(InjectionPoint injectionPoint) {
        return LoggerFactory.getLogger(injectionPoint.getMember().getDeclaringClass());
    }
}
```

Spring also auto-configures "beans" if they're found in the classpath. To see which configurations are applied start the application with `--debug` switch.

## Application Settings Configuration
Prefer yaml-based configuration over the traditional key-pair style in application.properties. Any configuration that may be changed in any environment must be configurable from start.  

The application file is found in resources/application.yml.

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

## Dates
Avoid using the old, non-thread safe `Date`. Instead use types from java.time such as `LocalDate`, `LocalTime`, `Instant`.  


## Optional (may return null)
Use the `Optional<>` type to indicate a method may not return anything.


## Spring WebFlux
A reactive spring framework available from spring 5.x.  
WebFlux offers an alternative programming model from the traditional spring MVC. WebFlux operates on reactive streams and are by nature async.

The whole stack has to be reactive, so you can't mix regular, blocking code with WebFlux.

## Testing

### Testing Spring endpoints

You can create test classes that are annotated as demonstrated below.  

```java
@ExtendWith(SpringExtension.class)
@SpringBootTest
@AutoConfigureMockMVc
class SomeTest {
    @MockBean
    private SomeServiceClass service;

    @Autowired
    private MockMvc mvc;
}
```

Use the `mvc` field to test endpoints and `hamcrest` result matchers to verify the response.

Use `Mockito` for mocking interfaces and abstract classes.  
- Prefer the `doReturn/when` over `when/thenReturn` approach.