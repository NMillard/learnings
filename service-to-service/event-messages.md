# Event Messages

## Advanced Message Queuing Protocol (AMQP)

AMQP binary wire protocol (application layer) and an open standard that allows interoperability between systems. Whatever AMQP-compliant client can produce and consume AMQP-based messages.

### Note
There are two primary versions. 0-9-1 and 1-0.
The older version is used by RabbitMQ and the newer is implemented by e.g. Azure Service bus and ArtemisMQ.


### Links
- [AMQP spec](https://live.rabbitmq.com/resources/specs/amqp0-9-1.pdf)
- [MSDN AMQP 1.0 in Service Bus](https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-amqp-overview)


## Azure Service Bus

### Concepts
**Queues** are used as point-to-point communication.  
**Topics** on the other hand are useful in publish/subscribe scenarios, where one message is sent to all subscribers.  
**Subscriptions** a receive a copy of the message to a topic, and are named entities.

### CLI
Create a new servicebus namespace  
`az servicebus namespace create -n $SB_NAME -g $RG --sku Standard -l northeurope`

Create a topic  
`az servicebus topic create --namespace-name $SB_NAME -n $TOPIC_NAME -g $RG `

## Links
- [Azure Service Bus](https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-messaging-overview)
- [MSDN AMQP Guide](https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-amqp-protocol-guide)
- [Examples](https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/servicebus/Azure.Messaging.ServiceBus)