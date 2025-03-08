## Messaging Queue
- Enables exchange of messages between different nodes in a distributed system
- Allows asynchronuous communication, helpful when:
  - Writes can take long time
  - Data may have to be written in different places
  - System could be under high load
  - It may have to wait on other processes
- Used in large scale systems
  - Can be used to decouple different parts of the system (e.g. microservices)
  - Allowing them to operate independently and improving resilience & scalability
- Fault tolerance: it can retry service requests that have failed
- Frameworks: Apache Kafka, RabbitMQ, AWS SQS (Simple Queue Service)

## Workflow
- Producers send messages to a queue
- The queue stores and manages the messages until they are consumed
- As soon as any consumer/worker has capacity to process, they can pick up from queue
- Multiple consumers can read messages concurrently from the queue
- In some systems, there is a broker for message routing, filtering, transformation
- In some cases, consumers send back an acknowledgement
  - To ensure message delivery and preventing message loss

## Messages
- Structure
  - Headers: Metadata about the message, unique identifier, timstamp, message type, routing info
  - Body: Contains message payload
- Serialization
  - Converting complex data structures or objects into a format
    - That is easily transmitted, stored, reconstructed
  - JSON (JavaScript Object Notation)
  - XML (eXtensible Markup Language)
  - Protobuf (Protocol Buffers)
  - Binary Serialization: used for performance critical applications due to their compactness and speed
- Routing
  - Topic-based: Sent to topics or channels which can be subscribed
  - Direct: Sent to specific queues or consumers based on addresses or routing keys
  - Content-based: Filters or rules are defined to route messages

## Types
- Point to Point (P2P): Messages are deliverd to a specific recipient
- Publish Subsribe (Pub-Sub):
  - Messages are published to a topic and are delivered to all the subscribers of that topic
- Hybrid: Combination of P2P & Pub-Sub
- Dead Letter Queues
  - Temporarily stores & handle messages that cannot be processed successfully
    - Messages with errors in their content or format
    - Messages that excedd time-to-live (TTL) or delivery attempts
    - Messages that cannot be delivered to any consumer
  - Investigates and reprocesses failed messages preventing them from blocking the system

## Security
- Keep separate queues for sensitive data or urgent processing
- Enforce access controls to restrict who can send, receive, or administer the message queue
- Implement data encryption in transmit and at rest to protect messages from eavesdropping
