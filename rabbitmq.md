# RabbitMQ

Use Docker image with GUI management for convenience:  
`docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management`

## Exchanges
Think of an exchange as a message routing mediator. Producers always send messages to an exchange, and never to a queue as it's done with e.g. Azure Service Bus.  

There are four exchange types:
- Direct: forwards a message to a single queue based on a routing key.
- Fanout: forwards a message to all bounded queues, ignoring the routing key.
- Topic: messages are routed to bounded queues using a match between exchange pattern and routing keys on the queues.
- Headers: message headers attributes are used to determine routing to one or more queues.

Exchanges also contain properties such as:
- Name
- Durability: if the exchange is removed or not during restart/crash.
- Auto-Delete: determine if the exchange is deleted when there's no bound queue.


## Queues
Messages are routed to queues and consumers will listen for messages by listening to a queue bound to an exchange.

Queues contains properties, such as:
- Name
- Durability: determines if the queue survives a restart/crash
- Exclusive: determines if the queue is deleted when the connection owning the queue is closed.
- Auto-delete: broker deletes queue when there are no consumers.

It's possible to set message Time To Live in a queue by adding property "x-message-ttl".

Queues may have a routing-key used in Direct and Topic exchanges.

## Bindings
Exchanges use bindings to route messages to specific queues.