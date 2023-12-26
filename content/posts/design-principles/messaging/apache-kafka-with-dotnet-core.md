+++
title = 'Apache Kafka With Dotnet Core'
date = 2022-04-09T13:58:28Z
draft = false
series = "Design Principles"
tags = ["Microservices", "Messaging", "Event-driven architecture"]
+++

# Introduction

Apache Kafka has emerged as a game-changer in the realm of distributed systems, offering a robust and scalable solution for building event-driven architectures. In this technical blog, we will delve into the intricacies of Apache Kafka, exploring its core concepts, implementation in a .NET Core e-commerce system, and discussing the advantages, disadvantages, issues, and best practices associated with its use.

# Understanding Apache Kafka

## What is Apache Kafka?

Apache Kafka is a distributed streaming platform that excels at handling real-time data feeds and creating scalable, fault-tolerant, and high-throughput messaging systems. Born out of the need for efficiently handling massive amounts of data at LinkedIn, Kafka has evolved into an open-source powerhouse, providing a unified solution for distributed event processing.

## Why Kafka?

The rationale for Kafka lies in addressing challenges posed by real-time data processing in large-scale distributed environments. It acts as a bridge, connecting disparate components and enabling seamless communication, making it an ideal choice for scenarios where traditional messaging systems fall short.

## Problems Resolved by Kafka

- **Scalability**: Kafka's distributed architecture allows it to scale horizontally, ensuring it can handle large volumes of data and high throughput.
- **Reliability**: Kafka guarantees fault tolerance by replicating data across multiple nodes, ensuring that even if a node fails, the system continues to function.
- **Real-time Processing**: Kafka's ability to process events in real-time makes it suitable for scenarios requiring low-latency communication and immediate data updates.

# Kafka in Action: .NET Core E-Commerce System

Let's dive into a practical example of implementing an event-driven architecture using Apache Kafka in the context of a .NET Core e-commerce system.

## .NET Core Services and APIs

Our e-commerce system is built using .NET Core, leveraging the MediatR library for implementing the Command Query Responsibility Segregation (CQRS) pattern. AutoMapper facilitates seamless object-to-object mapping, and EF Core InMemory database ensures efficient data management. Unit tests are handled using xUnit.

## Event-Driven Architecture with Kafka

In an event-driven architecture using Apache Kafka, the key components are the Event Producer, Event Consumer, and the Event Bus. Let's delve into each of these elements to understand their roles in facilitating seamless communication within a distributed system.

### Event Producer

The Event Producer is responsible for initiating and publishing events to the Kafka cluster. In the context of our e-commerce system, consider scenarios where events such as "Order Placed," "Inventory Updated," or "Payment Confirmed" occur. The Event Producer, implemented using C# .NET Core, sends these events to specific Kafka topics.

```csharp
// Example of an Event Producer
public class OrderPlacedEventProducer
{
    private readonly IKafkaProducer _kafkaProducer;

    public OrderPlacedEventProducer(IKafkaProducer kafkaProducer)
    {
        _kafkaProducer = kafkaProducer;
    }

    public async Task PublishOrderPlacedEvent(OrderPlacedEvent orderEvent)
    {
        await _kafkaProducer.ProduceAsync("order-placed-topic", orderEvent);
    }
}
```

In this example, the `OrderPlacedEventProducer` encapsulates the logic for publishing an "Order Placed" event to the "order-placed-topic" Kafka topic. The use of a dedicated producer for each event type ensures a modular and maintainable design.

### Event Consumer

On the other side of the spectrum, the Event Consumer listens for and processes events from Kafka topics. In the e-commerce system, the Event Consumer is designed to react to events like "Order Placed" and initiate actions such as updating inventory or sending notifications.

```csharp
// Example of an Event Consumer
public class OrderPlacedEventHandler : INotificationHandler<OrderPlacedEvent>
{
    public async Task Handle(OrderPlacedEvent notification, CancellationToken cancellationToken)
    {
        // Handle the order placed event, update inventory, send notifications, etc.
    }
}
```

Here, the `OrderPlacedEventHandler` is a consumer designed to handle the "Order Placed" event. The `INotificationHandler` interface, part of the MediatR library, facilitates the decoupling of event handlers, promoting a modular and extensible architecture.

### Event Bus

The Event Bus serves as the backbone of the event-driven architecture, acting as the communication channel between the Event Producers and Consumers. In the context of Kafka, the Event Bus is realized through Kafka topics, which act as message queues where events are published and consumed.

```csharp
// Example of Kafka Consumer Configuration
var consumerConfig = new ConsumerConfig
{
    BootstrapServers = "localhost:9092",
    GroupId = "order-service-group",
    AutoOffsetReset = AutoOffsetReset.Earliest
};

// Example of Kafka Consumer
var consumerBuilder = new ConsumerBuilder<Ignore, string>(consumerConfig);
var consumer = consumerBuilder.Build();
consumer.Subscribe("order-placed-topic");

while (true)
{
    var consumeResult = consumer.Consume();
    // Process the consumed event
}
```

In this Kafka Consumer example, a consumer is configured to subscribe to the "order-placed-topic." The continuous loop ensures that the consumer is always listening for new events. As events arrive on the topic, the consumer processes and reacts to them, forming the core of the event-driven communication.

## Local Environment Setup

### .NET Core API Project

1. Create a new .NET Core API project using the following command:

```sh
dotnet new web -n ECommerceApi
```

2. Add necessary dependencies using the MediatR and AutoMapper NuGet packages.

3. Docker Compose YAML

```yaml
version: "3.8"

services:
  ecommerce-api:
    build:
      context: .
      dockerfile: ECommerceApi/Dockerfile
    ports:
      - "5000:5000"
    depends_on:
      - kafka
      - kafdrop
      - schema-registry

  kafka:
    image: wurstmeister/kafka:2.13-2.8.0
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_LISTENERS: INSIDE://kafka:9093,OUTSIDE://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT
      KAFKA_LISTENERS: INSIDE://0.0.0.0:9093,OUTSIDE://0.0.0.0:9092
      KAFKA_INTER_BROKER_LISTENER_NAME: INSIDE
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

  zookeeper:
    image: wurstmeister/zookeeper:3.4.6
    ports:
      - "2181:2181"

  kafdrop:
    image: obsidiandynamics/kafdrop:3.27.0
    ports:
      - "9000:9000"
    environment:
      KAFKA_BROKERCONNECT: kafka:9093

  schema-registry:
    image: confluentinc/cp-schema-registry:7.1.0
    ports:
      - "8081:8081"
    environment:
      SCHEMA_REGISTRY_KAFKASTORE_CONNECTION_URL: "zookeeper:2181"
      SCHEMA_REGISTRY_HOST_NAME: schema-registry
      SCHEMA_REGISTRY_LISTENERS: "http://0.0.0.0:8081"
      SCHEMA_REGISTRY_AVRO_COMPATIBILITY_LEVEL: "full"
    depends_on:
      - kafka
```

- **Apache Zookeeper**: Apache ZooKeeper is a distributed coordination service essential for maintaining configuration information, naming, providing distributed synchronization, and group services. Kafka relies on ZooKeeper for distributed coordination among its broker nodes, managing topics, and handling leader election for partitions.
- **Apache Kafka**: Apache Kafka is a distributed streaming platform that enables the building of real-time data pipelines and streaming applications. It acts as a durable, fault-tolerant publish-subscribe messaging system, handling high-throughput data streams. Kafka is structured around topics, partitions, producers, and consumers, providing scalability and fault tolerance.
- **Kafdrop**: Kafdrop is a web-based UI for monitoring Apache Kafka clusters. It offers insights into topics, partitions, and messages, making it a valuable tool for visualizing the state of Kafka topics and consumer groups. Kafdrop simplifies Kafka cluster management and enhances visibility for developers and administrators.
- **Schema Registry, Avro Schema, and Avrogen**:
  - **Avro Schema**: Avro is a binary serialization format used by Kafka for efficient and compact data representation. Avro schemas define the structure of the data being exchanged. They provide a way to encode and decode messages uniformly, ensuring compatibility across different systems and programming languages.
  - **Avrogen**: Avrogen is a tool that generates Avro serialization and deserialization code based on Avro schema files. It simplifies the integration of Avro with programming languages by creating language-specific classes and methods for working with Avro data. Avrogen aids in maintaining a consistent data format between producers and consumers in a Kafka ecosystem.
  - **Schema Registry**: The Schema Registry is a critical component in a Kafka ecosystem using Avro. It centralizes the storage and management of Avro schemas, ensuring that producers and consumers share a common schema for data serialization. This guarantees schema evolution without breaking existing systems. The Schema Registry provides RESTful APIs for schema registration, retrieval, and compatibility checks.

## Running and Debugging Docker Compose and the Project

Steps for Running Docker Compose:

1. Ensure Docker Desktop is installed and running on your machine.

2. Open a terminal and navigate to the directory containing your `docker-compose.yaml` file.

3. Run the following command to start the services defined in the Docker Compose file:

```sh
docker-compose up -d
```

4. To view the logs of the running services, use:

```sh
docker-compose logs -f
```

### Steps for Running and Debugging the .NET Core Project:

1. Ensure you have the latest version of the .NET Core SDK installed.

2. Open a terminal and navigate to the root directory of your .NET Core project.

3. Build the project using:

```sh
dotnet build
```

4. Run the project:

```sh
dotnet run --project ECommerceApi/
ECommerceApi.csproj
```

5. Access the API at `http://localhost:5000` in your web browser or a tool like Postman.

6. For debugging, use Visual Studio Code or Visual Studio. Set breakpoints, inspect variables, and step through the code to identify and fix issues.

7. To stop the Docker Compose services, use:

```sh
docker-compose down
```

8. To clean up containers, networks, and volumes, use:

```sh
docker-compose down -v
```

By following these steps, you can run and debug both the Docker Compose environment and the .NET Core project effectively. This ensures a smooth development and testing experience for your Kafka-based e-commerce system.

# Advantages and Disadvantages of Apache Kafka

## Advantages

- **Scalability**: Kafka scales horizontally, handling increased load by adding more nodes to the cluster.
- **Reliability**: Data replication ensures fault tolerance, making Kafka highly reliable.
- **Real-time Processing**: Kafka processes events in real-time, making it suitable for low-latency applications.
- **Durable Storage**: Kafka provides durable storage of events, allowing historical data retrieval.

## Disadvantages

- **Complexity**: Setting up and managing a Kafka cluster can be complex, especially for beginners.
- **Resource Intensive**: Kafka may require substantial resources, particularly in terms of storage and memory.
- **Learning Curve**: Developers unfamiliar with event-driven architectures may face a learning curve when adopting Kafka.

# Issues and Considerations

- **Schema Evolution**: Ensuring compatibility during schema evolution is crucial when using Kafka, and the Schema Registry plays a key role in managing schema changes.
- **Monitoring and Operations**: Proper monitoring tools are necessary to ensure the health and performance of the Kafka cluster.
- **Security Considerations**: Implementing authentication, authorization, and encryption mechanisms is vital to securing the Kafka cluster.

# When to Use and When Not to Use Kafka

Use Cases/Scenarios

- **Event-Driven Microservices**: Kafka is ideal for microservices architectures that rely on event-driven communication patterns.
- **Real-Time Analytics**: Applications requiring real-time analytics and insights benefit from Kafka's ability to process events as they occur.
- **Log Aggregation**: Kafka serves as a robust solution for log aggregation, collecting and centralizing logs from various services.

When Not to Use Kafka

- **Simple Point-to-Point Communication**: For straightforward, point-to-point communication, simpler messaging systems like RabbitMQ may be more suitable.
- **Small-Scale Applications**: In scenarios where the complexity of setting up and managing Kafka outweighs the benefits, simpler solutions may be more practical.

# Best Practices

- **Topic Naming Conventions**: Adopt a clear and consistent naming convention for Kafka topics, reflecting the nature of the events they represent.
- **Data Serialization**: Use efficient serialization formats (Avro, Protobuf) to optimize data transfer and storage.
- **Partitioning Strategies**: Choose appropriate partitioning strategies to ensure even distribution of data across Kafka brokers.
- **Monitoring and Metrics**: Implement monitoring tools to track Kafka cluster health, performance, and consumer lag.
- **Security Considerations**: Secure your Kafka cluster using authentication, authorization, and encryption mechanisms.

# Conclusion

Apache Kafka stands as a powerful solution for building scalable, real-time event-driven architectures. In this blog, we explored its fundamentals, walked through a practical .NET Core implementation for an e-commerce system, discussed advantages, disadvantages, issues, and provided best practices. By leveraging Kafka's capabilities, developers can build resilient and efficient systems that thrive in the world of distributed computing.

# References

- [Apache Kafka documentation](https://kafka.apache.org/documentation.html)
- [Zookeeper documentation](https://cwiki.apache.org/confluence/display/ZOOKEEPER/Index)
- [Kafka client protocol](https://cwiki.apache.org/confluence/display/KAFKA/A+Guide+To+The+Kafka+Protocol)
- [Apache Kafka .NET Client](https://docs.confluent.io/kafka-clients/dotnet/current/overview.html)
- [Apache Kafka developer guide for Azure Event Hubs](https://learn.microsoft.com/en-us/azure/event-hubs/apache-kafka-developer-guide)
